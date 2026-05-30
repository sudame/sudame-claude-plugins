---
name: using-gwq
description: Git の worktree を `git worktree` ではなく gwq で操作する。worktree を作る・一覧する・移動する・中でコマンドを実行する・削除するときや、ブランチを隔離して作業するとき、複数ブランチで並行作業するときに必ず使う。
---

# gwq で Git worktree を扱う

worktree の操作は常に `gwq` を使い、`git worktree` は使わない。`gwq` は worktree を
固定のベースディレクトリ配下（`~/worktrees/{{.Host}}/{{.Owner}}/{{.Repository}}/{{.Branch}}`）
に必ず配置するので、パスが予測可能になる。引数の体系が `git worktree` とは違うため、
推測せず以下のルールに従うこと。

## `git worktree` との重要な違い

- **パスは渡さない。** worktree の配置場所は gwq に任せる。ブランチだけを渡す。
- **引数はブランチが先**: `gwq add [branch] [path]` で、`git worktree add [path] [branch]`
  とは逆の順序。
- **`remove` はパスではなくパターンを取る**: ブランチ名やパスの一部にマッチさせて削除する。
- **対話モードを絶対に起動しない。** パターンやブランチを省略すると fuzzy finder (fzf)
  が起動し、非対話のエージェントはハングする。常に明示的にブランチ／パターンを渡し、
  `-i`/`--interactive` は使わない。

## worktree を作る

```bash
# 新しいブランチ＋worktree
gwq add -b feature/new-thing

# 既存ブランチから worktree
gwq add feature/existing
```

パス引数は渡さないこと。`-s`/`--stay`（シェルを起動する）も使わないこと。

## worktree のパスを調べて移動する

`gwq cd` は新しい対話シェルを立ち上げる。人間には便利だが、エージェントにとっては
不便（立ち上がったシェルの中で操作できない）。代わりに、まずパスを解決してから
自分で `cd` する：

```bash
# 正確なパスを取得する（曖昧なときに fuzzy finder を避けるため --json を使う）
gwq list --json
# 該当する .path を選び、別ステップでディレクトリを移動する
```

`gwq get <pattern>` もパスを表示するが、パターンが複数の worktree にマッチすると
fuzzy finder が起動する。`gwq list --json` で正確なエントリを自分で選び、その `.path`
へ `cd` する方を優先すること。

ディレクトリを移動せずに worktree の中でコマンドを実行したいときは `exec` を使う：

```bash
gwq exec feature/existing -- npm test
```

## 一覧と状態確認

```bash
gwq list            # 今のリポジトリの worktree
gwq list --json     # 機械可読。パス抽出にはこれを使う
gwq list -g         # 全リポジトリの worktree（どこからでも）
gwq status --json   # 全 worktree の git status
```

## worktree を削除する

```bash
# worktree を削除（ブランチは残す）。パターンはブランチ名かパスにマッチ
gwq remove feature/old

# worktree とブランチを両方削除
gwq remove -b feature/done

# 削除せずに対象だけ確認
gwq remove --dry-run feature/old
```

引数なしの `gwq remove` は実行しない（fuzzy finder が起動する）。常にパターンを渡すこと。

## クイックリファレンス

| やりたいこと | コマンド |
| --- | --- |
| 新ブランチ＋worktree | `gwq add -b <branch>` |
| 既存ブランチから worktree | `gwq add <branch>` |
| 一覧（パス付き） | `gwq list --json` |
| worktree 内でコマンド実行 | `gwq exec <pattern> -- <cmd>` |
| cd するためのパス取得 | `gwq list --json` → `.path` を選ぶ → 別ステップで `cd` |
| worktree を削除 | `gwq remove <pattern>` |
| worktree＋ブランチを削除 | `gwq remove -b <pattern>` |

避けること（対話的／シェルを起動するためエージェントがハングする）: 引数なしの
`gwq add`/`gwq remove`/`gwq get`、`-i`/`--interactive`、`-s`/`--stay`、`gwq cd`。
