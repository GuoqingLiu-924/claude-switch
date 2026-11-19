# claude-switch

Switch between multiple Claude Desktop accounts on Linux with one command — no
logging out, no re-entering your email. Optionally shares the Code tab's
session list across accounts.

[中文说明](#中文说明)

## Quick start

```bash
# 1. Get it and put it on your PATH
git clone https://github.com/GuoqingLiu-924/claude-switch.git
cd claude-switch
mkdir -p ~/.local/bin && ln -s "$PWD/claude-switch" ~/.local/bin/claude-switch

# 2. (optional) tab completion — add this line to ~/.bashrc, then reopen the terminal
source <(claude-switch completion bash)      # zsh: use "zsh" in ~/.zshrc

# 3. Fully quit Claude Desktop, then migrate your current account (once)
claude-switch init

# 4. Add a second account and switch to it
claude-switch add work
claude-switch use work

# 5. Open Claude Desktop and log in once. From now on switching is just:
#    quit Claude -> claude-switch use <name> -> reopen Claude
```

## How it works

Claude Desktop keeps its login state in `~/.config/Claude`. `claude-switch`
turns that path into a symlink and keeps one profile directory per account
under `~/.local/share/claude-switch/profiles/`. Switching = repointing the
symlink. It never launches the app: quit Claude, run `use`, reopen it.

## Usage

```bash
claude-switch init          # once: quit Claude first; moves ~/.config/Claude into profiles/default
claude-switch add work      # create an empty profile
claude-switch use work      # switch (Claude must be closed), then open Claude and log in once
claude-switch use default   # switch back
claude-switch list          # * marks the current profile
claude-switch current
claude-switch rm work       # delete a profile's data (asks first)
claude-switch share         # merge all profiles' Code sessions (also done automatically on init/use)
```

## Tab completion

Completes commands and profile names (for `use` / `rm`). Add one line to your shell rc:

```bash
# bash (~/.bashrc)
source <(claude-switch completion bash)
# zsh (~/.zshrc)
source <(claude-switch completion zsh)
```

## Shared Code sessions

The Code tab stores sessions in `claude-code-sessions/<accountId>/<orgId>/`.
On `init`/`use`, each such directory is merged into one shared directory
(`~/.local/share/claude-switch/shared-sessions`) and replaced by a symlink, so
every account sees the same session list. A timestamped backup of
`claude-code-sessions` is made first, next to the original.

Caveats: the list is read at app start, so restart after switching; don't use
the same session from two accounts at once. Whether the app filters shared
sessions per account is unverified — if an account can't see them, remove the
symlink and restore the backup.

## Configuration

| Variable | Default |
|---|---|
| `CLAUDE_CONFIG_DIR` | `${XDG_CONFIG_HOME:-~/.config}/Claude` |
| `CLAUDE_SWITCH_HOME` | `${XDG_DATA_HOME:-~/.local/share}/claude-switch` |
| `CLAUDE_SWITCH_PROC` | `claude-desktop\|Claude` (process-name regex used to detect a running app) |
| `CLAUDE_SWITCH_FORCE` | set to `1` to skip the "app is running" check |

Profiles contain login cookies — never commit or share them.

## 中文说明

在 Linux 上一条命令切换多个 Claude Desktop 账号，不用退出登录、重新输入邮箱。
可选地让各账号共享 Code 标签的会话列表。

原理：把 `~/.config/Claude` 变成软链接，每个账号一个目录
（`~/.local/share/claude-switch/profiles/<名字>`），切换就是改软链接。工具不会启动应用：
先完全退出 Claude，运行 `claude-switch use <名字>`，再打开应用。

快速开始：见上方 Quick start（克隆、链接到 PATH、退出 Claude 后 `init`、`add` + `use`、打开应用登录一次）。

用法：先退出 Claude 执行一次 `claude-switch init`；之后 `add <名字>` 新建账号，
`use <名字>` 切换（首次切到新账号需登录一次），`list` 查看，`rm` 删除。
补全：在 `~/.bashrc` 加 `source <(claude-switch completion bash)`（zsh 同理），可补全命令和账号名。
会话共享在 `init`/`use` 时自动完成，会先备份。切换后需重启应用才能刷新会话列表。
