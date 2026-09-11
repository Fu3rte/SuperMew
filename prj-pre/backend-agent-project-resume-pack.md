# RagProof——RAG 质量治理与评测平台：简历规格与落地计划（agent-only × safe-mode）

> 生成日期：2026-09-09
> 推荐模式：agent-only（safe-mode 落地口径）
> 用户画像：Python 候选人，前端 React+TS、后端 Python/MySQL/Redis、Docker（学过）、LangChain/LangGraph、Agent 评测/RAG 评测、RAG。
> 项目名：RagProof（知鉴）｜参考基线：开源项目 SuperMew（MIT，已本地源码验证）
> 可信度边界：SuperMew 是**开源参考项目**（MIT，已本地源码验证），不是用户已完成的成果。下文的"负责功能 / 技术难点"是**复现并完成改造后的目标口径**；完成前不得以"已实现"口吻写入简历，当前可写的是"建议简历功能点（完成对应改造后可写）"。

## 结论先行

- **推荐项目**：RagProof（基于 `icey1287/SuperMew` 二次开发）——知识库优先、面向真实运行与评测的 Agent 平台，LangGraph + LangChain + RAG + 版本化 RAG 评测门禁。
- **模式解释**：agent-only 要求完整业务型 Agent 项目，safe-mode 要求依赖可控、7 天可落地。SuperMew 用 `uv` + `docker-compose` 起（Postgres + Redis + Milvus + etcd/MinIO），仓库仅 6.1MB、580 文件、853 star、MIT，符合"能快速跑通 + 有真实改造空间"。
- **定位分工**：SuperMew 主打**评测治理**（评测结果可复现、可审计、可卡发布），与用户另一个"业务 Agent harness（上下文压缩 / 自动重试）"项目形成"**运行时内核 vs 质量门禁**"的清晰对照；本项目刻意不写 harness 中间件、上下文压缩、Provider 重试，避免两个项目撞卖点。
- **选择原因**：四个需求里，RAG、RAG 评测、LangChain 生态**全部命中且评测最完整**；多智能体为"并行子问题 + Grader 路由"的**半成品**，正好留作 safe-mode 的改造空间。

## 业务需求与痛点（为什么这个项目存在）

**一句话**：企业 RAG 上线前后，怎么证明"这次调参是真变好"，并且不让变差的版本发出去。

**用户不是 C 端**，是企业里做 RAG 应用的 AI 团队（RAG 工程师 / 技术 lead / 值班运维）。他们每天都在改 chunk 大小、切分策略、embedding 模型、top_k、混合检索权重、rerank 开关与阈值、Answer/Grader/Evaluator 用哪个模型，以及重建索引。

| 痛点 | 现状 |
| --- | --- |
| 调参靠人肉 | 改一个参数，手工问三五条问题，凭感觉判断"好像好点了" |
| 结论不可复现 | 没有固定测试集，换个人再测结论对不上 |
| 假对比 | 换了模型/索引后拿旧结论比，得出"变好了"的错觉 |
| 上线没门禁 | 改坏了靠 code review 拍脑袋，没人拦 |
| 出错无法归因 | 答错了不知道是没召回、重排排掉了，还是证据不足硬答 |
| 评测本身不可信 | LLM 当裁判分数飘；或手改数据美化指标 |
| 出事无证可查 | 拿不出"当时用哪套配置、哪个索引、哪份数据、什么指标" |

**一个真实场景**：把 rerank 阈值调高一点，手工测 5 条感觉更好就上线，结果多跳问题变差——单事实问答没受影响，人肉测试根本发现不了。SuperMew 的做法是跑版本化数据集、真实执行 RAG、算确定性指标，发现 `document_recall_at_10` 从 0.92 掉到 0.81、超过 `max_regression: 0.02`，直接让门禁 FAILED、卡住 PR，并在报告里点名回退的 case。对应源码：`evals/rag/live_gates_v1.json` + `backend/evaluation/rag.py::_evaluate_gates`。

**与普通 RAG 项目的区别**：普通项目解决"能问答"；本项目解决"问答质量**可证明、可管控、可发布**"——调参从人肉试变成数据集 + baseline + 门禁，结果从看感觉变成 provenance + 指纹 + 确定性指标，上线从拍脑袋变成 CI 卡口，出问题从说不清变成报告可追溯。

