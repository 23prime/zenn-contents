---
title: "今日もまた Arch Linux が壊れたので直した"
emoji: "🐧"
type: "tech"
topics: ["archlinux", "linux", "xfs", "mkinitcpio", "recovery"]
published: false
---

## 💥 起きたこと

…ということで、今日もまた作業中に急に電源が落ちました。
バッテリー切れではなく、本当に急にね。

再起動したら Arch Linux が立ち上がらず、画面にこんなエラーが出ていました。

```txt
EFI stub: ERROR: Failed to open file: initramfs-linux.img
EFI stub: ERROR: Failed to load initrd: 0x800000000000e
EFI stub: ERROR: efi_stub_entry() failed!
Failed to execute Arch Linux (\vmlinuz-linux): Not Found
```

ちなみにパッケージの更新中だったわけではありません。ただの電源断です。

## 🖥️ 環境

- ThinkPad X1 Carbon Gen 10
- NVMe SSD 1TB
- パーティション構成：
  - `/dev/nvme0n1p1` -- ESP（FAT32）、`/boot` に直接マウント
  - `/dev/nvme0n1p2` -- swap
  - `/dev/nvme0n1p3` -- ルート（**XFS**）
- ブートローダー：**systemd-boot**
- Intel CPU なので `intel-ucode.img` も併用

