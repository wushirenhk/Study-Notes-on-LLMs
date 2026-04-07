# 腾讯 Agent 研发岗位面试问题与回答

---

## 一、Agent 架构与设计

### 1.1 介绍 auto deployer 的实现难点

**Auto deployer** 是指自动化部署系统，核心难点包括：

1. **环境一致性**：开发、测试、生产环境差异导致部署失败，需要容器化或环境标准化
2. **依赖解析**：复杂依赖树需要正确解析顺序，循环依赖检测
3. **回滚机制**：部署失败时需要快速回滚到稳定版本
4. **状态管理**：分布式环境下服务状态同步困难
5. **灰度发布**：如何平滑切换流量，金丝雀发布策略

```python
# 简化版 Auto Deployer 核心逻辑
class AutoDeployer:
    def deploy(self, service: str, version: str) -> DeploymentResult:
        # 1. 健康检查
        if not self.health_check(service):
            raise DeploymentError("Service unhealthy")
        # 2. 部署新版本
        self.deploy_new_version(service, version)
        # 3. 验证
        if not self.verify_deployment(service):
            self.rollback(service)  # 回滚
            raise DeploymentError("Verification failed")
        # 4. 切换流量
        self.switch_traffic(service, version)
        return DeploymentResult(success=True)
```

---

### 1.2 介绍 motion graphic agent 是什么，架构是什么

**Motion Graphic Agent** 是处理动画/ motion graphics 制作的 AI agent。

**核心能力**：

- 理解视频脚本和创意描述
- 生成动画序列描述（关键帧、时间线）
- 与动画渲染工具集成

**典型架构**：

```
┌─────────────────────────────────────────────────────┐
│                  Motion Graphic Agent               │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │
│  │ User Intent │→ │ Motion Plan │→ │ Asset Gen   │ │
│  │  Parser     │  │  Generator  │  │  (Images,   │ │
│  │             │  │             │  │  Audio)     │ │
│  └─────────────┘  └─────────────┘  └────────────┘ │
│         ↓               ↓               ↓         │
│  ┌─────────────────────────────────────────────┐   │
│  │         Timeline Coordinator                │   │
│  │    (关键帧同步、时间线编排、过渡效果)         │   │
│  └─────────────────────────────────────────────┘   │
│                        ↓                            │
│  ┌─────────────────────────────────────────────┐   │
│  │         Render Engine Interface             │   │
│  │    (输出到 AE、Blender、Canvas 等)           │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

### 1.3 Skill 太多导致上下文窗口占用过大，如何处理？

**问题**：将所有 skill 直接塞进 system prompt，当 skill 数量多时上下文爆炸。

**解决方案**：

1. **Skill 分层加载**

   - 只加载与当前任务相关的 skill
   - 用 meta-skill 做路由，先判断需要哪类 skill
2. **Skill 索引与检索**

   - 建立 skill description 向量索引
   - 只在需要时动态注入相关 skill
3. **Skill 模板化**

   - skill 只保留 signature（名称、描述、参数）
   - 详细内容通过 function call 按需获取

```python
# 分层加载示例
class SkillManager:
    def __init__(self):
        self.meta_skills = [...]  # 元 skill 始终加载
        self.detailed_skills = {} # 详细 skill 按需加载

    def get_relevant_skills(self, task: str) -> List[Skill]:
        # 向量检索获取相关 skill
        relevant = self.vector_search(task, top_k=3)
        return relevant
```

4. **动态 Skill Composition**
   - 根据上下文动态组合 skill 子集

---

### 1.4 两个 skill 描述相似但内容不同，导致选错 skill，怎么解决？

**问题本质**：skill 路由的语义匹配不精确。

**解决方案**：

1. **增加区分性描述**

   - 每个 skill 的 description 要有明确的边界说明
   - 避免模糊的通用描述
2. **建立 Skill 互斥矩阵**

   - 标注相似 skill 之间的核心差异点
   - 路由时增加二次确认
3. **利用参数签名区分**

   - 如果输入参数类型/结构不同，可以作为区分依据
4. **反馈纠错机制**

   - 记录每次 skill 调用结果
   - 错误调用时修正路由权重

```python
# 区分性描述示例
skill_1 = Skill(
    name="image_generation",
    description="Generate images from text prompts. Input: {prompt: str}. Output: image_url."
)

skill_2 = Skill(
    name="image_edit",
    description="Edit existing images using masks or inpainting. Input: {image_url: str, mask: str, prompt:str}."
)
```

---

### 1.5 Agent 设计最重要的部分

**核心要素**：

1. **规划能力（Planning）**

   - 任务分解、步骤规划
   - 自我反思与纠错
2. **工具调用（Tool Use）**

   - 准确的 function calling
   - 工具结果的解析与利用
3. **记忆系统（Memory）**

   - 短期记忆：当前会话上下文
   - 长期记忆：跨会话知识积累
   - 层次化记忆管理
4. **安全边界（Safety）**

   - 权限控制
   - 操作审计
5. **上下文窗口管理（Context Engineering）**

   - 信息压缩与摘要
   - 关键信息保留

### 1.5.1 记忆系统深度问题

#### Q1: 短期记忆的存储结构

**本质**：短期记忆是会话级别的 KV 存储，用于保存当前会话的上下文。

```python
# 短期记忆的存储结构
class ShortTermMemory:
    """会话级短期记忆"""

    def __init__(self, session_id: str):
        self.session_id = session_id
        # 结构：List[Message] + 滑动窗口
        self.messages: List[Message] = []
        self.metadata: Dict[str, Any] = {
            "created_at": datetime.now(),
            "last_access": datetime.now(),
            "token_count": 0,
        }

    def add(self, role: str, content: str, meta: dict = None):
        """添加记忆单元"""
        msg = Message(
            role=role,
            content=content,
            timestamp=time.time(),
            meta=meta or {}
        )
        self.messages.append(msg)
        self.metadata["token_count"] += self.estimate_tokens(content)

    def get_recent(self, k: int = 10) -> List[Message]:
        """获取最近 k 条消息"""
        return self.messages[-k:]

    def search(self, query: str, top_k: int = 5) -> List[Message]:
        """基于语义检索相关记忆"""
        query_emb = self.embed(query)
        scores = []
        for msg in self.messages:
            msg_emb = self.embed(msg.content)
            score = cosine_sim(query_emb, msg_emb)
            scores.append((msg, score))
        return [msg for msg, _ in sorted(scores, key=lambda x: x[1], reverse=True)[:top_k]]
```

**存储特点**：
- 内存存储，会话结束时可选择持久化或丢弃
- 按时间顺序排列（List 结构）
- 支持按位置访问（recent）和语义检索（search）

---

#### Q2: 生成摘要是怎么存储的？

**摘要的存储策略**：摘要作为 "压缩后的记忆"，通常存储在独立表中，或作为消息的元字段。

```python
class MemoryStore:
    """统一记忆存储"""

    def __init__(self):
        self.short_term: Dict[str, List[Message]] = {}      # session_id -> messages
        self.summaries: Dict[str, List[Summary]] = {}        # session_id -> summaries
        self.long_term: VectorStore = VectorStore()          # 长期记忆向量库

    def add_message(self, session_id: str, msg: Message):
        """添加消息，自动触发摘要"""
        self.short_term.setdefault(session_id, []).append(msg)

        # 触发摘要条件：消息数量或 token 数超阈值
        if self._should_summarize(session_id):
            summary_text = await self._generate_summary(session_id)
            self.summaries.setdefault(session_id, []).append(
                Summary(
                    text=summary_text,
                    source_range=(0, len(self.short_term[session_id])),
                    created_at=datetime.now()
                )
            )
            # 摘要完成后清空原始消息（保留摘要）
            self.short_term[session_id] = []

    def get_context_for_prompt(self, session_id: str) -> str:
        """为 prompt 组装上下文"""
        parts = []

        # 1. 短期记忆中的摘要
        for summary in self.summaries.get(session_id, []):
            parts.append(f"[摘要 {summary.created_at}]: {summary.text}")

        # 2. 短期记忆中的原始消息（最近的部分）
        recent = self.short_term.get(session_id, [])[-10:]
        for msg in recent:
            parts.append(f"[{msg.role}]: {msg.content}")

        return "\n".join(parts)
