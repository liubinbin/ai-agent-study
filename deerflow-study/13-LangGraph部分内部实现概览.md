# LangGraph 部分内部实现概览

在前面的学习中，我们了解到 deerflow 中的 Agent 是如何调用工具的，但底层究竟是如何实现的呢？本章将从 LangGraph 的内部机制出发，解析 Agent 调用工具的完整流程。

## make_lead_agent

在调用 create_agent 方法时，我们通过 tools 参数将系统可用的工具传递给 Agent。

## 模型绑定工具

在 LangChain 项目的 /libs/langchain_v1/langchain/agents/factory.py 中，_get_bound_model 方法会调用 model.bind_tools() 将工具绑定到模型上。这样，模型就能"感知"到有哪些工具可供调用。

## 模型返回 tool_calls

当为模型绑定了工具后，模型的返回结果中会包含 tool_calls 字段，用于指明需要调用哪些工具以及如何调用（包括工具名称和参数）。这是当前主流大模型通过专门训练获得的**函数调用（Function Calling）**能力。

## 包装为 AIMessage 和 ToolMessage

从大模型获取原始返回结果后，需要将其转换为系统内部的标准消息结构。以 OpenAI 为例，流程如下：

1. 调用 chat_models.py 中的 agenerate 方法，异步获取模型的原始返回结果；
2. 通过 _create_chat_result 方法生成中间结果；
3. 调用 _convert_dict_to_message 方法将结果转换为 AIMessage 或 ToolMessage。

这样，不同类型的模型返回都能被统一为系统内部的消息格式，便于后续处理。

## Agent 构建循环图（Loop Graph）

在 /libs/langchain_v1/langchain/agents/factory.py 的 create_agent 方法中，会默认构建一个状态图（State Graph），如参考资料[1]所示。在这个循环中，模型与工具之间可以反复交互：模型决定调用工具，工具执行后返回结果，模型再根据结果决定下一步动作，直到任务完成并生成最终输出。

具体构建步骤如下：

1. **L1034**：初始化一个 Graph；
2. **L1372**：添加模型节点 graph.add_node("model", RunnableCallable(model_node, amodel_node, trace=False))；
3. **L1376**：添加工具节点 graph.add_node("tools", tool_node)；
4. **L1502**：添加从工具到模型的边；
5. **L1526**：添加从模型到工具的边。

## 模型到工具的边

具体实现在 _make_model_to_tools_edge 方法中，其核心逻辑包括：

1. **判断是否需要继续执行工具**：通过 pending_tool_calls 检查是否还有未完成的工具调用，如果有，则继续路由到工具节点执行；
2. **生成最终结果**：当所有工具调用都完成后，模型会整合所有工具的输出结果，整理为最终的回复内容返回给客户端。

## 参考资料

1. [LangChain Agents 文档](https://docs.langchain.com/oss/python/langchain/agents)
