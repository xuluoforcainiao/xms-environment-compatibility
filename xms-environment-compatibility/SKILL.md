---
name: xms-environment-compatibility
description: XMS 自动化任务运行环境兼容性验证参考。记录在不同屏幕状态（常亮/锁屏/息屏）和浏览器状态下（Chrome 打开/关闭/Extension 断开）的任务执行验证结果，以及已知问题和规避方案。适用于评估 XMS 定时任务在不同运行环境下的可靠性。
---

# XMS 运行环境兼容性验证表

## 已验证环境

| 环境 | 状态 | 验证时间 | 说明 |
|------|------|---------|------|
| 屏幕常亮 + Chrome 已打开 | 正常 | 2026-05-10 15:00 | 首次定时任务，SSO 登录、导出、推送全流程成功 |
| 屏幕常亮 + Chrome 完全关闭 | 正常 | 2026-05-10 21:28 | Agent 检测到无浏览器，自动调用 `tabs_context_mcp` 启动新 Chrome，全流程成功 |
| Win+L 锁屏 + Chrome 已打开 | 正常 | 2026-05-10 16:00 | 锁屏状态下执行，浏览器 DOM 操作不受影响，全流程成功 |
| Win+L 锁屏 + Chrome 完全关闭 | 未明确验证 | — | 推断与「屏幕常亮 + Chrome 关闭」相同，Agent 会自动启动浏览器 |
| 纯息屏（显示器关闭，未锁屏） | 未验证 | — | 仅验证了 Win+L，未单独验证显示器关闭场景 |

## Chrome Extension 连接状态

| 状态 | 结果 | 验证时间 | 说明与处理方案 |
|------|------|---------|--------------|
| Chrome 打开 + Extension 正常连接 | 正常 | 多次验证 | 理想状态，直接执行 |
| Chrome 打开 + Extension 断开（`Extension not connected`） | **失败** | 2026-05-10 20:55 | CDP WebSocket 连接后立即断开（code=1005）。**仅重新加载 Extension 无效，必须完全关闭 Chrome 后重开** |
| Chrome 完全关闭 | 正常（自动恢复） | 2026-05-10 21:28 | Agent 四级恢复策略会检测到无浏览器，自动启动新 Chrome 实例 |

## 环境对比与注意事项

| 对比项 | Win+L 锁屏 | 纯息屏（未锁屏） |
|--------|-----------|----------------|
| 系统状态 | 用户会话锁定，需要密码解锁 | 仅显示器关闭，用户会话仍活跃 |
| 后台进程 | 继续运行 | 继续运行 |
| 浏览器 MCP | 可用（已验证） | 推断可用（未验证） |
| 建议 | 生产环境可用 | 如需使用，建议先单独测试一次 |

## 已知问题与规避方案

| 问题 | 根因 | 规避方案 |
|------|------|---------|
| `Extension not connected` | Chrome 长期运行后 Extension 与 CDP 的 WebSocket 连接断开 | **完全关闭 Chrome 再重开**，不要仅重新加载 Extension |
| PowerShell 屏幕熄灭脚本失败 | ConstrainedLanguage 模式阻止 `Add-Type` | 使用 Python ctypes：`python -c "import ctypes; ctypes.windll.user32.SendMessageW(-1, 0x0112, 0xF170, 2)"` |
| 窗口极小（256x116） | 浏览器长时间闲置后窗口自动缩小 | JS DOM 操作不受窗口大小影响；`computer` 坐标点击会受影响，需优先用 JS |

## 生产环境建议

- **Chrome 状态**：任务执行前保持 Chrome **完全关闭**，让 Agent 自动启动干净实例，避免 Extension 断开风险
- **屏幕状态**：Win+L 锁屏已验证可用，可放心使用；纯息屏建议额外测试一次
- **恢复能力**：Agent 内置四级浏览器恢复策略（JS resize → 新建标签页 → 重建窗口 → 报错重试），即使浏览器异常也能自动恢复