```

**存储位置**：
- **摘要消息表**：独立存储每个会话的摘要链
- **消息元字段**：原始消息可内联存储摘要版本
- **向量数据库**：摘要文本向量化后存入长期记忆

---

#### Q3: 短期记忆是会话级的，跨对话的记忆怎么处理？

**跨会话记忆 = 长期记忆**，需要显式设计和存储。

```python
class LongTermMemory:
    """跨会话长期记忆"""

    def __init__(self):
        self.vector_store: VectorStore = VectorStore()  # Chroma / Milvus / FAISS
        self.entity_store: GraphStore = GraphStore()    # 知识图谱
        self.policy_store: Dict = {}                     # 用户偏好、政策规则

    def store_from_session(self, session_id: str, summary: str, entities: List[Entity]):
        """从会话中提取有价值的信息存入长期记忆"""
        # 1. 摘要向量化存入向量库
        self.vector_store.add(
            text=summary,
            metadata={"session_id": session_id, "type": "session_summary"}
        )

        # 2. 实体存入知识图谱
        for entity in entities:
            self.entity_store.upsert(entity)

        # 3. 用户偏好单独存储
        if entities.preferences:
            self.policy_store[session_id] = entities.preferences

    def retrieve(self, query: str, session_id: str = None) -> List[MemoryEntry]:
        """召回与当前任务相关的长期记忆"""
        # 1. 语义检索
        semantic_results = self.vector_store.search(query, top_k=10)

        # 2. 如果有 session_id，优先召回该用户的历史
        if session_id:
            user_specific = self.vector_store.search(
                query,
                filter={"session_id": session_id},
                top_k=5
            )
            semantic_results = user_specific + semantic_results

        # 3. 知识图谱召回（实体关联）
        graph_results = self.entity_store.query(query)

        # 4. 融合排序
        return self._rerank(semantic_results, graph_results, query)
```

**跨会话记忆的内容类型**：

| 类型 | 存储方式 | 召回方式 |
|------|---------|---------|
| 项目背景 | 向量库 | 语义检索 |
| 用户偏好 | KV 存储 | 精确匹配 |
| 实体关系 | 知识图谱 | 关系遍历 |
| 常用模式 | 结构化存储 | 规则匹配 |

---

#### Q4: 摘要压缩时，怎么限制摘要长度？

**两种策略：规则限制 vs 自由生成**

```python
class SummaryCompressor:
    """摘要压缩策略"""

    # 策略1：固定比例压缩
    def compress_by_ratio(self, text: str, ratio: float = 0.3) -> str:
        """压缩到原来的 ratio 比例"""
        target_tokens = int(self.count_tokens(text) * ratio)
        prompt = f"""请将以下内容摘要，目标 token 数约 {target_tokens}：

{text}

摘要："""
        return self.llm.generate(prompt)

    # 策略2：固定长度限制
    def compress_by_limit(self, text: str, max_tokens: int = 500) -> str:
        """压缩到不超过 max_tokens"""
        prompt = f"""请用简洁的语言摘要以下内容，不超过 {max_tokens} tokens：

{text}

摘要："""
        return self.llm.generate(prompt)

    # 策略3：语义单元保留
    def compress_by_units(self, text: str, max_units: int = 5) -> str:
        """按语义单元保留，保留最重要的 N 个单元"""
        prompt = f"""分析以下内容的语义结构，提取最重要的 {max_units} 个要点：

{text}

要点："""
        return self.llm.generate(prompt)

    # 推荐：混合策略
    def smart_compress(self, text: str) -> str:
        """智能压缩：根据内容类型选择策略"""
        tokens = self.count_tokens(text)

        if tokens < 500:
            return text  # 太短不压缩
        elif tokens < 2000:
            return self.compress_by_ratio(text, ratio=0.5)
        elif tokens < 5000:
            return self.compress_by_limit(text, max_tokens=500)
        else:
            # 复杂内容：先分段摘要，再总体摘要
            chunks = self.split_by_semantics(text)
            partial_summaries = [self.compress_by_limit(c, 300) for c in chunks]
            return self.compress_by_limit("\n".join(partial_summaries), 500)
```

**实践经验**：
- 简单对话：压缩到 30-50%
- 技术讨论：保留更多细节，压缩到 50%
- 复杂任务：分段摘要 + 总体摘要两层

---

#### Q5: 长期记忆的触发时机？如果只是主动录入 + 召回，和 RAG 有什么区别？

**长期记忆的触发时机**：

```python
class MemoryTrigger:
    """长期记忆的触发时机"""

    TRIGGERS = {
        # 1. 会话结束时：提取有价值信息
        "session_end": lambda ctx: ctx.session_has_value,

        # 2. 关键词触发：检测到重要概念
        "keyword": lambda ctx: ctx.contains_keywords([
            "项目", "需求", "决策", "方案", "用户", "问题"
        ]),

        # 3. 重复触发：某个信息被多次提到
        "repeated": lambda ctx: ctx.frequency >= 3,

        # 4. 间隔触发：每隔 N 轮对话强制摘要
        "interval": lambda ctx: ctx.turn_count % 10 == 0,

        # 5. 主动召回：当前任务需要时
        "on_demand": lambda ctx: ctx.task_requires_memory,
    }

    def should_store_long_term(self, session_context: SessionContext) -> bool:
        """判断是否应该存入长期记忆"""
        # 组合触发条件
        return any(trigger(session_context) for trigger in self.TRIGGERS.values())
```

**和 RAG 的核心区别**：

| 方面 | 长期记忆 (Agent) | RAG |
|------|-----------------|-----|
| **数据来源** | Agent 与用户的交互中主动提取 | 外部文档库 |
| **更新方式** | 增量学习：会话结束自动提取 | 批量索引：文档入库时 |
| **内容类型** | 经验、决策、偏好、模式 | 事实、知识、文档 |
| **召回粒度** | 碎片化：每个记忆单元很小 | 文档级：通常按 Chunk |
| **一致性** | 可能冲突（需要合并策略） | 相对一致（文档即事实） |
| **应用场景** | 个性化、连续性任务 | 知识问答、文档理解 |

**本质区别**：
- **RAG** = 检索外部知识，Agent "查询"
- **长期记忆** = Agent 自己学习到的 "经验"，Agent "记住"

```python
# 两者结合使用
class AgentMemorySystem:
    def __init__(self):
        self.rag = RAGSystem()      # 外部知识
        self.ltm = LongTermMemory() # Agent 经验

    async def retrieve(self, query: str, context: dict) -> str:
        # 1. RAG 召回外部知识（文档、API 文档、代码库）
        rag_results = await self.rag.search(query)

        # 2. 长期记忆召回 Agent 自己的经验
        ltm_results = await self.ltm.retrieve(query, context.get("session_id"))

        # 3. 融合：经验指导 RAG 检索，RAG 补充事实
        combined = self.fuse(ltm_results, rag_results)

        return combined
```

---

#### Q6: 两种检索方式是怎么融合的？

**检索融合策略：轻量级融合 vs 重排序融合**

```python
class RetrievalFusion:
    """多检索方式融合"""

    def __init__(self):
        self.dense_retriever = DenseRetriever()   # 向量检索
        self.sparse_retriever = SparseRetriever() # BM25/关键词检索
        self.graph_retriever = GraphRetriever()   # 知识图谱

    # 策略1：倒数融合（RRF）- 简单高效
    def reciprocal_rank_fusion(self, results_list: List[List[SearchResult]]) -> List[SearchResult]:
        """RRF 融合，多个检索结果取倒数排名加权和"""
        fused_scores: Dict[str, float] = {}

        for results in results_list:
            for rank, result in enumerate(results):
                doc_id = result.doc_id
                # RRF 公式：1 / (k + rank)，k 通常取 60
                fused_scores[doc_id] = fused_scores.get(doc_id, 0) + 1 / (60 + rank)

        # 按分数排序
        sorted_results = sorted(fused_scores.items(), key=lambda x: x[1], reverse=True)
        return [self.get_doc_by_id(doc_id) for doc_id, _ in sorted_results]

    # 策略2：学习型融合（需要训练数据）
    def learned_fusion(self, results_list: List[List[SearchResult]], query: str) -> List[SearchResult]:
        """训练一个小型模型来学习如何融合"""
        features = []
        for result_set in results_list:
            for result in result_set:
                features.append({
                    "dense_score": result.dense_score,
                    "sparse_score": result.sparse_score,
                    "graph_score": result.graph_score,
                    "recency": result.recency_score,
                    "query_match": result.query_match_score,
                })

        # 融合模型预测分数
        scores = self.fusion_model.predict(features)
        return self._sort_by_scores(results_list, scores)

    # 策略3：两阶段重排
    def two_stage_rerank(self, query: str, initial_results: List[SearchResult]) -> List[SearchResult]:
        """第一阶段多路召回，第二阶段 Cross-Encoder 重排"""
        # Stage 1: 多路召回
        dense_results = self.dense_retriever.search(query, top_k=50)
        sparse_results = self.sparse_retriever.search(query, top_k=50)
        initial = self.reciprocal_rank_fusion([dense_results, sparse_results])

        # Stage 2: Cross-Encoder 重排
        pairs = [(query, doc.text) for doc in initial]
        rerank_scores = self.cross_encoder.predict(pairs)

        return [doc for doc, _ in sorted(
            zip(initial, rerank_scores),
            key=lambda x: x[1],
            reverse=True
        )]
