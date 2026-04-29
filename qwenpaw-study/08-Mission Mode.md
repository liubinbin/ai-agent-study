# Mission Mode

在 QwenPaw 的描述中，Mission Mode 时一个专门为的长期、复杂任务设计的自主执行模式。
把复杂任务拆解为多个故事，通过 master -> worker -> verifier 的流水线，确保质量和可靠性。
在执行时，会把整个任务分解为两个阶段，阶段一：prd 生成。阶段二：执行流程。

## 使用方式
- 创建任务： /mission <任务描述>
- 查看进度： /mission status
- 列出所有：/mission list

## 请求解析
在 AgentRunner 的 query_handler 方法。
1. 先使用 maybe_handle_mission_command 尝试解析参数。
2. 补充一些 prompt 。
3. 根据 mission_phase 参数执行 src/qwenpaw/agents/mission/mission_runner.py 的 run_mission_phase1 或 run_mission_phase2

## handle_mission_command 方法
1. 根据 WORKER_PROMPT_TEMPLATE 构建 worker prompt，指定 worker 自主独立的完成一个 story。
2. 根据 VERIFIER_PROMPT_TEMPLATE 构建 verifier prompt，指定 virfier 做好任务的验证工作。
3. 根据 MASTER_PROMPT，把所有 prompt 整合为一个 master prompt，让 master 先生成 prd.json，然后作为管理者监督 worker 完成任务并通过 verifier。

## run_mission_phase1
1. 通过 agent 执行 master prompt。
2. 检测 prd 文件是否有问题。
3. 若用户通过，则执行 run_mission_phase2。

## run_mission_phase2
1. 读取 prd 检测一下。
2. 更新状态为 execution。
3. 从设置 controller 的 tool 无效，不能备 worker 使用，tool 指 IMPLEMENTATION_TOOLS。
4. _remaining_summary 构造执行任务 msg，计算未结束的 story 个数，指示通过通过 worker-> verifier 流水线去处理。
5. 在执行外层抱一个迭代次数。
6. 通过 agent 执行 msg，则给提示全部通过，然后更新状态为 completed。
7. 若超过迭代次数，则完成结果，更新状态为 max_iterations_reached

## 主要文件
1. prd.json ：计划文件
2. progress.txt：保存故事进度
3. loop_config.json：用来在多次对话轮次之间保持和传递 mission 的状态。

## 参考资料
- 1：https://qwenpaw.agentscope.io/docs/commands#Mission-Mode---%E5%A4%8D%E6%9D%82%E4%BB%BB%E5%8A%A1%E8%87%AA%E4%B8%BB%E6%89%A7%E8%A1%8C
