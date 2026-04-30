# ReActAgent 解析

在上一章节，通过 `QwenPawAgent(msg)` 方法传入请求并获取数据，下面看一下这个方法。

## 请求实际转发至 reply

通过 `QwenPawAgent(msg)` 触发了 Python 的 callable 协议。

调用了 AgentBase 的 `__call__` 方法，方法内又调用了 reply 方法。

接下来看 QwenPawAgent 的 reply 方法。

## QwenPawAgent reply 方法

1. 处理对话中的多媒体文件。
2. 处理魔法命令。
3. 添加长期记忆。
4. 调用 agentScope/src/agentscope/agent/_react_agent.py 文件 ReActAgent 的 reply 方法。

下面关注 ReActAgent 的 reply 方法。

## ReActAgent 的 reply 方法

1. 从长期记忆和 RAG 中取出相关信息，加入到短期记忆中（memory 字段）。
2. ReAct Loop。
3. 兜底生成输出结果。
4. 处理长期记忆，默认不处理。

## ReAct Loop

这个 loop 有次数限制，在一次迭代内：

1. 压缩短期记忆。
2. `_reasoning` 获取推理结果。
3. 推理结果里的 tool_use 并行使用 `_acting` 执行，获取工具结果。
4. 检测是否可结束，包括是否还有未执行的 tool_use，模型返回的结果中是否包含最终输出。

## _reasoning 方法

1. 构造 prompt，包括 system_prompt 和通过 `get_memory` 获取的 memory 内容。
2. 调用 `self.model()` 方法从模型获取调用结果。
3. 在无 TTS 时，主要是构造 msg。
4. 处理被用户打断的情况。
5. 直接返回 msg。

## _acting 方法

1. 构造 tool_res_msg。
2. 调用 toolkit 的 `call_tool_function` 执行函数，获取结果，并将结果设置到 tool_res_msg 中。
3. 获取结构化输出并返回。
4. 将输出结果加入短期记忆。

## get_memroy 方法

在 `_reasoning` 方法中，若配置文件启用了压缩，则传入 `_MemoryMark.COMPRESSED`，`get_memroy` 按 mark 获取 memory。

1. 若开启压缩，则返回除已被压缩的 message 外的内容。
2. 若不开启压缩，则返回全部内容。

## model() 方法

此方法实际调用 model 的 `__call__` 方法，以 src/agentscope/model/_openai_model.py 为例。

实际就是传入 tools、response_format 等参数，调用大模型的 client 获取结果。