```

**融合的关键点**：

```python
# 实际应用中的融合配置
class FusionConfig:
    # 1. 权重配置：不同检索来源的权重
    WEIGHTS = {
        "dense": 0.4,      # 向量检索
        "sparse": 0.3,     # BM25 检索
        "graph": 0.2,      # 知识图谱
        "recency": 0.1,    # 时效性
    }

    # 2. 召回量配置
    RECALL_K = 100        # 初召回 100 条
    RERANK_K = 20         # 重排后取 20 条

    # 3. 去重规则
    DEDUP_SIMILARITY = 0.95  # 相似度 > 95% 认为重复
```

---

### 1.6 上下文工程要注意哪些点？

1. **信息分层**

   - 系统级：全局规则、长期目标
   - 任务级：当前任务描述、约束
   - 会话级：历史交互摘要
   - 实时级：最新工具调用结果
2. **压缩策略**

   - 关键信息提取
   - 冗余消除
   - 层次化摘要
3. **召回率 vs 精确度**

   - 保留与当前任务高度相关的信息
   - 平衡上下文长度与信息完整性
4. **时序信息维护**

   - 对话/操作顺序的重要性
   - 避免上下文位置偏差
5. **多模态上下文**

   - 图像、代码、表格等不同模态的处理

---

### 1.7 了解主流的 Agent 设计吗？

| Agent 框架                 | 设计特点                     | 代表项目         |
| -------------------------- | ---------------------------- | ---------------- |
| **ReAct**            | 交替执行 Reasoning 和 Action | LangChain Agents |
| **Plan-and-Execute** | 先规划后执行，关注长期目标   |                  |
| **Reflexion**        | 自我反思与纠错机制           |                  |
| **AutoGPT**          | 自主目标分解与执行           |                  |
| **Claude Code**      | 分层上下文、安全优先         | Anthropic        |
| **Gemini CLI**       | 原生集成工具链               | Google           |
| **OpenClaude**       | 开源架构参考                 |                  |

**核心架构模式**：

- 单 agent + 工具生态
- 多 agent 协作
- 层次化 agent（主 agent + 子 agent）

---

### 1.8 Gemini CLI 设计最重要的部分

1. **工具生态集成**

   - 与 Google 服务深度集成
   - 统一的工具调用协议
2. **多模态原生支持**

   - 视频、音频、代码的统一处理
   - 跨模态推理能力
3. **安全与权限管理**

   - 操作权限的精细控制
   - 用户意图的安全校验
4. **实时反馈机制**

   - 流式输出
   - 进度可视化

---

### 1.9 Gemini CLI 可以优化的点

1. **工具调用效率**

   - 批量工具调用优化
   - 工具调用缓存
2. **上下文管理**

   - 更智能的上下文压缩
   - 长期记忆的持久化
3. **多 agent 协作**

   - 支持子 agent 分解任务
   - agent 间通信协议
4. **错误恢复**

   - 更强的自我纠错
   - 失败后的重试策略

---

### 1.10 Multi Agent 架构主流的实现方案

1. **层次化（Hierarchical）**

   ```
   Main Agent → Sub Agents (分工协作)
   ```

   - 主 agent 负责任务分解和协调
   - 子 agent 负责具体执行
2. **平等协作（Peer-to-Peer）**

   - 所有 agent 地位平等
   - 通过消息传递协作
3. **图结构（Graph-based）**

   - Agent 作为节点
   - 边定义通信关系和依赖

**主流框架**：

- **AutoGen** (Microsoft)：对话式多 agent
- **CrewAI**：角色扮演式多 agent
- **LangGraph**：状态机式多 agent
- **MetaGPT**：SOP 驱动的多 agent

---

### 1.11 Multi Agent 的交流靠什么？

1. **消息传递**

   - 共享消息队列
   - 发布-订阅模式
2. **共享状态**

   - 共享内存/知识库
   - 黑板系统（Blackboard Pattern）
3. **协议定义**

   - 统一通信协议（JSON/Protobuf）
   - 预定义消息类型

```python
# 消息传递示例
class AgentMessage:
    sender: str
    receiver: str
    content: dict  # 结构化内容
    type: MessageType  # REQUEST, RESPONSE, BROADCAST

# 通信方式
# 1. 直接消息：agent_a.send_to(agent_b, message)
# 2. 广播：agent_a.broadcast(message)
# 3. 中央协调：所有消息经过 coordinator
```

---

### 1.12 介绍一下 Harness 工程

**Harness 是什么**：
Harness（测试/运行 harness）是支撑 Agent 运行的工程框架，负责管理 Agent 的生命周期、工具调用、上下文分发、安全控制等核心功能。可以理解为 Agent 的 "操作系统"。

**核心架构组件**：

```
┌─────────────────────────────────────────────────────────────────┐
│                         Harness Framework                        │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   Session   │  │   Context   │  │    Tool     │            │
│  │  Manager    │  │  Manager    │  │  Registry   │            │
│  │             │  │             │  │             │            │
│  │ - 会话管理   │  │ - 上下文分页│  │ - 工具注册  │            │
│  │ - 状态持久化 │  │ - 窗口压缩  │  │ - 版本控制  │            │
│  │ - 多会话    │  │ - 层级管理  │  │ - 调用路由  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   Safety    │  │   Output    │  │    Hooks    │            │
│  │   Guard     │  │  Processor  │  │   System    │            │
│  │             │  │             │  │             │            │
│  │ - 操作审计   │  │ - 流式处理  │  │ - 生命周期 │            │
│  │ - 权限控制   │  │ - 结果校验  │  │ - 事件触发 │            │
│  │ - 风险拦截   │  │ - 格式化    │  │ - 插件扩展  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                      LLM Provider Interface                      │
│            (OpenAI / Anthropic / Gemini / Local)                │
└─────────────────────────────────────────────────────────────────┘
```

**核心模块详解**：

#### 1. Session Manager（会话管理）

```python
class SessionManager:
    """管理 Agent 的会话生命周期"""

    def __init__(self):
        self.sessions: Dict[str, Session] = {}
        self.active_session: Optional[str] = None

    def create_session(self, session_id: str, config: SessionConfig) -> Session:
        """创建新会话"""
        session = Session(id=session_id, config=config)
        self.sessions[session_id] = session
        return session

    def switch_session(self, session_id: str) -> bool:
        """切换活跃会话"""
        if session_id in self.sessions:
            self.active_session = session_id
            return True
        return False

    def persist_session(self, session_id: str) -> None:
        """持久化会话状态"""
        # 保存到磁盘或数据库
        save_to_disk(self.sessions[session_id])

    def restore_session(self, session_id: str) -> Optional[Session]:
        """恢复会话"""
        return load_from_disk(session_id)