**简历口径**：面向企业 RAG 上线质量治理，把检索配置和模型调整从"人肉试错 + 口头确认"变成"版本化评测 + 回归门禁 + 可审计报告"。

## 项目候选池（agent-only × safe-mode）

| 项目 | 链接 | 分桶 | 模式匹配 | 定位 | 推荐度 | 取舍理由 |
| --- | --- | --- | --- | --- | --- | --- |
| icey1287/SuperMew | https://github.com/icey1287/SuperMew | 企业知识库 Agent / RAG 评测 | agent-only + safe-mode | 稳妥落地 + 差异化评测治理 | ★★★★★ | LangGraph 多智能体 + 完整 RAG + 版本化评测门禁；6.1MB、MIT、uv/compose |
| xerrors/Yuxi | https://github.com/xerrors/Yuxi | 企业知识库 Agent / 多智能体 | agent-only | 平台业务型 | ★★★★ | 业务闭环最强，但 5+ 基础设施组件，且 middlewares 与用户 harness 项目撞卖点 |
| skygazer42/MimirQ | https://github.com/skygazer42/MimirQ | 企业 RAG / RAG 评测 | agent-only | RAG 评测型 | ★★★ | 评测门禁完整，但 torch+Milvus+MinIO、392MB，safe-mode 冲突 |
| 1517005260/graph-rag-agent | https://github.com/1517005260/graph-rag-agent | 多智能体 / GraphRAG | agent-only | 多智能体型 | ★★★ | planner/executor/reporter + 自研评测，但需 Neo4j + GraphRAG 建索引，2025-11 停更 |
| liangdabiao/langgraph_multi-agent-rag-customer-support | https://github.com/liangdabiao/langgraph_multi-agent-rag-customer-support | 客服自动化 | agent-only | 轻量落地 | ★★ | 最轻的 LangGraph 多智能体 RAG，但无 License、无 RAG 评测 |
| Ricky-7-Yan/intelligent-audit-system | https://github.com/Ricky-7-Yan/intelligent-audit-system | 审计 Agent | — | — | 淘汰 | 评测 harness 强，但用 OpenAI SDK + TF-IDF，**非 LangChain 生态** |
| ggozad/haiku.rag | https://github.com/ggozad/haiku.rag | 本地 Agentic RAG | — | — | 淘汰 | Pydantic AI 非 LangGraph，偏库/CLI |
| Zleap-AI/SAG | https://github.com/Zleap-AI/SAG | 知识库应用 | — | — | 淘汰 | 自研检索架构 + 桌面应用，非 LangChain 生态 |
| vibrantlabsai/ragas / promptfoo / opik / langfuse / giskard / AutoRAG / future-agi | — | 评测框架/平台 | — | — | 淘汰 | 纯框架或基础设施平台，不是业务型 Agent 系统 |
| nicoladisabato/MultiAgenticRAG / didilili/deepsearch-agents | — | 多智能体 RAG | — | — | 淘汰 | 教程仓库，无评测、无业务闭环 |

## 多样性说明

- **搜索覆盖**：脚本 9 组桶查询（75 仓库）+ 定向搜索 22 组，覆盖多智能体、企业知识库 Agent、RAG 评测、GraphRAG、深度研究、评测 harness、可观测平台等方向；star 已按用户要求从 300 下探到 200。
- **分桶策略**：agent-only 下按"评测治理 / 平台业务 / RAG 评测 / 多智能体 / 研究报告 / 客服自动化"分桶，避免只取全局高星。
- **模式约束**：只保留完整业务型 Agent，剔除纯框架（ragas/promptfoo/opik/langfuse/giskard/AutoRAG）、纯平台（future-agi）、教程仓库（MultiAgenticRAG/deepsearch-agents）、非 LangChain 项目（SAG/haiku.rag/AuditPilot）。
- **过热降权**：34k/24k/21k star 的评测平台与知识平台不因 star 加分；1k 以上不再额外加权。
- **排除说明**：未纳入 IoT / 硬件 / 设备管理类项目。
- **本次为什么选 SuperMew**：它是唯一同时满足"LangGraph 多智能体 + 完整 RAG 链路 + 版本化 RAG 评测与 CI 门禁"且体量适合 safe-mode 的项目。

