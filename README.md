# dsh-cross-session-agent（DSH 0.2 适配 fork）

[English](README.en.md) | 中文

非官方插件，基于 [dd2673/dsh-cross-session-agent](https://github.com/dd2673/dsh-cross-session-agent) 的 MIT 源码。本 fork 针对 **DSH 0.2.0-rc.2、Node 24** 修复回执、前端和跨会话授权。没有声明兼容其他 DSH 版本。

适用于用户明确授权的 A 执行 → B 审查 → A 整改 → B 复审。四个 Host 工具共用具体 Session ID 配对及相同 `cwd` 的权限边界。**未配置配对时默认不能通信。** 普通 fork 可作为独立成员；真正的 subagent、自身、归档和跨工作目录目标被拒绝。

## 安装与配置

在自己的 profile 中安装本 fork，建议先在隔离的 `DSH_HOME` 验证：

```sh
dsh plugin --profile desktop add 'github:lilongsky/dsh-cross-session-agent#fork-v0.1.3' --ignore-scripts
```

桌面版使用 `desktop`；Web 界面使用 `web`。本机实测已安装在桌面 profile。

安装会通过 `cordis.patch.yml` 加载插件，但不会自动授权任何配对。在该 profile 的 `cordis.patch.yml` 加入以下配置（将 ID 换成两条真实会话的完整 ID）：

```yaml
- id: cross-session-agent
  config:
    allowedPairs:
      - id: review-task-001
        sessions: [session-A完整ID, session-B完整ID]
        maxMessages: 20
```

`id` 是本次协作的标识，允许 1–64 个英文字母、数字、下划线或连字符。`sessions` 必须是两个不同的稳定 Session ID；标题不能作为身份。每个配对和协作 `id` 只能配置一次。`maxMessages` 是双向消息总数上限，默认 20，允许 1–100；不是整改轮次。如果每轮往返两条，20 条约为 10 次往返。预算从两条会话自己的原生 Inbox 日志计算，进程或插件重启不会清零。并发投递按配对串行检查，另保留双向合计每 60 秒最多 10 条的进程内限流。

下一项独立任务由用户配置新的 `id`；模型不能通过工具修改配置或重置预算。任务结束时删除该配对配置即可撤销授权。两条会话各自的工具审批和权限仍由 Harness 管理。

## 复制 Session ID

侧栏会话行的 **“…” → “复制会话 ID”** 会复制该行的完整稳定 ID；无需先打开该会话。会话顶部也保留“复制会话 ID”按钮。标题不是身份，复制 ID 不会授予通信权限。

## 使用

在 A、B 各自的任务指令中授予明确的编排职责。例如：

- A：“完成任务后把改动和验证结果发给 B；收到 B 的审查意见后整改并再次交给 B。最多整改 3 轮，超限或阻塞时停止并报告。”
- B：“审查 A 的结果。有问题时把具体意见发回 A；复审通过后报告通过并结束，不发送仅表示‘收到’的消息。”

配置配对仅授予工具访问能力，不能代替任务指令中的用户意图。`@会话` 和客户端候选列表只是目标引用；Host 会再次检查授权，未配对的引用不会获得通信权限。

| 工具 | 行为 |
| --- | --- |
| `list_peer_agents` | 只发现调用方已授权、同工作目录、未归档的 peer |
| `send_agent_message` | `followup` 排队下一 turn；离线时用公开 `resume` 恢复；`steer` / `inject` 仅用于 running 目标 |
| `check_delivery` | 只查调用方自己发给授权 peer 的消息；支持指定 `messageId` 后跨重启查询 |
| `get_peer_context` | 只读受限的 `overview` / `conversation` / `activity`，不恢复或唤醒目标 |

回执 `accepted`、`pending`、`claimed`、`discarded` 只表示传输事实，不能证明已读、回复或任务完成。业务通过以 B 的明确审查结果为准。查询在线和离线目标都使用公开 `sessionQuery`；fork 继承的父会话 Inbox 事件不参与其自己的回执。

## 隐私与生命周期

插件运行时代码不发网络请求、遥测，不读取凭据文件，不执行 shell，不建立独立数据库或邮箱。消息保留真实发送方 ID 和 `dsh-cross-session-agent` 来源；不会伪装成用户批准。客户端使用 React 和 `textContent`，支持 DSH 0.2 的图标导出，并保留旧 `dsh-agent-message` 日志的只读显示。

上下文白名单仅包括直接用户文字、规范压缩摘要、已完成助手文字，以及受限的 turn/tool 状态。排除 reasoning、工具参数和结果、错误正文、系统提示、插件/relay 消息、审批和附件。错误码只保留最多 64 字符的安全符号，否则返回 `TOOL_ERROR`。**允许的正文没有秘密内容扫描；如果用户或助手把凭据写入可读正文，授权 peer 仍可能读到。** 不应在会话正文中放秘密。[PRIVACY.md](PRIVACY.md) 给出准确边界。

取消信号在异步查询前后、恢复与原生投递前检查；归档和身份在恢复 setup、发布 commit 和投递前复查。已经被 Harness 接受的消息不因发送方随后停止而撤回。冷恢复本身可能接续目标既有 Inbox；调用方仍必须有用户授予的协作职责。插件持有恢复 handle 至卸载，避免另建生命周期；失效投递会释放自己新创建的 handle。

## 验证

```sh
pnpm install --frozen-lockfile --ignore-scripts
pnpm test
# 使用自己的 DSH CLI；脚本会创建全新的临时 DSH_HOME，启动后再重启隔离 profile。
node scripts/live-profile-e2e.mjs /path/to/dsh
```

隔离验收使用真实 DSH profile、Agent、工具派发、SessionQuery 和 JSONL 持久化；模型由无网络的确定性测试提供者替代。脚本不接受现有服务器地址，也不会向个人会话投递。详见 [TDD_CONTRACT.md](TDD_CONTRACT.md)。测试不证明任意在线模型一定遵循协作流程，也不是对未来代码版本的安全保证。

本 fork 保留 MIT License 和 [NOTICE.md](NOTICE.md) 中的上游来源。没有向同名 npm 包发布权限；fork 使用 `fork-v*` 标签提供已测版本，不触发上游按 `v*` 标签运行的 npm 发布工作流。

[0.1.3 验证结果 / verification results](docs/validation-0.2.0-rc.2.md)