```

#### 2. Context Manager（上下文管理）

```python
class ContextManager:
    """分层上下文管理 + 窗口压缩"""

    # 层级定义
    LEVELS = {
        4: "system",      # 系统级：全局规则
        3: "project",     # 项目级：代码结构
        2: "task",        # 任务级：当前任务
        1: "conversation", # 会话级：最近对话
        0: "realtime",    # 实时级：最新输入
    }

    def __init__(self, max_window_tokens: int = 200000):
        self.max_window = max_window_tokens
        self.layers = {i: [] for i in range(5)}  # L0-L4

    def add_context(self, level: int, content: str, metadata: dict = None):
        """添加上下文到指定层级"""
        self.layers[level].append({
            "content": content,
            "metadata": metadata or {}
        })

    def compress(self) -> str:
        """压缩上下文，返回扁平化的 prompt"""
        # 按优先级排序
        # 超出窗口时从低优先级层级开始丢弃
        compressed = []
        total_tokens = 0

        for level in range(4, -1, -1):
            for item in self.layers[level]:
                tokens = self.estimate_tokens(item["content"])
                if total_tokens + tokens <= self.max_window:
                    compressed.append(item["content"])
                    total_tokens += tokens
                else:
                    # 该层级超限，尝试摘要
                    summary = self.summarize(item["content"])
                    compressed.append(f"[摘要] {summary}")

        return "\n\n".join(reversed(compressed))

    def summarize(self, content: str) -> str:
        """生成摘要"""
        # 调用小模型或规则进行摘要
        pass
```

#### 3. Tool Registry（工具注册）

```python
class ToolRegistry:
    """统一工具管理"""

    def __init__(self):
        self.tools: Dict[str, Tool] = {}
        self.tool_schemas: Dict[str, dict] = {}  # MCP 风格 schema

    def register(self, tool: Tool, metadata: ToolMetadata):
        """注册工具"""
        self.tools[tool.name] = tool
        self.tool_schemas[tool.name] = {
            "name": tool.name,
            "description": metadata.description,
            "parameters": tool.parameters_schema,
            "version": metadata.version,
            "deprecation": metadata.deprecated,
        }

    def dispatch(self, tool_name: str, params: dict) -> ToolResult:
        """分发工具调用"""
        if tool_name not in self.tools:
            raise ToolNotFoundError(f"Tool {tool_name} not found")

        tool = self.tools[tool_name]

        # 安全检查
        self._check_permissions(tool_name, params)

        # 调用
        return tool.execute(params)

    def get_relevant_tools(self, task: str, top_k: int = 5) -> List[Tool]:
        """基于任务检索相关工具"""
        # 向量相似度检索
        task_embedding = self.embed(task)
        scores = [
            (name, cosine_sim(task_embedding, self.embed(desc)))
            for name, desc in self.tool_schemas.items()
        ]
        return [self.tools[name] for name, _ in sorted(scores, key=lambda x: x[1], reverse=True)[:top_k]]
```

#### 4. Safety Guard（安全控制）

```python
class SafetyGuard:
    """安全审计与权限控制"""

    def __init__(self):
        self.dangerous_patterns = [
            r"rm\s+-rf\s+/",           # 删除根目录
            r"drop\s+database",         # 删除数据库
            r"rm\s+-rf\s+\*",          # 全局删除
            # ... 更多危险命令模式
        ]
        self.permission_matrix: Dict[str, Set[str]] = {}  # user_id -> allowed_tools

    def pre_check(self, action: Action) -> CheckResult:
        """执行前检查"""
        # 1. 权限检查
        if action.tool_name not in self.permission_matrix.get(action.user_id, set()):
            return CheckResult(allowed=False, reason="Permission denied")

        # 2. 危险模式检测
        for pattern in self.dangerous_patterns:
            if re.search(pattern, action.params_str):
                return CheckResult(allowed=False, reason=f"Dangerous pattern: {pattern}")

        # 3. 操作审计日志
        self.audit_log.log(action)

        return CheckResult(allowed=True)

    def post_check(self, result: ToolResult) -> ToolResult:
        """执行后检查"""
        # 检查返回内容是否包含敏感信息
        if self.contains_secrets(result.output):
            return ToolResult(output="[已脱敏]", success=True)
        return result
```

#### 5. Hooks System（生命周期钩子）

```python
class HooksSystem:
    """事件驱动的插件系统"""

    HOOK_POINTS = [
        "before_agent_run",    # Agent 运行前
        "after_agent_run",     # Agent 运行后
        "before_tool_call",    # 工具调用前
        "after_tool_call",     # 工具调用后
        "on_error",            # 错误发生
        "on_session_start",    # 会话开始
        "on_session_end",      # 会话结束
    ]

    def __init__(self):
        self.hooks: Dict[str, List[Callable]] = {point: [] for point in self.HOOK_POINTS}

    def register(self, point: str, callback: Callable):
        """注册钩子"""
        if point not in self.HOOK_POINTS:
            raise ValueError(f"Unknown hook point: {point}")
        self.hooks[point].append(callback)

    def trigger(self, point: str, context: dict):
        """触发钩子"""
        for callback in self.hooks[point]:
            callback(context)
```

**Harness 工程的核心挑战**：

1. **上下文窗口管理**

   - 如何在有限窗口内塞入最多有效信息
   - 智能摘要 vs 截断的权衡
2. **工具调用可靠性**

   - 工具返回格式不统一
   - 超时、重试、熔断策略
3. **安全与自由的平衡**

   - 限制太少：危险操作风险
   - 限制太多：影响 Agent 能力
4. **多会话状态隔离**

   - 会话间不泄露信息
   - 状态持久化与恢复
5. **流式输出的实时性**

   - Token 级别的流式处理
   - 进度反馈与打断机制

**工程实践建议**：

1. **模块化设计**：各组件松耦合，便于独立测试和迭代
2. **接口抽象**：Tool、Session、Context 等都有标准接口
3. **可观测性**：完善的日志、监控、审计
4. **优雅降级**：部分组件失败不影响整体
5. **配置驱动**：通过配置而非代码控制行为

---

## 二、项目深度问题（围绕 Claude Code 类项目）

### 2.1 有了 Claude Code，为什么还要做类似项目？

1. **定制化需求**：特定业务场景需要定制化 agent
2. **技术探索**：学习 agent 架构设计
3. **差异化功能**：在某些垂直领域做得更深
4. **开源贡献**：推动 agent 技术发展
5. **企业级需求**：私有化部署、安全审计

---

### 2.2 核心差异和优劣势

| 方面           | 自研项目                             | Claude Code                      |
| -------------- | ------------------------------------ | -------------------------------- |
| **优势** | 定制化能力强、可私有化、垂直场景优化 | 品牌背书、工程成熟度高、生态完善 |
| **劣势** | 工程量大、持续维护成本高             | 定制受限、依赖闭源               |

---

### 2.3 分层上下文管理，每层管什么？

```
┌─────────────────────────────────────────┐
│          L4: 系统级上下文                │
│   (全局规则、安全策略、长期目标)           │
├─────────────────────────────────────────┤
│          L3: 项目级上下文                 │
│   (项目结构、技术栈、代码规范)             │
├─────────────────────────────────────────┤
│          L2: 任务级上下文                 │
│   (当前任务描述、约束、历史决策)           │
├─────────────────────────────────────────┤
│          L1: 会话级上下文                 │
│   (最近交互、工具调用结果、中间状态)        │
├─────────────────────────────────────────┤
│          L0: 实时上下文                   │
│   (用户最新输入、光标位置、文件状态)        │
└─────────────────────────────────────────┘
```

---

### 2.4 摘要生成器用什么模型？质量如何保证？

**常用方案**：

1. **小模型专门微调**：Llama 7B + 垂直微调
2. **大模型 API**：GPT-4、Claude
3. **抽取式摘要**：BERT-based 模型

**质量保证**：

- 人工评估 + 自动指标（ROUGE、BERT-score）
- A/B 测试对比
- 反馈循环优化

---

### 2.5 Subagent 的探索，启动多个 agent 的作用

**作用**：

1. **并行执行**：多个子任务同时处理
2. **专业分工**：子 agent 各有所长
3. **故障隔离**：一个子 agent 失败不影响其他
4. **复杂任务分解**：复杂任务拆解为多个子任务

---

### 2.6 主 agent 和子 agent 通信怎么实现？

1. **共享内存/消息队列**

   ```python
   shared_context = {
       "task": task_description,
       "subtasks": [],
       "results": {},
       "status": "running"
   }
   ```
2. **层级调用**

   - 主 agent 通过 spawn 创建子 agent
   - 子 agent 执行后返回结果给主 agent
3. **状态同步**

   - 定期同步中间结果
   - 主 agent 监控子 agent 状态

---

### 2.7 Agent 陷入死循环的解决方案

1. **深度限制**

   - 设置最大循环次数
   - 超出后强制终止或寻求用户确认
2. **去重机制**

   - 记录已执行的操作
   - 相同状态/操作时跳过
3. **进度检测**

   - 监控任务是否有实际进展
   - 无进展时触发干预
4. **外部打断**

   - 用户可以随时打断
   - 提供放弃/重启选项

```python
class LoopDetector:
    def __init__(self, max_iterations=20):
        self.seen_states = {}
        self.iteration = 0

    def check(self, state: str) -> bool:
        """检测是否陷入循环"""
        if state in self.seen_states:
            self.seen_states[state] += 1
        else:
            self.seen_states[state] = 1

        self.iteration += 1

        # 超过阈值或重复状态
        if self.iteration > self.max_iterations:
            return True
        if self.seen_states[state] > 3:
            return True
        return False
