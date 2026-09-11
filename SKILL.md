---
name: qq-chime
description: 用自定义音效（默认早期 QQ「咳咳」与「滴滴滴」）在 Codex / WorkBuddy 里做提醒——需要用户确认/批准时响一声，任务完成时响一声。当用户希望 AI 在等他确认或任务完成时响铃、或提到"提醒音/咳咳/滴滴滴/QQ音效"时使用。
---

# qq-chime —— 多音效提醒（Codex + WorkBuddy）

## 用途
在以下两个时刻播放提示音：
- **需要用户确认 / 批准 / 选择**（等用户输入）
- **任务完成**

支持每个事件独立音效、多音效库随时切换。

## 组件
- 脚本 `qq_chime`（本目录已附带，可直接 `cp` 到 `~/.local/bin/`）：
  - `qq_chime approval-requested` → 播"等待批准"音效
  - `qq_chime turn-ended` → 播"任务完成"音效
  - `qq_chime list` / `qq_chime set approval|turn|all <文件名>` 管理音效
  - 各事件选择记录在 `~/.local/share/sounds/sound_approval` 与 `sound_turn`（存文件名）
- 音效文件：已随本仓库提供（根目录），直接 `cp` 到 `~/.local/share/sounds/` 即可：
  - `qq-ke-ke.mp3` → 任务完成（turn-ended）
  - `滴滴滴声音.mp3` → 等待批准（approval-requested）
  如需替换，把自定义音频放进 `~/.local/share/sounds/` 再用 `qq_chime set` 切换。

## 安装步骤
1. 脚本就位：`cp qq_chime ~/.local/bin/qq_chime && chmod +x ~/.local/bin/qq_chime`
2. 放音效：`cp qq-ke-ke.mp3 滴滴滴声音.mp3 ~/.local/share/sounds/`（仓库自带，无需自备）。
3. 设指针：
   `printf '%s' "滴滴滴声音.mp3" > ~/.local/share/sounds/sound_approval`
   `printf '%s' "qq-ke-ke.mp3" > ~/.local/share/sounds/sound_turn`
4. Codex 全自动：编辑 `~/.codex/config.toml` 的 `notify`（扁平字符串数组，`[命令, 事件]` 成对）：
   ```toml
   notify = [
     "<原有 computer-use 通知命令>", "turn-ended",
     "/Users/<用户名>/.local/bin/qq_chime", "approval-requested",
     "/Users/<用户名>/.local/bin/qq_chime", "turn-ended",
   ]
   ```
   验证：`python3 -c "import tomllib;tomllib.load(open('/Users/<用户名>/.codex/config.toml','rb'))"`
5. WorkBuddy 手动触发：在 `~/.workbuddy/MEMORY.md` 补一条规则——每次需用户确认前调 `qq_chime approval-requested`、任务完成时调 `qq_chime turn-ended`。

## 切换音效
- 加新音效：丢进 `~/.local/share/sounds/`，再 `qq_chime set approval|turn <文件名>`。
- 查看：`qq_chime list`。
- 删除 `~/.local/share/sounds/sound_approval` / `sound_turn` 任一则回退默认。

## 路由判断
- 用户说"等批准/需要我确认时响" → `approval-requested` 对应音效。
- 用户说"做完了/任务完成时响" → `turn-ended` 对应音效。
- 用户给新音频文件 → 复制进 `~/.local/share/sounds/` 并用 `set` 切换，不改动脚本与 Codex 配置。