## 可替换项目

- 想做**平台业务型**（多租户 / 知识图谱 / 权限）→ 换 `xerrors/Yuxi`。
- 想做**RAG 评测型**（质量门禁 / 混合检索）→ 换 `skygazer42/MimirQ`。
- 想做**多智能体型**（planner/executor/reporter）→ 换 `1517005260/graph-rag-agent`。
- 只想**最轻量**地讲 LangGraph 多智能体 RAG → 换 `liangdabiao/langgraph_multi-agent-rag-customer-support`（注意无 License）。

---

## 推荐项目：RagProof（基线 `icey1287/SuperMew`）

- **项目定位**：稳妥落地 + 差异化（评测治理型业务 Agent 平台）
- **链接**：https://github.com/icey1287/SuperMew
- **适合人群**：目标智能体 / AI 应用实习，已有 LangChain/LangGraph 基础，想用一个"结果可信"的项目区别于普通 RAG 问答
- **为什么适合写简历**：
  - 有完整业务闭环：知识库入库 → RAG 问答 → 人工澄清（HITL）→ 评测 Job → 门禁报告
  - 有真实工程纵深：Postgres + Alembic 迁移、租约 + fencing token、事件流、Provider 失败语义、Docker 沙箱
  - 有强差异化：**版本化 RAG 评测 + provenance + 回归门禁 + CI**，不是"跑几个指标"
- **已有能力**（源码已验证）：见下方"代码验证摘要"

### 代码验证摘要

- 拉取脚本：`references/pull_github_repos.py`
- manifest：`repo-source-manifest.json`（`status: cloned`，`all_ok: true`）
- 本地仓库：`.repo-source-cache/icey1287__SuperMew`
- commit：`e2ebabe81e066e0498bdb58000942ef5199d338c`（main，2026-09-07）
- 已阅读的关键源码 / 配置 / 测试：
  - `backend/rag/pipeline.py`（1526 行）：`build_rag_graph`、`classify_complexity`、`_fanout_sub_questions`（LangGraph `Send`）、`rag_sub_agent`、`synthesis`、`grade_documents_node`、`rewrite_question_node`、`await_hitl_node`（`interrupt`）、`resume_rag_state`
  - `backend/rag/reranking.py`、`backend/rag/utils.py`、`backend/documents/retrieval.py`：`RerankStage`、`dedupe_documents`、`_auto_merge_candidates`、`resolve_retrieval_snapshot`、`RetrievalTarget`/`DocumentRetrievalScope`
  - `backend/evaluation/rag.py`（1371 行）：`RagEvalDataset`、`RagEvalObservation`、`RagEvalGatePolicy`、`RagMetricGate`、`evaluate_rag`、`_score_case`、`_aggregate_metrics`、`_evaluate_gates`、`_evaluate_metric_gate`
  - `backend/evaluation/worker.py`、`backend/evaluation/repository.py`（1131 行）：`claim_next`（`with_for_update(skip_locked=True)` + `fencing_token += 1`）、`heartbeat`、`claim_case`、`complete_case`、`finish_job`、`_assert_owner`
  - `backend/evaluation/service.py`、`backend/model_control/contracts.py`：`create_job` 冻结 `runtime_snapshot(required_roles=frozenset(ModelRole))`、`ModelCatalogSnapshot(frozen=True)`
  - `backend/runs/repository.py`（1371 行）、`backend/runs/cancellation.py`、`backend/runs/resume.py`：`reserve`（idempotency_key + `hash_run_request`）、`claim`、`heartbeat`、`_assert_fencing`、Redis 取消广播 + durable 状态、`consume_resume`
  - `scripts/evaluate_rag.py`（409 行）：`validate`/`score`/`run`、`--fail-on-regression`、`provenance`（`contract_smoke` / `live_rag`）、`_report_metadata`、`_require_comparable_source`、`_validate_release_policy`
  - `evals/rag/`：`rag_smoke_v1.json`（20 例，含单事实/跨文档/多跳/歧义 HITL/冲突/注入标签）、`gates_v1.json`、`live_gates_v1.json`、`baseline_v1.json`、`schema/{dataset,gates,observations,report}_v1.schema.json`
  - `.github/workflows/rag-eval.yml`、`rag-eval-live.yml`；`tests/test_rag_evaluation*.py`、`test_rag_eval_cli.py`、`test_rag_fault_injection.py`、`test_rag_latency_guards.py` 等 16 个 RAG/评测测试文件
  - `pyproject.toml`：`langchain==1.3.9`、`langgraph>=1.2.4,<1.3`、`langgraph-checkpoint-postgres`、`pymilvus`、`alembic`、`redis`、`sentence-transformers`