```

---

## 三、强化学习与训练（GRPO/PPO/LoRA）

### 3.1 GRPO 和 PPO 的区别？

| 方面               | PPO                              | GRPO                   |
| ------------------ | -------------------------------- | ---------------------- |
| **采样方式** | 同一策略采样，通过重要性采样校正 | 对同一提示生成多个响应 |
| **样本效率** | 相对较低（需要新旧策略对比）     | 相对较高（组内对比）   |
| **计算开销** | 较大（需要保存旧策略）           | 较小                   |
| **稳定性**   | 更稳定                           | 取决于组内多样性       |
| **适用场景** | 通用场景                         | 生成任务、代码等       |

**GRPO（Group Relative Policy Optimization）**：

- 对同一 prompt 生成多个 response
- 通过组内相对排序计算优势
- 减少对旧策略的依赖

---

### 3.2 KL 散度如何加入？太大或太小的问题？

**KL 散度在 RLHF 中的作用**：限制策略更新幅度，防止灾难性遗忘。

**加入方式**：

```python
# KL 散度惩罚项
loss = -reward + beta * KL(pi_old || pi_new)

# 或者作为 PPO 的 KL 约束
clip_ratio = torch.clamp(ratio, 1-eps, 1+eps)
loss = -min(ratio * advantage, clip_ratio * advantage) + beta * KL
```

**KL 散度太大的影响**：

- 策略更新过于激进
- 可能导致训练不稳定
- 破坏已有能力

**KL 散度太小的后果**：

- 策略更新过于保守
- 训练收敛太慢
- 无法有效学习新行为

---

### 3.3 QLoRA 的 rank 怎么设置？

**常见配置**：

- **小 rank（4-16）**：参数量小，适合小模型或资源受限场景
- **中等 rank（32-64）**：平衡性能和效率
- **大 rank（128-256）**：性能更好，但显存占用更大

**选择依据**：

1. 模型大小：大模型可用更大 rank
2. 任务复杂度：复杂任务需要更高 rank
3. 显存限制：根据硬件调整
4. 实验调优：A/B 测试选择最优

**常用 rank**：16（LLaMA 7B）、32（LLaMA 13B）、64（LLaMA 30B+）

---

### 3.4 训练参数怎么选？有没有调参测试？

**关键参数**：

1. **学习率**：1e-4 到 1e-5 之间，常用 2e-4
2. **Batch size**：根据显存，通常 8-64
3. **Epochs**：3-10，根据数据量调整
4. **LoRA rank**：16-64
5. **LoRA alpha**：通常是 rank 的两倍
6. **Dropout**：0.05-0.1

**调参策略**：

- Grid search 或随机搜索
- 先在小数据上快速验证
- 监控 loss 曲线和验证集指标

---

### 3.5 LoRA 和 QLoRA 的区别

| 方面                | LoRA      | QLoRA                |
| ------------------- | --------- | -------------------- |
| **全量参数**  | 不量化    | 4-bit 量化           |
| **LoRA 权重** | FP16/BF16 | FP16/BF16（不量化）  |
| **显存需求**  | 较高      | 大幅降低             |
| **训练速度**  | 快        | 较慢（需要反量化）   |
| **精度**      | 原始精度  | 略有下降，通常可接受 |

**QLoRA 核心**：

- NF4（4-bit NormalFloat）量化
- 双重量化（量化 LoRA 输入）
- 分页优化器（处理显存峰值）

Qlora 核心目标：在不牺牲太多性能的前提下 极大降低模型加载的显存

核心思想是：“用 4-bit 精度加载基础模型 W0，但用 16-bit 精度训练 LoRA 适配器。”

但是是 NF4 量化，NF4 是一种专门为正态分布数据设计的 4-bit 数据类型。 量化的时候取的代表值不是均匀取的，

越远离 0 间隔越大。因为大量的权重都集中在 0 附近 而且近似正态分布 所以在量化的时候 就把新的代表值尽量设置

在 0 附近 这样才能更好的表示出权重值的差异 对大量集中的数据做一个有效的区分

---

### 3.6 量化对训练效果的影响

**正面影响**：

- 大幅降低显存占用
- 可以训练更大的模型

**负面影响**：

- 精度略有下降
- 对极端值处理可能有问题

**经验**：

- 4-bit 量化通常精度损失 < 1%
- 重要任务建议使用 8-bit 或 16-bit
- 量化后微调可弥补部分精度损失

---

### 3.7 梯度检查点原理，对训练速度的影响

**原理**：

- 训练时不保存所有中间激活值
- 反向传播时重新计算需要的激活值
- 用计算换显存

**速度影响**：

- 通常减慢 20-30%
- 但允许使用更大 batch size
- 总体效率可能更高

```python
# PyTorch 梯度检查点
from torch.utils.checkpoint import checkpoint

class ModelWithGradientCheckpointing(nn.Module):
    def forward(self, x):
        # 分段使用 checkpoint
        x = checkpoint(self.layer1, x)
        x = checkpoint(self.layer2, x)
        return self.layer3(x)
```

---

## 四、随机问题

### 4.1 用过哪些 AI Agent 工具？

- **Claude Code**：代码能力强，架构清晰
- **Cursor**：IDE 集成好，智能补全强
- **GitHub Copilot**：生态完善，普及度高
- **ChatGPT with Plugins**：多模态能力强
- **Gemini CLI**：Google 生态集成
- **Devin**：独立 agent 能力

---

### 4.2 AI 工具最大的帮助场景

1. **代码生成与补全**：快速实现功能
2. **代码审查**：发现潜在 bug
3. **文档生成**：自动生成注释和文档
4. **调试辅助**：分析错误原因
5. **学习助手**：解释代码和概念
6. **技术调研**：快速了解新领域

---

### 4.3 遇到过 AI 应用无法解决的场景

1. **需要业务知识的任务**：不了解公司特定业务逻辑
2. **实时信息查询**：需要最新数据
3. **复杂系统调试**：涉及多系统交互
4. **需要主观判断**：产品设计决策
5. **物理实验操作**：无法操作真实环境

---

### 4.4 写的代码有多少是 AI 生成的

- 新手阶段：可能 60-70%
- 有经验后：30-50%（更有判断力）
- 核心架构/算法：仍然主要自己写
- AI 主要用于：模板代码、测试、文档、重复性工作

---

### 4.5 OpenClaw 有没有实际使用过？

**OpenClaw 架构优势**：

1. **开源透明**：架构可追溯
2. **模块化设计**：组件可替换
3. **多模型支持**：不绑定单一 provider
4. **工具生态**：丰富的工具集成

**使用经验**：有在实验中参考其架构设计

---

### 4.6 Claude Code/ OpenClaw 可改进的点

1. **更智能的上下文选择**：自动判断哪些文件需要加载
2. **更好的长任务处理**：长时间运行任务的进度反馈
3. **多 agent 协作**：更好地支持复杂任务分解
4. **个性化学习**：学习用户的编码习惯
5. **更强的调试能力**：更好地理解运行时状态

---

### 4.7 Claude Code 源码泄露，有没有了解？

**泄露内容分析**：

1. **分层上下文管理**：清晰的层级划分
2. **安全设计**：敏感操作的二次确认
3. **工具集成**：统一的工具调用协议
4. **状态机设计**：agent 行为的确定性

**值得借鉴**：

- 安全优先的设计理念
- 详细的日志和审计
- 用户可控的操作权限

---

### 4.8 做 Agent 最难的部分

1. **上下文管理**：如何在有限窗口内保持关键信息
2. **规划稳定性**：避免重复和死循环
3. **工具调用准确性**：正确选择和调用工具
4. **错误恢复**：从失败中恢复的能力
5. **安全与自由的平衡**：限制太多功能受限，限制太少危险

---

### 4.9 踩过最大的坑

1. **上下文爆炸**：没有限制导致 token 费用爆炸
2. **循环调用**：缺少去重机制导致死循环
3. **工具返回解析失败**：返回格式不符合预期
4. **状态丢失**：长时间任务中断后状态丢失
5. **Prompt 注入**：恶意用户尝试 prompt 注入

---

### 4.10 好 Prompt vs 差 Prompt

| 好 Prompt        | 差 Prompt      |
| ---------------- | -------------- |
| 明确具体，有示例 | 模糊笼统       |
| 指定输出格式     | 没有格式要求   |
| 包含约束条件     | 缺少限制       |
| 角色定义清晰     | 无角色设定     |
| 分步骤说明       | 一次性全部给出 |

**示例**：

```python
# 差 Prompt
"帮我写一个排序算法"

