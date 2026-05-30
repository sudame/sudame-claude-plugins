# sudame-claude-plugins

sudame 個人用の [Claude Code](https://claude.com/claude-code) プラグイン marketplace。

## インストール

```
/plugin marketplace add sudame/sudame-claude-plugins
/plugin install gwq@sudame-claude-plugins
```

## プラグイン一覧

### `gwq`

`using-gwq` スキルを提供する。Claude が Git の worktree を `git worktree` ではなく
[`gwq`](https://github.com/d-kuro/gwq) 経由で操作するようになり、worktree が常に
固定の予測可能な場所に作られる。また、エージェントをハングさせる gwq の対話モード
（fuzzy finder／シェル起動）を避けるよう誘導する。