### 简历写法

#### 项目简介

RagProof 基于开源 Agent 平台 SuperMew 二次开发，搭建企业 RAG 质量治理与评测平台，围绕知识库问答的真实运行链路，把语料入库、混合检索、证据评判、人工澄清、评测任务与发布门禁串成一条可追溯流程，让每次检索配置或模型调整都能用版本化数据集和回归门禁验证，最终产出可审计的评测报告，替代工程师人肉试跑和口头确认。

#### 负责功能 / 技术难点

（复现并完成改造后的目标口径，完成前不得写入简历）

1. 编排 RAG 问答主链路：用 LangGraph 把复杂度判定、子问题拆解、检索、证据评判、一次重写与回答编成状态图，简单问题走"检索→评判"直通，复杂问题用 `Send` API 把 2-4 个子问题扇出给子 Agent 并行执行"检索+评判"，再在合成节点按 `chunk_id` 去重、合并 RRF 排名并归一化子链路 trace；单个子问题 Provider 失败只返回结构化失败快照，全部失败才向上抛错，避免一个子任务拖垮整条问答链路。

2. 混合检索链路落地：向量召回与 Milvus 原生 BM25 双路取回后，用 RRF 融合、按父子块关系做 auto-merging、可选 Rerank 并设置候选数量与字符预算；重排超时或失败时按 `rerank_fallback_applied` 降级回融合结果并把降级原因写入 `rag_trace`，保证检索链路可观测、可复现，而不是静默退化。

3. 澄清与断点恢复：歧义问题交给 LangGraph 原生 `interrupt` 挂起，Run 落到 `waiting_input` 并持久化 checkpoint；用户提交一次性 `hitl_token` 与幂等键后从同一节点恢复，把澄清内容拼回原问题、重跑检索并写入 `hitl_resumed` 标记；token 消费走单事务 `consume_resume`，重复提交命中幂等返回同一 checkpoint，防止重复触发和越权恢复。

4. RAG 评测契约与发布门禁：把评测拆成版本化 Dataset / Observation / GatePolicy / Report 四类契约，Dataset 改动即指纹变化、旧 Observation 直接拒绝；指标覆盖 recall@k、MRR、NDCG、document recall、HITL 命中率、provider 失败率等，门禁同时校验 provenance（`contract_smoke` 与 `live_rag` 不得混用）、关键用例零回归、逐指标阈值与最大回退幅度，发布策略强制 `critical_no_regression` + `required_provenance` + top-10。

5. 评测任务租约与故障收敛：持久化 Evaluation Worker 用 `FOR UPDATE SKIP LOCKED` 领取 Job，写入 `owner_worker_id`、`lease_expires_at` 并递增 `fencing_token`，独立心跳线程续租；每个 Case 也单独 claim / complete，心跳失败或所有权失配时拒绝写入；任务异常时生成 partial report 落库、取消请求先落 durable 状态再收敛，保证过期 worker 无法覆盖评测结果。

6. 模型快照与报告可追溯：创建评测任务时冻结 Answer / Fast / Grader / Evaluator 四类角色的模型目录快照，运行中修改控制面不影响在途任务；报告元数据绑定语料指纹、profile / index 指纹与 RAG 源码指纹，并校验 baseline 来源身份一致，不一致直接拒绝比较（除显式 override），避免"换了模型或索引还拿旧 baseline 比"的假回归结论；离线门禁在 PR 上重算 baseline 做 `cmp` 比对并按 base 分支做回归卡口，真实质量门禁则用自托管 runner 跑 live 评测。

