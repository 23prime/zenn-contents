# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## General agent rules

- When users ask questions, answer them instead of doing the work.

### Shell Rules

- Always use `rm -f` (never bare `rm`)
- Before running a series of `git` commands, confirm you are in the project root; if not, `cd` there first. Then run all subsequent `git` commands from that directory without the `-C` option.

## Project Overview

[23prime](https://zenn.dev/23prime) の Zenn 投稿コンテンツ管理リポジトリ。
[Zenn CLI](https://zenn.dev/zenn/articles/zenn-cli-guide) を使って記事・本を管理する。

- `articles/` — 記事
- `books/` — 本