[Arch Linux Install Battle 2022 Summer](https://zenn.dev/23prime/articles/a120f9c00bf6db) にて構築し、育ててきた環境です。

## 🔍 原因の推測

エラー文だけ見ると EFISTUB 直接ブートのように見えますが、実際は systemd-boot が `vmlinuz-linux` をロードしようとした直後に発生したエラーです。
`initramfs-linux.img` を開けない、と。

電源断によってファイルシステムが正しくアンマウントされずに壊れた可能性を疑いました。
特に ESP は FAT32 でジャーナルがないため、書き込み中の電源断に非常に弱い特性があります。

## 🛠️ 復旧手順

### 💿 1. ライブ USB から起動

手元にインストール時の USB メモリが残っていたので、それを使いました。
ない場合は別のマシンで [Arch Linux の ISO](https://archlinux.org/download/) を落として Rufus 等で作りましょう。

Secure Boot は最初から OFF にしていたので、USB を挿して起動するだけです。

### 🔎 2. ESP（FAT32）のチェック

:::message alert
以下の `/dev/nvme0n1p*` は**この記事の環境の例**です。実行前に必ず `lsblk -f` または `blkid` で自分の環境のデバイス名を確認し、置き換えてください。
:::

ネットには接続せず、まずファイルシステムの状態を確認します。**マウントせずに**実行するのがポイントです。

```bash
fsck.fat -a -w -V /dev/nvme0n1p1
```

結果：

```txt
Dirty bit is set. Fs was not properly unmounted and some data may be corrupt.
  Automatically removing dirty bit.
Starting verification pass.
/dev/nvme0n1p1: 14 files, 7740/140520 clusters
```

やはり dirty bit が立っていました。
電源断で正しくアンマウントされなかったようです。
自動で削除され、クラスタ数も妥当でした。

ブートセクタとバックアップに 1 バイトの差異が報告されましたが、「mostly harmless」とのことなので無視します。

### 📋 3. ルート（XFS）のチェック

XFS は `fsck.xfs` が実質何もしないため、`xfs_repair` を使います。まずは dry-run で状態だけ確認します。

```bash
xfs_repair -n /dev/nvme0n1p3
```

結果、`would have reset inode ... nlinks from 0 to 1` というログが大量に出力されました。
inode にデータはあるのにディレクトリエントリから参照されていない、いわゆる孤児 inode 状態です。
電源断でメタデータの書き込みが中途半端になったときの典型的なパターンです。

### ⏯️ 4. XFS ログの再生

`-n` を外して実行すると：

```txt
ERROR: The filesystem has valuable metadata changes in a log
which needs to be replayed. Mount the filesystem to replay the
log, and unmount it before re-running xfs_repair.
```

XFS のログ（ジャーナル）に電源断直前の未反映の変更が残っており、先にそれを再生するよう求められます。

:::message alert
ここで `-L` を付けるとログを破棄してしまうので、マニュアル通り素直にマウントします。
:::

```bash
mount /dev/nvme0n1p3 /mnt
```

マウントに成功した瞬間、カーネルが自動でログを再生してくれます。
`ls /mnt` で `bin boot dev etc home ...` が見えれば成功です。

続いてアンマウントします：

```bash
umount /mnt
```

### 🔧 5. xfs_repair を再度実行

```bash
xfs_repair /dev/nvme0n1p3
```

今度は Phase 1 〜 Phase 7 まで正常に通り、最後に `done` が表示されました。
Phase 6 で `moving disconnected inodes to lost+found ...` と出力され、孤児 inode が `/lost+found` に移動されたことを確認できました。
多くはログ再生で解消されていたため、修正件数自体は少数でした。

### 📂 6. /boot の中身を確認

ファイルシステムが健全な状態になったので、中身を確認します。

```bash
mount /dev/nvme0n1p3 /mnt
mount /dev/nvme0n1p1 /mnt/boot
ls -la /mnt/boot/
```

結果：

```txt
EFI/
intel-ucode.img      14 MB
loader/
vmlinuz-linux        16 MB
```

やはりエラーメッセージどおり、`initramfs-linux.img` と `initramfs-linux-fallback.img` が両方とも消えています。

`vmlinuz-linux` は残っていたため、電源断時に initramfs の書き込みが走っていた、あるいは壊れた状態で削除されたと思われます。

### ⚙️ 7. chroot して initramfs を再生成

```bash
arch-chroot /mnt
mkinitcpio -P
```

`-P` は全プリセットを処理するオプションです。
通常版と fallback 版の両方が作成されます。

fallback 生成時に `WARNING: Possibly missing firmware for module: 'ast', 'xhci_pci_renesas', ...` のような警告が出ますが、これは fallback に含まれる汎用ドライバ向けのもので、ThinkPad には無関係なハードウェア（サーバー用 RAID や古い GPU など）のファームウェアが存在しないことを示しています。
通常版では autodetect が効くので問題ありません。

### ✅ 8. 確認

```bash
ls -la /boot/
```

```txt
EFI/
initramfs-linux-fallback.img   204 MB
initramfs-linux.img             22 MB
intel-ucode.img                 14 MB
loader/
vmlinuz-linux                   16 MB
```

お、無事に生成されました。ローダーエントリも念のため確認します。

```bash
cat /boot/loader/entries/arch.conf
```

```txt
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=UUID=... rw
```

問題、なし！

### 🔄 9. 再起動

```bash
exit
umount -R /mnt
reboot
```

USB を抜いて、あとは祈るだけ…

…

………

無事に起動しました。🎉

## 🗒️ 振り返り

全体のストーリーはシンプルです：

1. 電源断で ESP（FAT32）に dirty bit
2. 電源断で XFS のジャーナルが未再生
3. その流れで initramfs が消えた

対処の流れ：

1. `fsck.fat` で ESP をチェック
2. `xfs_repair -n` で XFS の状態を確認
3. マウントで XFS のログを再生
4. `xfs_repair` で本修復
5. chroot して `mkinitcpio -P` で initramfs を再生成

## 💡 教訓

- **電源断はファイルシステムの敵**。UPS があると安心です。ノートならバッテリーのヘタり具合をたまにチェックする価値があります。
- **XFS の `-L` オプションは最終手段**。ログ再生でいけるならそちらを優先します。エラーメッセージをきちんと読むと、ツールが正しい手順を教えてくれています。
- **FAT32 はジャーナルがないので壊れやすい**。ESP のような場所だからこそ、書き込み中の電源断には十分な注意が必要です。
- **ライブ USB は捨てずに取っておく**。今回のような緊急時にすぐ使えます。
- **構築手順（ログ）を残しておく**。復旧時にパーティション構成やブートローダーの設定をすぐ確認できて役立ちました。

## 🔗 参考

- [Arch Linux Install Battle 2022 Summer](https://zenn.dev/23prime/articles/a120f9c00bf6db)
- [Arch Linux Downloads](https://archlinux.org/download/)
- [mkinitcpio - ArchWiki](https://wiki.archlinux.jp/index.php/Mkinitcpio)
- [XFS - ArchWiki](https://wiki.archlinux.jp/index.php/XFS)
- [systemd-boot - ArchWiki](https://wiki.archlinux.jp/index.php/Systemd-boot)