#### 建议简历功能点（完成对应改造后可写）

- 把并行子问题升级为**有预算与恢复语义的多步骤专业 Agent 协作**：为每个子 Agent 定义工具白名单、步数/预算上限与失败恢复语义（README 路线图已列，属于天然改造点）。
- 评测失败归因聚合：按超时、指标不达标、Provider 失败、Case 缺证分类聚合，输出趋势报表与失败 Top-N。
- 评测任务化：语料或索引更新自动触发重测，指标漂移超阈值时告警，把一次性评测升级为持续评测。
- 多租户评测配额池：复用现有 lease / fencing 机制做租户隔离与并发配额。
- 沙箱审计日志：记录每次命令、挂载路径与资源占用，越权或超额请求写入告警事件。
- Holdout 与开发集指标对比报告导出，支持发布前复核。

### 可改造方向

- **多智能体增强**：从"并行子问题"升级到 planner / researcher / writer / reviewer 角色协作，保留现有预算与 trace 契约。
- **评测平台化**：评测 Job 增加调度队列、优先级、配额与失败重试策略。
- **可观测性**：评测报告接入趋势看板、指标漂移告警、失败归因聚合。
- **业务化包装**：面向"企业 RAG 上线前调参"场景，增加配置候选、审批与发布记录。

## 落地计划（7 天，safe-mode）

| 天 | 任务 | 验收 |
| --- | --- | --- |
| D1 | 跑通 `docker-compose` + `uv sync`；通读 `pipeline.py` / `evaluation/rag.py` / `worker.py`，画出 RAG 状态图与评测数据流 | 本地能问答一次 + 跑通 `scripts/evaluate_rag.py score` |
| D2 | 理解评测契约：Dataset/Observation/GatePolicy/Report + schema + fingerprint | 手改 Dataset 触发指纹变化并被拒绝 |
| D3 | 复现离线门禁：重算 baseline、base 分支回归、`--fail-on-regression` | 本地模拟一次指标回退被卡住 |
| D4 | 复现评测 Worker：claim / heartbeat / fencing / partial report | kill worker 后 Job 能收敛，旧 worker 无法写入 |
| D5 | 复现 HITL：interrupt / resume / 幂等 token | 澄清后同一 Run 恢复并重跑检索 |
| D6 | 改造点 1：子问题 → 多步骤专业 Agent 协作（加预算/恢复语义） | 复杂问题走新协作路径且 trace 完整 |
| D7 | 改造点 2：失败归因聚合 + 趋势报表；简历条目定稿与自检 | 评测报告带失败归因视图 |

升级项（待用户拍板）：前端评测工作台增强（React 栈已具备）、评测任务自动触发（语料更新 → 重测）。

## 面试防御与边界

- **必答三问**：① 为什么说评测结果是可信的——版本化契约 + 指纹 + provenance + 门禁，LLM 不当判官；② 过期 worker 怎么防——lease + fencing_token + 单事务 ownership 校验；③ 多智能体做到哪一步——目前是并行子问题 + Grader 路由，角色协作是规划中的改造点，不吹成已实现。
- **命门红线**：指标由确定性代码计算，LLM 只做 Judge 维度且可被离线断言覆盖；不能把 LLM 评分说成唯一判据。
- **诚实边界**：RagProof 基于 MIT 开源项目 SuperMew 二次开发，SuperMew 不是用户原创；简历要写"基于/复现并改造"，改造点必须真做。
- **防线**：每条功能点都要能追到实现细节（租约字段、fencing_token、gates 阈值、provenance 取值、RRF/降级码），答不出细节的条目不写。

## 最终建议

- 用 RagProof 做"**评测治理**"项目，和另一个"业务 Agent harness（上下文压缩 / 自动重试）"项目配对：一个管 Agent 怎么跑，一个管 Agent 跑得对不对、敢不敢上线。
- 时间只剩 3 天时，保 D1–D4（跑通 + 评测契约 + 离线门禁 + Worker 租约），砍多智能体改造与前端工作台叙事。
- 若后续想补"业务平台"维度，再考虑 Yuxi，但注意它和 harness 项目的卖点重叠。
