# 小助理学会通过团队完成工作

随着时间的推移，小助理做的事情也越来越复杂，正好有个新的员工加入，上级就让新员工帮助小助理做事。
小助理原来只是自己拿到任务解决问题，现在他头大了，现在事情多的做不完，而且新员工也不能闲着，得让他也干活。
在这种情况下，小助理逐渐学会了分配任务。

## deerflow 开启 subagent 模式
在 make_lead_agent 方法中，若 开启 subagent 模式，则在 tools 会加入一个特殊的 tool，名字为 task，它可以独立处理问题，意味着他和 lead_agent 一样，接收的是 prompt。
具体文件位于 /backend/packages/harness/deerflow/tools/builtins/task_tool.py。

## subagent 执行
我们关注 /backend/packages/harness/deerflow/tools/builtings/task_tool.py 文件

subagent 有两种类型，general-purpose 和 bash，具体哪一种类型是 LLM 决定，然后传递给 task_tool。

其他主要接收参数有 runtime 和 prompt。
1. 创建一个 SubagentExecutor 用于 subagent 的执行器，然后异步调用执行 prompt。
2. 在一个循环不停的接收执行器返回，回复给上一层。

具体到 SubagentExecutor 的异步执行，有两层线程池包装，分别是 _execution_pool 和 _execution_pool，这两层都是有线程个数限制的线程池。
_execution_pool 提交一个执行方法，方法内容就是往 _execution_pool 提交 future，然后带超时地获取结果。

最终实际执行方法在 /backend/packages/harness/deerflow/subagent/executor.py L208 _aexecute 方法。
1. 创建 agent，部分 middleware 和 tool 做了过滤。
2. 创建 state，继承 sandbox 和 state。
3. 通过 agent 不停返回 chunk，再返回至上层。