# 好 Prompt
"""
作为算法工程师，请实现一个通用的排序算法。

要求：
1. 语言：Python
2. 时间复杂度：O(n log n)
3. 输入：List[int]
4. 输出：List[int]（已排序）
5. 包含单元测试

请先解释思路，再实现代码。
"""
```

---

### 4.11 除了 Qwen3VL，还用过其他多模态模型吗？

- **GPT-4V**：能力强，但需要 API
- **Gemini Pro Vision**：Google 生态
- **LLaVA**：开源可本地部署
- ** CogVLM**：国产开源
- **InternVL**：商汤开源，性能不错

---

### 4.12 端侧部署模型了解吗？

1. **端侧模型**：

   - Phi-3-mini：微软，小而强
   - Mistral-Nemo：7B 级别
   - Qwen2-0.5B/1.5B/7B：阿里，多尺寸
   - SMOLLM：Meta，小型化
2. **端侧框架**：

   - **MLC-LLM**：支持多平台
   - **llama.cpp**：量化推理
   - **TensorRT-LLM**：高性能
   - **Core ML**：Apple 生态
3. **部署方案**：

   - 量化（INT4/INT8）
   - 知识蒸馏
   - 剪枝

---

## 五、工具对比与 MCP

### 5.1 Coding Agent 哪个最好用？

**按场景推荐**：

- **日常开发**：Claude Code > Cursor > Copilot
- **大型项目**：Claude Code（上下文管理强）
- **快速补全**：Cursor（实时性好）
- **全流程项目**：Claude Code（规划能力强）

**个人体验**：Claude Code 在复杂任务上表现最好

---

### 5.2 CC 和其他工具的差异

| 方面                 | Claude Code | Cursor   | Copilot  |
| -------------------- | ----------- | -------- | -------- |
| **上下文窗口** | 更大        | 有限     | 有限     |
| **工具调用**   | 原生        | 插件     | 插件     |
| **规划能力**   | 强          | 一般     | 一般     |
| **IDE 集成**   | CLI 为主    | 深度集成 | 深度集成 |
| **价格**       | 订阅制      | 免费/Pro | 订阅制   |

---

### 5.3 Claude Code 技术优化

1. **分层上下文**：只加载相关文件
2. **智能文件选择**：判断哪些文件需要读取
3. **安全优先**：敏感操作需要确认
4. **流式输出**：实时反馈进度
5. **工具设计**：统一、类型安全的工具协议

---

### 5.4 为什么业界从 MCP 转向 Skill？

**MCP 的问题**：

1. **学习成本高**：需要理解协议
2. **集成复杂**：每个 server 需要单独配置
3. **版本兼容**：不同版本可能不兼容
4. **调试困难**：协议层面的问题难排查

**Skill 的优势**：

1. **简单直观**：声明式描述
2. **统一格式**：更容易管理
3. **动态加载**：按需加载，不占上下文
4. **版本控制**：更容易追踪变更

---

### 5.5 MCP 和直接开放端口调用的区别

| 方面               | MCP          | 直接端口调用 |
| ------------------ | ------------ | ------------ |
| **标准化**   | 统一协议     | 各不相同     |
| **发现机制** | 支持服务发现 | 需要手动配置 |
| **安全性**   | 协议层安全   | 需要自己实现 |
| **工具描述** | 自描述       | 需要额外文档 |
| **适用场景** | 通用工具生态 | 特定系统集成 |

---

### 5.6 模型如何拥有 Function Call 能力

**本质**：

1. **训练阶段**：大量 function calling 示例数据
2. **微调阶段**：特定的 function call 格式训练
3. **推理阶段**：输出符合特定格式的调用指令

**常见格式**：

```json
{
  "name": "get_weather",
  "arguments": {
    "location": "北京",
    "unit": "celsius"
  }
}
```

---

### 5.7 Agent 上下文和 LLM 上下文的区别

| 方面             | Agent 上下文                   | LLM 上下文      |
| ---------------- | ------------------------------ | --------------- |
| **组成**   | 工具定义、历史消息、状态、知识 | 全部输入 tokens |
| **管理**   | Agent 负责管理                 | 模型自动处理    |
| **内容**   | 任务描述+工具+状态+知识        | 原始文本        |
| **压缩**   | 需要主动压缩                   | 位置编码限制    |
| **持久化** | 可跨会话                       | 通常单会话      |

---

### 5.8 上下文过多如何处理

1. **分层管理**：系统级/项目级/任务级分离
2. **选择性加载**：只加载当前任务相关的
3. **摘要压缩**：定期摘要历史信息
4. **向量检索**：用 embedding 检索相关内容
5. **窗口滑动**：丢弃最旧的信息

---

### 5.9 如何让 Agent 理解长 PRD 文档

**PRD 是什么**：

- **PRD（Product Requirements Document）**：产品需求文档
- 是产品经理编写的核心文档，描述产品的功能需求、用户故事、业务流程、非功能性需求（性能、安全、兼容性等）
- 通常包含：产品背景、目标用户、功能列表、流程图、接口需求、数据字典、验收标准等
- 在大公司中可能长达几十甚至上百页

**Agent 理解长 PRD 的方案**：

1. **分段处理**：将 PRD 拆分为多个部分
2. **关键信息提取**：自动提取核心需求和约束
3. **结构化转换**：将自然语言转为结构化需求
4. **验证理解**：让 Agent 复述确认
5. **关联知识**：补充相关背景知识

---

### 5.10 1M 上下文如何输入 2M 文档

**方案**：

1. **语义分块**：按语义段落切分
2. **滑动窗口**：重叠分块保留上下文
3. **层次化摘要**：先摘要再逐步展开
4. **RAG 检索**：只加载相关部分
5. **多轮输入**：分批次处理

---

### 5.11 模型知识和 RAG 知识在注意力上的区别

| 方面               | 模型知识       | RAG 知识         |
| ------------------ | -------------- | ---------------- |
| **存储位置** | 权重参数       | 外部向量数据库   |
| **访问方式** | 通过 attention | 检索后作为上下文 |
| **更新方式** | 需要训练       | 可实时更新       |
| **确定性**   | 概率性recall   | 精确召回         |
| **幻觉风险** | 存在           | 较低（但仍可能） |
| **计算方式** | 分布式权重     | 局部上下文       |

**注意力机制**：

- 模型知识：全局 attention 到所有 token
- RAG 知识：当前 token 只 attention 到检索到的 chunk

---

### 5.12 单智能体和多智能体优缺点

| 方面               | 单智能体 | 多智能体       |
| ------------------ | -------- | -------------- |
| **复杂性**   | 简单     | 复杂           |
| **扩展性**   | 差       | 好             |
| **并行性**   | 弱       | 强             |
| **一致性**   | 高       | 需要协调       |
| **容错性**   | 差       | 好（故障隔离） |
| **适用场景** | 简单任务 | 复杂任务       |
| **通信开销** | 无       | 有             |

---

### 5.13 HTTP 协议演进

1. **HTTP/0.9**：1991，只支持 GET，极简
2. **HTTP/1.0**：1996，引入 POST/HEAD，连接无法复用
3. **HTTP/1.1**：1999，keep-alive、缓存、断点续传
4. **HTTP/2**：2015，多路复用、头部压缩、Server Push
5. **HTTP/3**：2022，QUIC 协议（UDP）、0-RTT

**关键优化**：

- 连接复用 → 减少握手延迟
- 多路复用 → 并行请求
- 头部压缩 → 减少传输量
- QUIC → 解决队头阻塞

---

### 5.14 Redis vs SQL 优缺点和场景

| 方面             | Redis                | SQL            |
| ---------------- | -------------------- | -------------- |
| **类型**   | NoSQL，KV 为主       | 关系型         |
| **查询**   | 简单 K/V             | 复杂 JOIN      |
| **一致性** | 弱一致（可选强一致） | ACID 强一致    |
| **性能**   | 极高                 | 较低           |
| **容量**   | 内存限制             | 可存磁盘       |
| **场景**   | 缓存、会话、实时     | 业务数据、事务 |

**选择原则**：

- 高频读取、低频写入 → Redis
- 复杂查询、事务 → SQL
- 两者结合 → Cache Aside 模式

---

### 5.15 Redis 和 SQL 一致性策略

**Redis**：

- **最终一致**：异步复制
- **强一致**：WAIT 命令（少量数据）
- **事务**：MULTI/EXEC（乐观锁）

**SQL**：

- **强一致**：MVCC + 两阶段提交
- **最终一致**：读写分离 + 主从延迟

**Cache Aside 模式**：

```
读：cache → hit? return : db → update cache
写：db → delete cache (不是更新 cache)
```

---

### 5.16 项目读写策略

**典型策略**：

1. **读多写少**：Cache Aside + Redis
2. **写多读少**：写缓冲 + 批量落库
3. **一致性要求高**：同步写 DB + 消息队列异步更新 Cache
4. **最终一致即可**：异步更新 Cache

**实现要点**：

- 读写分离降低主库压力
- 延迟双删解决缓存一致性问题
- 雪花算法生成唯一 ID

---

## 六、RAG 与大数据

### 6.1 大数据 RAG 和上下文管理

**大数据 RAG 挑战**：

1. **召回量级**：百万级文档如何高效召回
2. **相关性排序**：Top-K 召回如何保证质量
3. **上下文窗口**：召回内容过多如何处理
4. **多跳推理**：需要跨多个文档推理

**解决方案**：

1. **层次化索引**：Document → Chunk → Entity
2. **混合检索**：Dense + Sparse 融合
3. **重排序**：两阶段召回 + 重排
4. **上下文压缩**：摘要 + 原文结合

---

### 6.2 如何确定召回提高了准确性

1. **离线评估**

   - Hit@K / MRR / NDCG
   - A/B 测试对比
2. **召回质量检测**

   - 相关性分数阈值
   - 多样性指标
3. **端到端评估**

   - RAG 回答质量评估
   - 用户反馈收集
4. **消融实验**

   - 有无 RAG 对比
   - 不同召回策略对比

---

### 6.3 减少幻觉的方法

1. **召回验证**：确保召回内容支持答案
2. **溯源标注**：答案必须标注来源
3. **置信度机制**：低置信度时明确告知
4. **拒答机制**：无法回答时拒绝生成
5. **多源交叉验证**：多个来源一致才输出
6. **后处理校验**：事实核查模块

---

### 6.4 多用户场景

**挑战**：

1. **权限控制**：不同用户访问不同数据
2. **个性化**：用户偏好和历史
3. **隔离性**：用户间数据不泄露

**解决方案**：

1. **租户隔离**：数据按用户/租户隔离
2. **向量空间隔离**：用户向量空间分离
3. **查询改写**：注入用户身份信息
4. **访问控制列表**：精细化权限管理

---

## 七、基础知识

### 7.1 FastAPI 是什么？

**FastAPI** 是一个现代化的 Python Web 框架，用于构建 APIs，于 2018 年发布。

**核心特点**：

1. **高性能**：基于 Starlette（异步框架）+ Pydantic（数据验证），性能接近 NodeJS/Go
2. **自动文档**：内置 Swagger UI 和 ReDoc，接口文档自动生成
3. **类型提示**：完全基于 Python 类型注解，无需额外 DSL
4. **异步支持**：原生 async/await 支持，高并发场景表现优异
5. **数据验证**：Pydantic 集成，自动请求/响应验证和序列化

**核心概念**：

```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

