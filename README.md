# qq-chime — 多音效提醒 Skill

让 Codex 和 WorkBuddy 在「需要你确认 / 批准 / 选择」和「任务完成」两个时刻，播放自定义提示音。
默认用早期 QQ「咳咳」和「滴滴滴」两款音效，且**两个事件可以各自用不同的声音**。

## 特性

- **Codex 全自动**：通过 `~/.codex/config.toml` 的 `notify` 钩子，Codex 自己触发，无需人工干预。
- **WorkBuddy 手动触发**：由助手在合适时机调用（已写入长期记忆作为默认习惯）。
- **每个事件独立音效**：等批准用一种声音，任务完成用另一种。
- **多音效库**：随时丢新音频进去、一句话切换，不用改配置。

## 安装

```bash
# 1. 把脚本装到 PATH 下
cp qq_chime ~/.local/bin/qq_chime
chmod +x ~/.local/bin/qq_chime

# 2. 准备音效文件，放进 ~/.local/share/sounds/
#    （仓库已自带 qq-ke-ke.mp3 与 滴滴滴声音.mp3，直接 cp 即可；也可换成你自己的任意音频）
#    默认需要的文件名：
#      qq-ke-ke.mp3      任务完成音（原版 QQ 咳咳）
#      滴滴滴声音.mp3     等待批准音

# 3. 设置各事件选用的音效
printf '%s' "滴滴滴声音.mp3" > ~/.local/share/sounds/sound_approval
printf '%s' "qq-ke-ke.mp3"   > ~/.local/share/sounds/sound_turn
```

### Codex 侧（全自动）
编辑 `~/.codex/config.toml`，在 `notify`（扁平字符串数组，`[命令, 事件]` 成对）中加入：

```toml
notify = [
  "<原有的 computer-use 通知命令>", "turn-ended",
  "/Users/<你的用户名>/.local/bin/qq_chime", "approval-requested",
  "/Users/<你的用户名>/.local/bin/qq_chime", "turn-ended",
]
```

验证可解析：

```bash
python3 -c "import tomllib; tomllib.load(open('/Users/<你的用户名>/.codex/config.toml','rb'))"
```

### WorkBuddy 侧（助手手动触发）
在 `~/.workbuddy/MEMORY.md` 补一条规则：每次需要用户确认 / 批准前调用
`bash ~/.local/bin/qq_chime approval-requested`；每次任务完成时调用
`bash ~/.local/bin/qq_chime turn-ended`。

## 用法

```bash
qq_chime approval-requested   # 播放"等待批准"音效（Codex 等批准时自动跑这条）
qq_chime turn-ended           # 播放"任务完成"音效（Codex 完成时自动跑这条）
qq_chime list                 # 列出所有可用音效 + 当前各事件选用
qq_chime set approval <文件名>  # 改"等待批准"音效
qq_chime set turn <文件名>      # 改"任务完成"音效
qq_chime set all <文件名>       # 两个事件统一用同一个
```

切换即时生效，Codex 和 WorkBuddy 两端一起变（它们调的是同一个脚本、读的是同一份音效指针）。

## 说明

- 本仓库**已含**两款默认音效（`qq-ke-ke.mp3`、`滴滴滴声音.mp3`），位于仓库根目录；如需替换成自己的音频，放进 `~/.local/share/sounds/` 再用 `qq_chime set` 切换即可。
- 各事件当前音效记录在 `~/.local/share/sounds/sound_approval` 与 `sound_turn`（存的是文件名），删除则回退默认。
