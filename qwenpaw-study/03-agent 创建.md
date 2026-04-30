# agent 创建

学习 QwenPaw，我们首先需要找到 agent 的创建入口，在那里可以串联起外层请求和 agent 内部的 skill、prompt 等信息。由于外层请求处理暂时不是我们的关注重点，所以我们从请求入口开始。

## 请求入口

在外层选定 agent 后，把请求转发至该 agent，此 agent 有自己独立的 md 文件（AGENTS.md / SOUL.md / PROFILE.md）。

文件入口：src/qwenpaw/app/runner/runner.py L349 query_handler 方法。

## 处理是否有被用户拒绝

在 L369 执行 `self._resolve_pending_approval(session_id, query)`，这方法应该会和后续的某个插件做关联，此处暂时不深入。

## 初始化 QwenPawAgent

在 L446 做了初始化，传入 agent_config、env_context、mcp_clients、memory_manager、request_context、workspace_dir、task_tracker。

其中 agent_config 是此 agent 的配置，因为 QwenPaw 支持多 agent 配置，所以每个 agent 都有独立的配置，使用时需要加载各自的配置。

memory_manager 的实现是 ReMeLightMemoryManager，详见 src/qwenpaw/app/workspace/workspace.py L36 `def _resolve_memory_class` 方法，此方法待后续深入看看。

## 处理 skill

在初始化 QwenPawAgent 时，L134 `toolkit=self._create_toolkit(namesake_strategy=namesake_strategy)` 获取所有内置 tool。

然后在 L137 `self._register_skills(toolkit)` 把所有 tool 注册到 agent 中，注册信息包括 name、desc、dir。

还需要处理以 `/` 开头的魔法命令。

## 处理 state

L523 `self.session.load_session_state` 从文件中读取 session 信息，放入 agent 中。这部分不知道是不是用于异常恢复，待以后验证。

## 处理 system_prompt

在初始化 QwenPawAgent 时，L140 `sys_prompt=self._build_sys_prompt()` 方法读取所有 md 文件（AGENTS.md / SOUL.md / PROFILE.md），构造 system_prompt。

初始化后，又调用一次 `self._build_sys_prompt()` 重新构造 sys_prompt，然后将 sys_prompt 写入 memory。

## 通过 agent 处理 msg

最后通过：

```python
stream_printing_messages(
    agents=[agent],
    coroutine_task=agent(msgs),
)
```

让 agent 处理请求。