# 1. 定义数据模型
class User(BaseModel):
    id: int
    name: str
    email: str
    age: Optional[int] = None

# 2. 定义路由
@app.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    user = db.get_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.post("/users/", response_model=User, status_code=201)
async def create_user(user: User):
    new_user = db.create_user(user)
    return new_user

# 3. 依赖注入
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(db = Depends(get_db)):
    return db.query(Item).all()
```

**关键优势对比**：

| 特性 | FastAPI | Flask | Django |
|------|---------|-------|--------|
| **性能** | 极高（异步） | 中等 | 中等 |
| **类型安全** | 原生类型提示 | 手动 | Django ORM |
| **自动文档** | Swagger/ReDoc | 无 | DRF 可选 |
| **异步支持** | 原生 | 需扩展 | 需 Channels |
| **适用场景** | 微服务、API | 小型项目 | 大型全栈 |

**异步工作原理**：

```python
# 同步 vs 异步对比
@app.get("/sync")
def sync_endpoint():
    # 同步：阻塞线程
    data = blocking_io_call()
    return data

@app.get("/async")
async def async_endpoint():
    # 异步：不阻塞，可在单线程处理更多请求
    data = await async_io_call()
    return data
```

**在 Agent 工程中的应用**：

```python
# 作为 Agent 的 Tool Server
class CodeRequest(BaseModel):
    code: str
    language: str

@app.post("/tools/execute_code")
async def execute_code(request: CodeRequest):
    result = await code_executor.run(request.code, request.language)
    return {"output": result}

# 启动：uvicorn main:app --host 0.0.0.0 --port 8000
```

### 7.1.2 FastAPI 在 Agent 项目中的作用

**核心定位**：FastAPI 通常作为 Agent 系统的 **HTTP API 层** 或 **Tool Server**，为 Agent 提供工具调用接口。

**典型架构**：

```
┌─────────────────────────────────────────────────────────────────┐
│                        Claude Code / Agent                      │
│                          (Harness 框架)                          │
└─────────────────────────┬───────────────────────────────────────┘
                          │  HTTP / Function Call
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                      FastAPI Tool Server                         │
├─────────────────────────────────────────────────────────────────┤
│  @app.post("/tools/execute_code")    → 代码执行工具              │
│  @app.post("/tools/search")          → 搜索工具                  │
│  @app.post("/tools/file_operations") → 文件操作工具              │
│  @app.post("/tools/web_fetch")       → 网页获取工具              │
│  @app.post("/tools/database")        → 数据库工具                │
└─────────────────────────────────────────────────────────────────┘
```

**FastAPI 在 Agent 项目中的具体作用**：

#### 1. 工具服务器（Tool Server）

```python
# 统一的工具服务化
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List, Dict, Any

app = FastAPI(title="Agent Tools Server")

class ToolRequest(BaseModel):
    tool_name: str
    parameters: Dict[str, Any]
    context: Dict[str, Any] = {}  # 可选的执行上下文

class ToolResponse(BaseModel):
    success: bool
    result: Any
    error: str = None
    metadata: Dict[str, Any] = {}

@app.post("/v1/tools/execute", response_model=ToolResponse)
async def execute_tool(request: ToolRequest):
    """Agent 调用工具的统一入口"""
    tool = tool_registry.get(request.tool_name)
    if not tool:
        raise HTTPException(status_code=404, detail=f"Tool {request.tool_name} not found")

    try:
        result = await tool.execute(request.parameters, request.context)
        return ToolResponse(success=True, result=result)
    except Exception as e:
        return ToolResponse(success=False, error=str(e))

@app.post("/v1/tools/batch", response_model=List[ToolResponse])
async def batch_execute_tools(requests: List[ToolRequest]):
    """批量工具调用，支持并行执行"""
    tasks = [execute_tool(req) for req in requests]
    return await asyncio.gather(*tasks)
```

#### 2. MCP Server 实现

```python
# 基于 FastAPI 实现 MCP (Model Context Protocol) Server
from fastapi import FastAPI
from sse_starlette import EventSourceResponse

app = FastAPI()

@app.get("/mcp/v1/resources")
async def list_resources():
    """MCP 资源列表"""
    return {
        "resources": [
            {"uri": "file:///project", "name": "Project Files"},
            {"uri": "db:///users", "name": "User Database"},
        ]
    }

@app.get("/mcp/v1/resources/{path}")
async def read_resource(path: str):
    """读取 MCP 资源"""
    if path.startswith("file://"):
        return {"contents": read_file(path[7:])}
    elif path.startswith("db://"):
        return {"contents": query_database(path[5:])}

