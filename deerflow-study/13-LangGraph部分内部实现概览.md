# 小助理学会通过团队完成工作

在整个过程中，还有个疑问，就是 deerflow 是怎么做到知道调用哪个 tool。

## make_lead_agent
在调用 create_agent 方法时，我们会给 tools 参数设置系统可提供 tool。

## 模型 bind tool
在 langchain项目 /libs/langchain_v1/langchain/agents/factory.py 的 _get_bound_model 方法，会调用 model.bind_tools 绑定工具。

## 模型返回带 tool_calls
在目前大模型中，若为 model 绑定 tools，在返回结果会有 tool_calls，代表需要调用的工具，及怎么调用工具。猜测这个是大模型特殊训练过的功能。

## 包装为 AIMessage 和 ToolMessage
从大模型获取到返回结果，需做准换变更为系统的结构。
以 openai 为例。
1.通过调用 chat_models.py 的 agenerate 方法从模型获取返回结果。
2.通过 _create_chat_result 方法生成转换。
3.调用 _convert_dict_to_message 方法转换为 AIMessage 或 ToolMessage。

## agent 构建 loop 循环
在 /libs/langchain_v1/langchain/agents/factory.py 的 create_agent 方法中，会默认构建的一个 graph，如资料[1]，在这个 loop 中，实现了 model 不停调用 tools，若判断完成，则构造输出。
1.L1034 构建一个 graph.
2.L1372 添加一个 model 节点，graph.add_node("model", RunnableCallable(model_node,amodel_node,trace=False))
3.L1376 添加一个 tool 节点 graph.add_node("tools",tool_node)
4.L1502 添加一个 tool 到 model 的边，L1526 添加一个 model 到 tool 的边。

## model 到 tool 的边
具体方法 def_make_model_to_tools_edge。
1.继续执行 tools：通过 pending_tool_calls 获取剩余 tool，判断是否要继续执行 tool。
2.最终返回给客户端的信息是由模型判断整合全部tool输出结果，然后整理为最终结果。

## 参考资料
1：https://docs.langchain.com/oss/python/langchain/agents