@app.post("/mcp/v1/tools/call")
async def call_mcp_tool(tool_name: str, arguments: dict):
    """调用 MCP 工具"""
    tool = mcp_registry.get(tool_name)
    return await tool.call(arguments)

@app.get("/mcp/v1/events")
async def mcp_events():
    """Server-Sent Events 用于实时推送"""
    async def event_generator():
        while True:
            event = await event_queue.get()
            yield {"event": "message", "data": json.dumps(event)}
    return EventSourceResponse(event_generator())
```

#### 3. Agent API 网关

```python
# Agent 系统的统一入口
@app.post("/api/v1/agent/run")
async def run_agent(request: AgentRequest):
    """启动 Agent 执行任务"""
    agent_id = await agent_manager.create(request)
    return {"agent_id": agent_id, "status": "running"}

@app.get("/api/v1/agent/{agent_id}/status")
async def get_agent_status(agent_id: str):
    """查询 Agent 状态"""
    return await agent_manager.get_status(agent_id)

@app.get("/api/v1/agent/{agent_id}/stream")
async def agent_stream(agent_id: str):
    """流式输出 Agent 执行过程"""
    async def event_stream():
        async for token in agent_manager.stream(agent_id):
            yield {"event": "token", "data": token}
    return EventSourceResponse(event_stream())

@app.post("/api/v1/agent/{agent_id}/interrupt")
async def interrupt_agent(agent_id: str):
    """打断 Agent 执行"""
    await agent_manager.interrupt(agent_id)
    return {"status": "interrupted"}
```

#### 4. Webhook 与回调

```python
# Agent 完成任务后的回调机制
@app.post("/webhook/agent/complete")
async def agent_complete(webhook: WebhookPayload):
    """Agent 完成后通知外部系统"""
    # 发送邮件、Slack 消息、触发 CI/CD 等
    await notification_service.send(webhook)

@app.post("/webhook/tool/result")
async def tool_resultallback(callback: ToolCallback):
    """长时间运行工具的回调"""
    await task_queue.complete(callback.task_id, callback.result)
```

**为什么 Agent 项目选择 FastAPI**：

| 原因 | 说明 |
|------|------|
| **异步原生** | Agent 工具调用多是 I/O 密集型，async 大幅提升吞吐 |
| **类型安全** | Pydantic 自动验证输入输出，减少 Tool Call 错误 |
| **自动文档** | Agent 需要频繁调试，Swagger UI 提供便捷的测试界面 |
| **易于集成** | 可快速包装现有 Python 工具为 REST API |
| **Streaming** | 支持 Server-Sent Events，实时推送 Agent 输出 |

---

### 7.2 TCP 三次握手

#### 详细过程

```
TCP 三次握手建立连接
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Client                                    Server
  │                                        │
  │  ─────── 1. SYN (seq=x) ─────────────→ │  Client 发送 SYN，请求建立连接
  │       seq=x, SYN=1, ACK=0              │  消耗一个序列号
  │                                        │
  │  ←────── 2. SYN-ACK (seq=y, ack=x+1) ──│  Server 收到 SYN，回复 SYN-ACK
  │       seq=y, ack=x+1, SYN=1, ACK=1     │  ack=x+1 表示确认收到 Client 数据
  │                                        │
  │  ─────── 3. ACK (ack=y+1) ────────────→ │  Client 发送 ACK，连接建立完成
  │       seq=x+1, ack=y+1, ACK=1           │  消耗一个序列号
  │                                        │
  ▼                                        ▼
  Client 进入 ESTABLISHED 状态        Server 进入 ESTABLISHED 状态
                    双方可以开始传输数据
```

#### 核心字段说明

| 字段          | 含义                  | 作用                                         |
| ------------- | --------------------- | -------------------------------------------- |
| **SYN** | Synchronize           | 请求建立连接，置 1 表示这是一个连接请求      |
| **ACK** | Acknowledgment        | 确认标志，置 1 表示确认号字段有效            |
| **seq** | Sequence Number       | 序列号，当前报文段的数据部分第一个字节的序号 |
| **ack** | Acknowledgment Number | 确认号，期望收到的下一个字节的序列号         |

#### 为什么需要三次握手？

**1. 同步双方序列号**

- Client 选择初始序列号 x，发送 SYN
- Server 选择初始序列号 y，回复 SYN-ACK（包含 y）
- 如果只有两次握手，Server 无法知道 Client 是否收到了自己的序列号 y

**2. 确认双方收发能力**

- 第一次握手：Client → Server，Server 知道 Client 能发
- 第二次握手：Server → Client，Client 知道 Server 能发能收
- 第三次握手：Client → Server，Server 知道 Client 能收

**3. 避免历史连接干扰**

- 如果 Client 发送了旧连接的 SYN
- Server 会回复 SYN-ACK，但 Client 检查 ack 不对会发送 RST
- 只有三次握手才能避免旧数据被误认为有效数据

#### 半连接队列与全连接队列

```
Client                          Server
   │                               │
   │  SYN →                       │  Server 收到 SYN，进入 SYN_RCVD 状态
   │                               │  连接放入 "半连接队列" (SYN Queue)
   │                               │
   │                    ACK ←     │  三次握手完成
   │                               │  连接从半连接队列移到 "全连接队列" (Accept Queue)
   │                               │
   │                               ▼
   │                         应用调用 accept() 取走连接
```

**常见问题**：

- **SYN Flood 攻击**：攻击者发送大量 SYN 不完成三次握手，占满半连接队列
- **解决方案**：SYN Cookie技术，不分配队列空间

#### 四次挥手（对比理解）

```
Client                                              Server
  │                                                   │
  │  ─────────────── FIN (seq=u) ────────────────→  │  Client 发送 FIN，请求关闭
  │                 ←────────────── ACK (ack=u+1) ──│  Server 确认，Client 进入 FIN_WAIT_2
  │                                                   │
  │                 ←────────────── FIN ────────────│  Server 发送 FIN，关闭 Server→Client
  │  ─────────────── ACK (ack=v+1) ───────────────→  │  Client 确认，进入 TIME_WAIT
  │                                                   │
  ▼                                                   ▼
  等待 2MSL                                           关闭完成
  然后关闭
```

**为什么挥手需要四次？**

- TCP 是全双工，每个方向需要单独关闭
- 收到 FIN 时只能表示对方不再发送，但自己可能还要发送数据
- 因此每个方向的 FIN 和 ACK 需要分开

---

### 7.2 线程阻塞和等待的区别

| 方面               | 阻塞             | 等待       |
| ------------------ | ---------------- | ---------- |
| **主动行为** | 被动             | 主动       |
| **资源占用** | 持有资源         | 释放资源   |
| **典型场景** | I/O 等待、锁等待 | 条件变量   |
| **唤醒方式** | 自动/被中断      | 必须被唤醒 |

**线程池线程阻塞是否出问题**：

- 偶尔阻塞：正常（等待任务）
- 大量持续阻塞：可能出问题（任务不足、死锁）
- 排查：线程 dump、监控等待队列

---

## 八、MCP 和 Skills

### 8.1 项目中有没有用到 MCP 和 Skills

**回答思路**：

- 如果用过：描述具体使用场景和效果
- 如果没用过：说明为什么选择其他方案（如直接调用）

**核心区别**：

- MCP：协议标准，适合工具生态
- Skills：简单直接，适合定制化

---

## 九、个人与 AI Coding 看法

### 9.1 对 AI Coding 的看法

**正面**：

- 大幅提升开发效率
- 降低编程门槛
- 让开发者聚焦创造性工作

**局限**：

- 复杂系统设计仍需人类
- 容易产生幻觉代码
- 安全漏洞风险

**未来趋势**：

- 更智能的上下文理解
- 多 agent 协作
- 端到端自动化

---

## 十、面试技巧提示

### 答题结构建议

1. **先定义概念**：先解释基本概念
2. **再谈原理**：解释技术原理
3. **结合实践**：举出实际项目中的例子
4. **总结升华**：总结要点或给出建议

### 注意事项

1. **诚实回答**：不会的明确说，不要瞎编
2. **展示思考**：展示分析问题的过程
3. **结合项目**：所有问题尽量结合自己项目
4. **主动拓展**：回答后可以主动补充相关点
5. **保持自信**：即使不会也展示学习意愿
