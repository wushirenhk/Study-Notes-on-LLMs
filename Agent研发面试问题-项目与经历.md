# Agent 研发面试问题 - 项目与经历

---

# 腾讯客户端QQ一面：

RAG如何压缩的，何时压缩的
什么时候采用主Agent+多个副Agent？
什么时候多个Agent结合？
什么时候用单个Agent，什么时候用MutiAgent？
C++多模块继承需要注意的地方
ClaudeCode源码泄露学到的技术？
ClaudeCode相较于其他技术的护城河在于？

了解过多线程吗？

简要介绍一下TCP协议

什么是harness engineering工程，简要介绍一下

数据库中存储索引的数据结构

## 一、项目经历

### 1. 你在项目中负责的内容是什么？

**回答思路**：

1. 明确角色：Agent 系统开发 / RAG 系统优化 / 工具平台搭建
2. 核心职责：架构设计 / 核心模块实现 / 性能优化
3. 技术亮点：解决了什么难题

**参考回答**：

> 我在项目中主要负责 Agent 系统的核心架构设计，包括：
>
> 1. **上下文管理模块**：设计并实现了分层上下文管理，支持会话级摘要压缩和跨会话长期记忆
> 2. **工具调用框架**：搭建了统一的 Tool Registry，实现了 Skill 动态加载和路由
> 3. **Harness 框架**：封装了 Session Manager、Safety Guard、Hooks System 等基础组件
> 4. **RAG 检索优化**：实现了混合检索 + 两阶段重排，召回率提升 XX%

---

### 2. RAG 项目中最难的点是什么？

**核心难点**：

1. **召回质量 vs 检索速度的平衡**

   - 向量检索精度高但慢
   - 稀疏检索快但精度有限
   - 需要混合策略 + 量化加速
2. **上下文窗口限制**

   - 召回内容过多无法全部输入
   - 需要智能选择最相关的片段
3. **多跳推理**

   - 简单 QA 无法满足复杂推理需求
   - 需要多次检索 + 推理联动
4. **领域适配**

   - 通用 Embedding 在垂直领域效果差
   - 需要领域微调

```python
# RAG 核心难点示意
class RAGChallenge:
    """RAG 项目的核心挑战"""

    challenge_1 = {
        "name": "召回质量 vs 速度",
        "problem": "向量检索精度高但 QPS 低，BM25 快但语义理解差",
        "solution": "混合检索 + 量化 + 两阶段重排"
    }

    challenge_2 = {
        "name": "上下文窗口限制",
        "problem": "召回数十条内容超过模型窗口",
        "solution": "摘要压缩 + 相关性过滤 + 分层输入"
    }

    challenge_3 = {
        "name": "多跳推理",
        "problem": "单次检索无法满足复杂问答",
        "solution": "Iterative RAG / Agent-based RAG"
    }
```

---

### 3. 检索是否基于向量实现？完整的 RAG 系统核心部分？

**回答**：是的，核心基于向量检索，结合稀疏检索。

**完整 RAG 架构**：

```
┌─────────────────────────────────────────────────────────────────┐
│                        RAG 系统架构                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│  │   文档上传   │→   │    分块     │→   │   向量化    │        │
│  │  (Upload)   │    │  (Chunk)   │    │  (Embed)   │        │
│  └─────────────┘    └─────────────┘    └──────┬──────┘        │
│                                                │                │
│                                                ▼                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    向量数据库                             │   │
│  │  (Chroma / Milvus / Pinecone / FAISS / Elasticsearch)   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                  │
│                              │                                  │
│  ┌─────────────┐    ┌───────┴───────┐    ┌─────────────┐     │
│  │   用户查询   │→   │   混合检索    │→   │    重排     │     │
│  │   (Query)   │    │ (Hybrid)      │    │  (Rerank)  │     │
│  └─────────────┘    └───────────────┘    └──────┬──────┘     │
│                                                  │              │
│                                                  ▼              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                      LLM 生成                          │   │
│  │          (Context + Query → Answer + Citation)         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**核心组件**：

| 组件                | 技术选型                     | 作用                 |
| ------------------- | ---------------------------- | -------------------- |
| **文档解析**  | PDFMiner / Unstructured      | 提取文本、表格、图像 |
| **分块策略**  | 滑动窗口 / 语义分块          | 合理切分文档         |
| **Embedding** | text-embedding-3 / BGE       | 文本向量化           |
| **向量存储**  | Chroma / Milvus              | 高效相似度检索       |
| **稀疏检索**  | BM25 / Elasticsearch         | 关键词精确匹配       |
| **重排模型**  | BGE-Reranker / Cross-Encoder | 精细化排序           |
| **生成模型**  | GPT-4 / Claude / Qwen        | 答案生成             |

---

### 4. RAG 项目的文档上传和分块是怎么实现的？

**文档上传流程**：

```python
class DocumentUploader:
    """文档上传处理流程"""

    async def upload(self, file: UploadFile) -> Document:
        # 1. 文件类型检测
        file_type = self.detect_type(file)

        # 2. 解析内容
        if file_type == "pdf":
            content = await self.parse_pdf(file)
        elif file_type == "docx":
            content = await self.parse_docx(file)
        elif file_type == "markdown":
            content = await self.parse_markdown(file)

        # 3. 清洗数据
        cleaned = self.clean(content)

        # 4. 分块处理
        chunks = self.chunker.chunk(cleaned)

        # 5. 向量化
        embeddings = await self.embedder.embed_batch([c.text for c in chunks])

        # 6. 存储
        doc = Document(id=uuid(), chunks=chunks, embeddings=embeddings)
        await self.vector_store.add(doc)

        return doc
```

**分块策略**：

```python
class ChunkingStrategy:
    """多种分块策略"""

    # 策略1：固定窗口分块
    def fixed_window(self, text: str, chunk_size: int = 500, overlap: int = 50):
        """简单但可能有句子截断"""
        chunks = []
        for i in range(0, len(text), chunk_size - overlap):
            chunks.append(text[i:i + chunk_size])
        return chunks

    # 策略2：语义分块（按段落/句子）
    def semantic_chunk(self, text: str, max_tokens: int = 500):
        """保持语义完整性"""
        paragraphs = text.split("\n\n")
        chunks = []
        current_chunk = []

        for para in paragraphs:
            para_tokens = self.count_tokens(para)
            current_tokens = sum(self.count_tokens(c) for c in current_chunk)

            if current_tokens + para_tokens <= max_tokens:
                current_chunk.append(para)
            else:
                if current_chunk:
                    chunks.append("\n\n".join(current_chunk))
                current_chunk = [para]

        if current_chunk:
            chunks.append("\n\n".join(current_chunk))

        return chunks

    # 策略3：递归分块（层次化）
    def recursive_chunk(self, text: str, separators: List[str] = None):
        """按优先级尝试不同分隔符"""
        if separators is None:
            separators = ["\n\n", "\n", ". ", " "]

        def split(text, sep):
            if not sep:
                return [text]
            parts = text.split(sep)
            result = []
            for p in parts:
                if self.count_tokens(p) <= 500:
                    result.append(p)
                else:
                    result.extend(split(p, sep[1:]))
            return result

        return split(text, separators)
```

**分块参数选择**：

| 参数           | 经验值         | 说明             |
| -------------- | -------------- | ---------------- |
| chunk_size     | 500-800 tokens | 平衡信息和完整性 |
| overlap        | 50-100 tokens  | 保持上下文连续性 |
| min_chunk_size | 100 tokens     | 过小丢弃         |

---

### 5. 向量检索召回的单次耗时？有没有用到 Rerank 模型？

**性能数据**：

| 阶段                | 耗时                | 说明               |
| ------------------- | ------------------- | ------------------ |
| 向量检索 (top 100)  | 50-100ms            | 取决于向量库和索引 |
| BM25 检索 (top 100) | 10-20ms             | Elasticsearch      |
| RRF 融合            | 5ms                 | 轻量计算           |
| Rerank (top 20)     | 200-500ms           | Cross-Encoder      |
| **总计**      | **300-600ms** | 端到端召回         |

**是否使用 Rerank**：是的，采用两阶段召回

```python
class TwoStageRetrieval:
    """两阶段召回 + 重排"""

    def __init__(self):
        self.vector_store = Milvus()      # 向量检索
        self.bm25 = Elasticsearch()        # 稀疏检索
        self.reranker = BGEReranker()     # 重排模型

    async def retrieve(self, query: str, top_k: int = 5) -> List[Chunk]:
        # Stage 1: 多路召回
        vector_results = await self.vector_store.search(query, top_k=50)
        bm25_results = await self.bm25.search(query, top_k=50)

        # RRF 融合
        fused = self.rrf_fusion([vector_results, bm25_results], top_k=20)

        # Stage 2: Cross-Encoder 重排
        reranked = await self.reranker.rerank(query, fused, top_k=top_k)

        return reranked
```

---

### 6. 如何评估检索召回内容与用户问题的匹配度？

**评估方法**：

```python
class RetrievalEvaluation:
    """检索召回评估体系"""

    # 1. 离线评估指标
    def offline_metrics(self):
        return {
            "Hit@K": "Top-K 结果中包含正确答案的比例",
            "MRR": "平均倒数排名，正确答案越靠前分数越高",
            "NDCG": "考虑位置权重的评估指标",
            "Recall@K": "Top-K 结果召回的相关文档比例",
        }

    # 2. 在线 A/B 测试
    async def online_ab_test(self, variant_a: str, variant_b: str):
        """
        A 组：纯向量检索
        B 组：混合检索 + Rerank
        评估指标：用户满意度、答案准确率、幻觉率
        """

    # 3. LLM 作为裁判
    async def llm_judge(self, query: str, retrieved_docs: List[str]) -> dict:
        """用大模型评估召回内容的相关性"""
        prompt = f"""评估以下检索结果与用户问题的相关性：

用户问题：{query}

检索内容：
{doc1}
{doc2}
...

请评估每个检索内容：
- 是否相关
- 相关程度（高/中/低）
- 理由
"""
        return self.llm.generate(prompt)

    # 4. 案例分析
    def case_study(self, query: str):
        """
        定期抽检典型 case：
        - 高频 query 的召回质量
        - 失败 case 的根因分析
        """
```

**评估维度**：

| 维度     | 指标           | 采集方式   |
| -------- | -------------- | ---------- |
| 召回精度 | Hit@K, MRR     | 标注数据集 |
| 排序质量 | NDCG           | 标注数据集 |
| 用户感知 | 满意度         | 用户反馈   |
| 答案质量 | 准确率、幻觉率 | LLM 评估   |

---

## 二、Agent 相关

### 7. 你对 Agent 的理解是什么，包含哪些核心模块？

**Agent 定义**：

> Agent = LLM + Planning + Memory + Tool Use
> Agent 是能够自主决策、规划、执行任务的智能系统。

**核心模块**：

```
┌─────────────────────────────────────────────────────────────────┐
│                         Agent 核心架构                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                      ┌─────────────────┐                        │
│                      │   Planning      │                        │
│                      │   - 任务分解     │                        │
│                      │   - 自我反思     │                        │
│                      └────────┬────────┘                        │
│                               │                                 │
│  ┌────────────────────────────┼────────────────────────────┐   │
│  │                            ▼                             │   │
│  │  ┌─────────────────────────────────────────────────────┐│   │
│  │  │                    Core LLM                          ││   │
│  │  │              (决策 + 生成 + 调用)                    ││   │
│  │  └─────────────────────────────────────────────────────┘│   │
│  │                            │                             │   │
│  └────────────────────────────┼────────────────────────────┘   │
│                               │                                 │
│         ┌─────────────────────┼─────────────────────┐           │
│         ▼                     ▼                     ▼           │
│  ┌─────────────┐       ┌─────────────┐       ┌─────────────┐   │
│  │   Memory    │       │    Tool     │       │   Safety    │   │
│  │             │       │   Use       │       │             │   │
│  │ 短期/长期   │       │   工具调用   │       │  操作审计   │   │
│  └─────────────┘       └─────────────┘       └─────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**模块详解**：

| 模块               | 职责                           | 关键技术                |
| ------------------ | ------------------------------ | ----------------------- |
| **Planning** | 任务分解、步骤规划、自我反思   | ReAct, Reflexion, CoT   |
| **Memory**   | 短期会话、长期积累、上下文管理 | 摘要压缩、向量存储      |
| **Tool Use** | 工具发现、调用、结果解析       | Function Call, MCP      |
| **Safety**   | 权限控制、危险检测、审计       | 正则匹配、权限矩阵      |
| **Harness**  | 会话管理、流式输出、生命周期   | FastAPI, SSE, WebSocket |

---

### 8. 是否了解 Agent 的设计范式，例如 ReAct 范式？

**主流 Agent 设计范式**：

| 范式                       | 核心思想                 | 适用场景           |
| -------------------------- | ------------------------ | ------------------ |
| **ReAct**            | Reasoning 和 Action 交替 | 工具调用、复杂推理 |
| **Plan-and-Execute** | 先规划后执行             | 长周期任务         |
| **Reflexion**        | 自我反思 + 记忆          | 试错学习           |
| **AutoGPT**          | 自主目标分解             | 开放式任务         |
| **Voyager**          | 持续技能习得             | 具身智能           |

**ReAct 范式详解**：

```python
class ReActAgent:
    """ReAct (Reasoning + Acting) 范式"""

    def run(self, task: str, max_steps: int = 10):
        """ReAct 循环"""
        observation = ""
        thought_chain = []

        for step in range(max_steps):
            # Step 1: Thought - 推理
            thought = self.think(task, observation, thought_chain)
            thought_chain.append(thought)

            # Step 2: Action - 选择工具并执行
            if "action" in thought:
                tool_name = thought["action"]["name"]
                tool_args = thought["action"]["args"]
                result = await self.execute_tool(tool_name, tool_args)
                observation = f"Observation: {result}"

            # Step 3: 判断是否完成
            if self.is_finished(thought):
                return thought["response"]

        return "达到最大步数未完成"

    def think(self, task, observation, history) -> dict:
        """LLM 生成 Thought"""
        prompt = f"""任务：{task}

历史推理：
{chr(10).join(history)}

观察：{observation}

请进行推理（Thought），决定下一步行动（Action）：
"""
        return self.llm.generate_json(prompt)
```

**ReAct 的标准输出格式**：

```
Thought: 我需要先搜索相关的政策文件来回答这个问题。
Action: search
Action Input: {"query": "2024年新能源汽车补贴政策"}
Observation: 根据搜索结果，2024年补贴标准为...
Thought: 搜索结果提到了补贴金额，但用户询问的是申请条件，我需要进一步查找。
Action: search
Action Input: {"query": "新能源汽车补贴申请条件"}
...
```

---

### 9. Agent 循环一般多少步完成任务？有没有达到最大步数仍无法完成的情况？

**Agent 循环步数统计**：

```python
class AgentLoopStats:
    """Agent 循环统计"""

    STATS = {
        "简单任务(单次工具调用)": "1-3 步",
        "中等任务(2-3个工具)": "3-7 步",
        "复杂任务(多步推理)": "7-15 步",
        "开放任务(自主探索)": "15-30+ 步",
    }

    MAX_STEPS_DEFAULT = 20  # 默认最大步数

    def handle_max_steps_exceeded(self):
        """达到最大步数的处理策略"""
        # 1. 尝试摘要当前进度
        progress_summary = self.summarize_progress()

        # 2. 请求用户确认
        ask_user = f"""已达到最大步数 {self.MAX_STEPS}，当前进度：

{progress_summary}

请选择：
1. 继续执行（增加步数限制）
2. 基于当前进度生成答案
3. 放弃任务
"""
        return self.prompt_user(ask_user)
```

**达到最大步数的原因**：

| 原因             | 解决方案             |
| ---------------- | -------------------- |
| 任务过于复杂     | 拆分为多个子任务     |
| 工具选择错误     | 增加路由准确性       |
| 进入死循环       | 添加去重检测         |
| LLM 推理能力不足 | 优化 Prompt 或换模型 |

**防止死循环的代码**：

```python
class LoopDetector:
    """循环检测"""

    def __init__(self, max_same_state=3, max_iterations=20):
        self.seen_states = {}
        self.iteration = 0

    def check_and_register(self, state: str) -> bool:
        """
        检测是否陷入循环
        返回 True 表示正常，返回 False 表示检测到循环
        """
        self.iteration += 1

        if self.iteration > max_iterations:
            return False  # 超总步数

        state_key = self._hash_state(state)
        count = self.seen_states.get(state_key, 0) + 1
        self.seen_states[state_key] = count

        if count > max_same_state:
            return False  # 同一状态重复超过阈值

        return True
```

---

### 10. Agent 目前接入了哪些工具？

**常见工具分类**：

```python
class AgentTools:
    """Agent 工具分类"""

    # 1. 搜索类
    SEARCH_TOOLS = [
        "web_search",      # 网页搜索
        "code_search",     # 代码搜索
        "document_search", # 文档搜索
    ]

    # 2. 文件类
    FILE_TOOLS = [
        "file_read",       # 读文件
        "file_write",      # 写文件
        "file_list",       # 列出目录
        "file_search",     # 文件内容搜索
    ]

    # 3. 执行类
    EXEC_TOOLS = [
        "bash",            # 执行命令
        "python",          # 执行 Python
        "code_execute",    # 执行代码
    ]

    # 4. API 类
    API_TOOLS = [
        "http_request",    # HTTP 请求
        "mcp_tool",        # MCP 工具调用
    ]

    # 5. 知识类
    KNOWLEDGE_TOOLS = [
        "rag_retrieve",    # RAG 检索
        "knowledge_graph", # 知识图谱
        "database_query",  # 数据库查询
    ]
```

---

### 11. 如何约定并约束大模型进行工具调用？

**核心策略：Prompt Engineering + Schema 约束**

```python
class ToolCallConstraint:
    """工具调用约束机制"""

    # 1. System Prompt 约束
    SYSTEM_PROMPT = """你是一个助手，必须在需要时调用工具。

可用工具：
{tool_schemas}

规则：
1. 只在必要时调用工具
2. 工具参数必须完整准确
3. 工具调用格式必须严格遵循 JSON 格式
4. 不确定时不要捏造工具名称
"""

    # 2. Tool Schema 定义
    TOOL_SCHEMA = {
        "type": "object",
        "properties": {
            "name": {
                "type": "string",
                "enum": ["search", "read_file", "bash"],  # 枚举限制
                "description": "工具名称"
            },
            "arguments": {
                "type": "object",
                "required": ["query"],  # 必填参数
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "搜索查询词"
                    }
                }
            }
        }
    }

    # 3. 输出解析 + 校验
    def parse_tool_call(self, llm_output: str) -> Optional[ToolCall]:
        """解析并校验 LLM 输出"""
        try:
            data = json.loads(llm_output)
        except json.JSONDecodeError:
            # LLM 没有输出有效 JSON，尝试修复
            data = self._try_fix_json(llm_output)

        # Schema 校验
        if self.validate_schema(data, TOOL_SCHEMA):
            return ToolCall(**data)
        else:
            return None

    # 4. 错误重试 + 纠错
    def handle_tool_call_error(self, error: ToolError) -> str:
        """工具调用失败时的纠错"""
        prompt = f"""工具调用失败：
错误：{error.message}

请重新调用工具，修正以下问题：
{error.suggestion}

只输出 JSON 格式的 tool_call，不要有其他内容。
"""
        return self.llm.generate(prompt)
```

**多层次约束**：

1. **Prompt 层**：System Prompt 明确规则
2. **Schema 层**：JSON Schema 限制格式
3. **Parser 层**：解析时校验 + 修复
4. **Execution 层**：工具执行时的权限检查
5. **Feedback 层**：失败后提供纠错信息

---

### 12. 项目推流是否使用 SSE？

**SSE vs WebSocket vs HTTP**:

| 方案                     | 协议   | 方向                   | 使用场景           |
| ------------------------ | ------ | ---------------------- | ------------------ |
| **SSE**            | HTTP   | 单向（Server→Client） | 实时日志、推送通知 |
| **WebSocket**      | WS/WSS | 双向                   | 实时聊天、协作编辑 |
| **HTTP 轮询**      | HTTP   | 轮询                   | 简单场景           |
| **gRPC Streaming** | HTTP/2 | 双向                   | 微服务内部         |

**Agent 场景推荐**：

- **SSE**：Agent 流式输出、日志推送（推荐）
- **WebSocket**：需要用户实时输入的场景

```python
# FastAPI SSE 实现
from fastapi import FastAPI
from sse_starlette import EventSourceResponse

app = FastAPI()

@app.get("/agent/stream/{session_id}")
async def agent_stream(session_id: str):
    """SSE 流式输出 Agent 执行过程"""

    async def event_generator():
        agent = await agent_manager.get(session_id)

        async for token in agent.stream():
            # 发送 token 事件
            yield {
                "event": "token",
                "data": token
            }

        # 发送完成事件
        yield {
            "event": "done",
            "data": json.dumps({"status": "completed"})
        }

    return EventSourceResponse(event_generator())

# 前端调用
# const eventSource = new EventSource('/agent/stream/session_123');
# eventSource.addEventListener('token', (e) => appendToken(e.data));
# eventSource.addEventListener('done', (e) => finish());
```

---

### 13. Agent 编排流程中有没有做 Plan 阶段？

**Plan 阶段的作用**：

```python
class PlanStage:
    """任务规划阶段"""

    async def plan(self, task: str) -> Plan:
        """
        在执行前，先让 Agent 规划执行步骤
        """
        prompt = f"""任务：{task}

请制定执行计划，包括：
1. 分解为哪些子任务
2. 执行顺序
3. 每个子任务需要什么工具
4. 预计的难点

输出 JSON 格式：
{{
  "steps": [
    {{"id": 1, "task": "xxx", "tool": "yyy", "expected_output": "zzz"}}
  ],
  "total_steps": 5,
  "difficulty": "medium"
}}
"""
        plan = await self.llm.generate_json(prompt)

        # 用户确认
        self.prompt_user(f"执行计划：\n{plan.to_display()}\n\n是否继续？")

        return plan

    async def execute_with_plan(self, plan: Plan):
        """按计划执行"""
        for step in plan.steps:
            # 执行当前步骤
            result = await self.execute_step(step)

            # 检查是否符合预期
            if not self.validate_result(step, result):
                # 不符合则重新规划
                new_plan = await self.replan(step, result)
                # 递归执行新计划
```

**Plan vs No Plan**：

| 场景           | 推荐    | 原因            |
| -------------- | ------- | --------------- |
| 简单明确任务   | No Plan | 规划开销不值得  |
| 复杂多步骤任务 | Plan    | 需要整体规划    |
| 开放式探索     | No Plan | 难以提前规划    |
| 用户可介入     | Plan    | 计划 + 用户确认 |

---

### 14. 从协议层面，介绍 SSE、WebSocket 与 HTTP 的区别和关联？

**协议对比**：

```
┌─────────────────────────────────────────────────────────────────┐
│                     HTTP / SSE / WebSocket 对比                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  HTTP (短连接)                                                  │
│  ────────────                                                   │
│  Client ──────────────────────────────→ Server                   │
│       ←───────────────────────────────────────                   │
│                     Response (一次请求-一次响应)                   │
│                                                                 │
│  HTTP/1.1 Keep-Alive: 复用 TCP 连接，但本质仍是 请求-响应         │
│  HTTP/2: 多路复用，一个连接内并行多个请求                          │
│  HTTP/3: QUIC 协议，UDP                                        │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  SSE (Server-Sent Events，单向)                                 │
│  ────────────────────────────────────────                        │
│  Client ──────────────────────────────→ Server                   │
│       ←═════════════════════════════════════                     │
│                     多个 Events (Server 持续推送)                 │
│                                                                 │
│  连接：HTTP 长连接，Content-Type: text/event-stream              │
│  断开：自动重连机制                                              │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  WebSocket (全双工)                                             │
│  ────────────────────────────────────────                        │
│  建立连接：                                                    │
│  Client ──HTTP Upgrade 请求──────────→ Server                     │
│       ←───101 Switching Protocols ─────                          │
│  之后：                                                        │
│  Client ◄═════════════════════════════► Server                   │
│           双向数据传输，不受 HTTP 限制                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**详细对比**：

| 方面                 | HTTP           | SSE            | WebSocket      |
| -------------------- | -------------- | -------------- | -------------- |
| **连接类型**   | 短连接/轮询    | 长连接（单向） | 长连接（双向） |
| **通信方向**   | Client→Server | Server→Client | 双向           |
| **实时性**     | 轮询延迟       | 近实时         | 实时           |
| **开销**       | 每次请求头     | 较低           | 最低           |
| **断线重连**   | 需手动处理     | 自动           | 需手动处理     |
| **适用场景**   | 简单请求       | 推送、日志     | 实时聊天、游戏 |
| **兼容性好**   | 所有浏览器     | 现代浏览器     | 现代浏览器     |
| **二进制支持** | Base64         | 不支持         | 支持           |

**Agent 场景选择**：

```python
# 推荐策略
RECOMMENDATIONS = {
    # Agent 流式输出：SSE（单向推送，开销小）
    "agent_stream": "SSE",

    # 需要用户实时输入：WebSocket
    "interactive_chat": "WebSocket",

    # 简单 API 调用：HTTP
    "tool_call_sync": "HTTP",
}
```

---

### 15. 项目为什么限制每个用户只能上传单个文件？

**设计考量**：

1. **技术原因**

   - 多文件并发上传增加服务端复杂度
   - 文件关联和去重处理困难
2. **业务原因**

   - 简化 RAG 处理流程（单文档线性处理）
   - 避免文件间冲突
   - 降低向量索引复杂度
3. **替代方案**

   - 提供文件夹/压缩包上传
   - 后台自动拆分为多个文档处理

```python
# 如果需要支持多文件
class MultiFileUploader:
    async def upload_batch(self, files: List[File], user_id: str):
        # 方案1：串行处理
        for file in files:
            await self.process_single(file)

        # 方案2：并行处理 + 向量合并
        results = await asyncio.gather(*[self.process(f) for f in files])
        merged = self.merge_vectors(results)

        # 方案3：先合并再处理
        combined_text = "\n".join([f.text for f in files])
        chunks = self.chunker.chunk(combined_text)
```

---

### 16. 项目的图像识别如何实现的？为什么不用多模态大模型？

**传统方案 vs 多模态方案**：

| 方案                 | 技术                     | 优点         | 缺点           |
| -------------------- | ------------------------ | ------------ | -------------- |
| **传统 OCR**   | PaddleOCR / Tesseract    | 快、便宜、准 | 只能识别文字   |
| **多模态 LLM** | GPT-4V / Claude / Qwen-V | 理解能力强   | 贵、慢、有幻觉 |

**推荐方案**：

```python
class ImageRecognition:
    """图像识别策略"""

    # 方案1：传统 OCR（推荐用于纯文本场景）
    def ocr(self, image) -> str:
        """PaddleOCR / EasyOCR"""
        text = paddyocr(image)
        return text

    # 方案2：多模态 LLM（用于需要理解的场景）
    def multimodal(self, image, prompt: str) -> str:
        """GPT-4V / Claude Vision"""
        response = openai.ChatCompletion.create(
            model="gpt-4-vision",
            messages=[{"role": "user", "content": [
                {"type": "image_url", "image_url": image},
                {"type": "text", "text": prompt}
            ]}]
        )
        return response

    # 方案3：混合策略（推荐）
    def hybrid(self, image, task: str):
        """
        简单文本提取 → OCR
        需要理解图像内容 → 多模态 LLM
        """
        if self.is_simple_text_extraction(task):
            return self.ocr(image)
        else:
            return self.multimodal(image, task)
```

**为什么不用多模态大模型**：

1. **成本**：OCR 几分钱，多模态 LLM 几毛钱
2. **速度**：OCR 秒级，LVM 几十秒
3. **准确性**：OCR 对文字识别准确率 > 99%
4. **幻觉**：多模态 LLM 可能误解图像内容

---

### 17. 项目中用到了哪些大语言模型？

**模型选型**：

```python
class LLMModels:
    """项目中的模型选型"""

    # 1. Agent 核心决策
    AGENT_MODEL = {
        "primary": "gpt-4-turbo",      # 能力强，响应快
        "fallback": "gpt-3.5-turbo",   # 降级使用
        "local": "qwen-72b-chat",      # 国产/私有化
    }

    # 2. 摘要生成
    SUMMARY_MODEL = {
        "primary": "gpt-3.5-turbo",     # 便宜、快速
        "local": "llama3-8b-instruct",  # 本地部署
    }

    # 3. Embedding
    EMBEDDING_MODEL = {
        "primary": "text-embedding-3-large",
        "fallback": "text-embedding-ada-002",
        "chinese": "bge-large-zh",
    }

    # 4. Rerank
    RERANK_MODEL = {
        "primary": "bge-reranker-large",
        "cross_encoder": "cross-encoder/ms-marco"
    }

    # 5. 多模态（图像理解）
    VISION_MODEL = {
        "primary": "gpt-4-vision-preview",
        "fallback": "claude-3-sonnet",
    }
```

---

## 三、Go 语言与并发

### 18. Go 语言的并发和其他语言并发的区别？

**核心哲学差异**：

| 方面               | Go                    | Java/Python       | JavaScript       |
| ------------------ | --------------------- | ----------------- | ---------------- |
| **并发模型** | Goroutine（轻量线程） | 线程 / ThreadPool | 事件循环 / Async |
| **创建成本** | 2KB Goroutine         | 1MB Thread        | 事件回调         |
| **调度**     | MPG 协程调度          | OS 线程调度       | 事件循环         |
| **内存占用** | 极低                  | 高                | 低               |
| **适用场景** | I/O 密集 + 高并发     | CPU 密集          | 异步 I/O         |

**Go 并发的核心优势**：

```go
// Go 并发：轻量、高效
func main() {
    // 创建一个 Goroutine，成本极低
    go func() {
        doWork()
    }()

    // 可以轻松创建数万个并发
    for i := 0; i < 100000; i++ {
        go process(i)
    }
}
```

**对比 Python**：

```python
# Python 多线程
import threading
# 创建成本高，每个线程约 1MB
thread = threading.Thread(target=do_work)
thread.start()

# Python 异步
import asyncio
async def main():
    await do_work()  # 协程，但有 GIL 限制
```

---

### 19. Goroutine 是什么，核心原理？

**Goroutine 定义**：

> Goroutine 是 Go 轻量级的执行单元，由 Go runtime 管理，共享同一个地址空间。

**MPG 调度模型**：

```
┌─────────────────────────────────────────────────────────────────┐
│                      Go Runtime Scheduler                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    M (Machine)      M (Machine)      M (Machine)                │
│       │                │                │                       │
│       ▼                ▼                ▼                       │
│    ┌──────┐         ┌──────┐         ┌──────┐                 │
│    │  P   │         │  P   │         │  P   │  ← Processor     │
│    │(调度)│         │(调度)│         │(调度)│    数量 = GOMAXPROCS
│    └──┬───┘         └──┬───┘         └──┬───┘                 │
│       │                │                │                       │
│       ▼                ▼                ▼                       │
│    G G G G G       G G G G G       G G G G G                   │
│    (Goroutines)    (Goroutines)    (Goroutines)                │
│                                                                 │
│    System Calls ──────────────────────────────────────────→   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**核心概念**：

| 概念                    | 说明                           |
| ----------------------- | ------------------------------ |
| **G (Goroutine)** | 轻量执行单元，保存执行栈和状态 |
| **M (Machine)**   | OS 线程，实际执行              |
| **P (Processor)** | 调度上下文，管理 G 的队列      |

**调度策略**：

1. **work stealing**：P 的队列空时，偷其他 P 的 G
2. **handoff**：G 阻塞 syscall 时，M 释放 P
3. **抢占式**：G 执行超过 10ms，被标记为可抢占

**栈管理**：

```go
// Goroutine 栈：动态增长
// 初始：2KB
// 最大：1GB（Go 1.22+）

func recursion() {
    // 栈不够时，runtime 自动扩容
    allocation := make([]byte, 1024*1024) // 1MB
    recursion()
}
```

---

### 20. 对锁的理解，锁是解决什么问题的？

**锁的本质**：解决**共享资源竞争**导致的竞态条件（Race Condition）。

**锁要解决的问题**：

```go
// 无锁问题示例
var counter = 0

func increment() {
    counter++ // 不是原子操作！
    // 实际：load → increment → store
    // 多线程下可能丢失更新
}
```

**Go 中的锁**：

```go
import "sync"

// 1. 互斥锁（Mutex）
var mu sync.Mutex
var counter int

func increment() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}

// 2. 读写锁（RWMutex）
var rwmu sync.RWMutex
var data map[string]string

func read(key string) string {
    rwmu.RLock()         // 读锁，可并发
    defer rwmu.RUnlock()
    return data[key]
}

func write(key, value string) {
    rwmu.Lock()           // 写锁，排他
    defer rwmu.Unlock()
    data[key] = value
}

// 3. 原子操作（无锁）
import "sync/atomic"
var counter atomic.Int64

func increment() {
    counter.Add(1)  // 原子加
}
```

**锁解决的问题**：

| 问题                 | 说明                   | 解决方案          |
| -------------------- | ---------------------- | ----------------- |
| **竞态条件**   | 多线程同时修改共享变量 | Mutex / RWMutex   |
| **数据不一致** | 部分更新导致状态损坏   | 事务锁            |
| **死锁**       | 多个锁相互等待         | 按顺序加锁 + 超时 |

---

### 21. 日常开发用哪种锁更多？还了解哪些锁？多机器、多进程场景的锁？

**日常使用的锁**：

| 锁类型                 | 使用频率   | 场景                   |
| ---------------------- | ---------- | ---------------------- |
| **sync.Mutex**   | ★★★★★ | 简单互斥，保护共享变量 |
| **sync.RWMutex** | ★★★☆☆ | 读多写少场景           |
| **sync.Map**     | ★★☆☆☆ | 并发安全的 map         |
| **atomic**       | ★★★★☆ | 简单计数器、标志位     |
| **channel**      | ★★★★★ | 协程间通信、信号传递   |

**其他锁类型**：

```go
// 1. Once（只执行一次）
var once sync.Once
once.Do(func() {
    // 只执行一次，常用于单例初始化
})

// 2. WaitGroup（等待一组 goroutine）
var wg sync.WaitGroup
for i := 0; i < 10; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        doWork()
    }()
}
wg.Wait() // 等待所有完成

// 3. Cond（条件变量）
var cond = sync.NewCond(&mutex)
go func() {
    cond.L.Lock()
    for !condition {
        cond.Wait()  // 等待信号
    }
    cond.L.Unlock()
}()

go func() {
    cond.L.Lock()
    condition = true
    cond.Signal()  // 通知一个
    // 或 cond.Broadcast() 通知所有
    cond.L.Unlock()
}()

// 4. Pool（对象池）
pool := sync.Pool{
    New: func() interface{} {
        return &bytes.Buffer{}
    },
}
buf := pool.Get().(*bytes.Buffer)
pool.Put(buf)
```

**多机器、多进程、分布式锁**：

| 场景                   | 解决方案                                     |
| ---------------------- | -------------------------------------------- |
| **同机器多进程** | 文件锁 (flock)、共享内存、Unix Socket        |
| **多机器**       | Redis 分布式锁、Etcd/ZooKeeper、数据库乐观锁 |
| **数据库**       | SELECT FOR UPDATE、版本戳、乐观锁            |

**Redis 分布式锁**：

```go
// Go 分布式锁实现
import "github.com/go-redis/redis/v8"

func acquireLock(ctx context.Context, rdb *redis.Client, key string, ttl time.Duration) (bool, error) {
    // SETNX + 过期时间
    result, err := rdb.SetNX(ctx, key, uuid.New(), ttl).Result()
    return result, err
}

func releaseLock(ctx context.Context, rdb *redis.Client, key string, value string) error {
    // Lua 脚本保证原子性释放
    script := `
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
    `
    _, err := rdb.Eval(ctx, script, []string{key}, value).Result()
    return err
}
```

---

## 四、协议与架构

### 22. 对 MCP、Function Call、A2A 分别是怎么理解的？

**三大协议对比**：

| 协议                    | 全称                   | 定位                 | 作用                        |
| ----------------------- | ---------------------- | -------------------- | --------------------------- |
| **MCP**           | Model Context Protocol | Agent ↔ 工具/数据源 | 标准化工具调用和数据获取    |
| **Function Call** | Function Calling       | LLM → 结构化输出    | 让 LLM 输出可执行的函数调用 |
| **A2A**           | Agent to Agent         | Agent ↔ Agent       | 多 Agent 协作通信           |

**MCP（Model Context Protocol）**：

```
┌─────────────────────────────────────────────────────────────────┐
│                         MCP 架构                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│     ┌─────────────┐          ┌─────────────┐                    │
│     │   Claude    │◄────────►│  MCP Host   │                    │
│     │   Client    │   MCP    │  (Harness)  │                    │
│     └─────────────┘          └──────┬──────┘                    │
│                                     │                            │
│                    ┌────────────────┼────────────────┐          │
│                    ▼                ▼                ▼          │
│             ┌──────────┐    ┌──────────┐    ┌──────────┐      │
│             │  MCP      │    │  MCP      │    │  MCP      │      │
│             │  Server   │    │  Server   │    │  Server   │      │
│             │ (File)    │    │ (DB)      │    │ (Git)     │      │
│             └───────────┘    └───────────┘    └───────────┘      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Function Call**：

```python
# Function Call 本质：让 LLM 输出 JSON 结构
response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "给张三发一封邮件"}],
    tools=[
        {
            "type": "function",
            "function": {
                "name": "send_email",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "to": {"type": "string"},
                        "subject": {"type": "string"},
                        "body": {"type": "string"}
                    },
                    "required": ["to", "subject", "body"]
                }
            }
        }
    ]
)

# LLM 输出：
# {
#   "tool_calls": [{
#     "id": "call_xxx",
#     "function": {
#       "name": "send_email",
#       "arguments": "{\"to\": \"zhangsan@example.com\", ...}"
#     }
#   }]
# }
```

**A2A（Agent to Agent）**：

```python
# A2A 协议：Agent 之间的通信协议
class A2AMessage:
    sender: str           # 发送方 Agent ID
    receiver: str         # 接收方 Agent ID
    content: dict          # 消息内容
    type: MessageType      # REQUEST / RESPONSE / BROADCAST
    correlation_id: str     # 用于关联请求和响应

# A2A 通信示例
async def agent_communicate():
    # Agent A 请求 Agent B 的帮助
    msg = A2AMessage(
        sender="agent_A",
        receiver="agent_B",
        content={
            "task": "分析这份代码",
            "code": "def foo(): pass"
        },
        type=MessageType.REQUEST
    )

    # 通过消息队列发送
    await message_queue.publish("agent_B", msg)

    # Agent B 处理后返回
    # Agent A 收到响应后继续执行
```

**三者关系**：

```
LLM + Function Call
        │
        ▼
   Harness 框架
        │
        ├── MCP ──────→ 外部工具/数据源
        │
        └── A2A ──────→ 其他 Agent
```

---

### 23. 项目接入了哪些 MCP 服务？有没有本地手写过 MCP 服务？

**常见 MCP 服务**：

| MCP Server              | 功能                             |
| ----------------------- | -------------------------------- |
| **Filesystem**    | 文件读写、目录操作               |
| **Git**           | Git 操作（commit、branch、diff） |
| **Database**      | SQL 查询、数据读写               |
| **Search**        | 搜索服务                         |
| **Slack/Discord** | 消息通知                         |
| **Puppeteer**     | 浏览器自动化                     |

**手写 MCP Server 示例**：

```python
# 自定义 MCP Server
from mcp.server import MCPServer
from mcp.types import Tool, Resource

class MyMCPServer(MCPServer):
    """自定义 MCP 服务"""

    def __init__(self):
        super().__init__(name="my-mcp-server")
        self.register_tool(self.get_weather)
        self.register_resource(self.get_config)

    @tool(description="查询天气")
    async def get_weather(self, location: str, unit: str = "celsius"):
        """获取天气信息"""
        weather_data = await self.weather_api.get(location, unit)
        return weather_data

    @resource(uri="config://app")
    async def get_config(self):
        """提供配置资源"""
        return {
            "version": "1.0.0",
            "env": "production"
        }

# 启动 MCP Server
server = MyMCPServer()
server.run(transport="stdio")  # 或 "sse", "http"
```

---

### 24. 日常开发借助哪些 AI 工具？写过相关的 command/skill？

**常用 AI 工具**：

| 工具                     | 用途               | 使用频率   |
| ------------------------ | ------------------ | ---------- |
| **Claude Code**    | 项目开发、代码生成 | ★★★★★ |
| **Cursor**         | 快速补全、IDE 编程 | ★★★★☆ |
| **GitHub Copilot** | 代码补全           | ★★★☆☆ |
| **ChatGPT**        | 问题解答、知识查询 | ★★★★☆ |
| **Perplexity**     | 技术调研           | ★★★☆☆ |

**Command/Skill 编写示例**：

```yaml
# 自定义 Skill: code-review
name: code-review
description: Perform a thorough code review
triggers:
  - "/review"
  - "review this code"
  - "代码审查"

actions:
  - step: "分析代码结构和设计模式"
    tool: "read_files"
  - step: "检查潜在 bug 和安全漏洞"
    tool: "semantic_search"
  - step: "评估代码可读性和性能"
    tool: "static_analysis"
  - step: "生成审查报告"
    output: "markdown"
```

---

### 25. Claude Code 的实现原理？

**核心架构**：

```
┌─────────────────────────────────────────────────────────────────┐
│                      Claude Code 架构                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     CLI Interface                        │   │
│  │                  (命令行交互层)                           │   │
│  └─────────────────────────┬───────────────────────────────┘   │
│                            │                                     │
│  ┌─────────────────────────▼───────────────────────────────┐   │
│  │                    Harness Framework                     │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │   │
│  │  │ Session  │  │ Context  │  │  Safety  │            │   │
│  │  │ Manager  │  │ Manager  │  │  Guard   │            │   │
│  │  └──────────┘  └──────────┘  └──────────┘            │   │
│  └─────────────────────────┬───────────────────────────────┘   │
│                            │                                     │
│  ┌─────────────────────────▼───────────────────────────────┐   │
│  │                    Agent Core (LLM)                       │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │   │
│  │  │Planning  │  │  Tool    │  │ Memory   │            │   │
│  │  │ Engine   │  │  Use     │  │ System   │            │   │
│  │  └──────────┘  └──────────┘  └──────────┘            │   │
│  └─────────────────────────┬───────────────────────────────┘   │
│                            │                                     │
│  ┌─────────────────────────▼───────────────────────────────┐   │
│  │                    Tool Ecosystem                         │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │   │
│  │  │  File    │  │  Bash    │  │   Git    │            │   │
│  │  │  Ops     │  │  Exec    │  │  Ops     │            │   │
│  │  └──────────┘  └──────────┘  └──────────┘            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**关键实现**：

1. **分层上下文**：只加载当前任务相关的文件
2. **流式输出**：Token 级别实时反馈
3. **安全审计**：敏感操作二次确认
4. **智能文件选择**：判断哪些文件需要读取
5. **循环检测**：防止死循环

---

### 26. 是否了解 OpenClaw？实现原理？

**OpenClaw** 是 Claude Code 的开源参考实现。

**核心设计**：

```python
class OpenClawArchitecture:
    """OpenClaw 核心架构"""

    # 1. 模块化设计
    components = {
        "agent": "LLM 核心，决策和生成",
        "harness": "会话、上下文、安全管理",
        "tools": "文件、Shell、Git 等工具",
        "mcp": "MCP 协议集成",
    }

    # 2. 关键创新
    innovations = {
        "分层上下文": "只加载相关文件，减少 token",
        "Skill 系统": "动态加载，按需注入",
        "安全优先": "操作审计，危险检测",
        "流式交互": "实时反馈，打断机制",
    }

    # 3. 与 Claude Code 差异
    differences = {
        "Claude Code": "官方闭源，品牌背书",
        "OpenClaw": "开源透明，可定制",
    }
```

---

### 27. 日常用 AI 工具做项目开发时，如何指导 AI 完成开发？

**有效指导策略**：

```python
class AIDevelopmentGuidance:
    """AI 辅助开发指导策略"""

    # 1. 任务分解策略
    TASK_BREAKDOWN = """指导 AI 的方式：

大任务 → 小任务 → 具体步骤

示例：
❌ "帮我做一个电商系统"
✅ "先帮我设计数据库结构，包含商品表、用户表、订单表"
✅ "然后实现商品表的基本 CRUD API"
✅ "再实现用户认证模块"
"""

    # 2. Prompt 模板
    PROMPT_TEMPLATE = """
## 角色
你是一个 {role}，负责 {responsibility}

## 上下文
- 技术栈：{tech_stack}
- 项目结构：{project_structure}
- 当前任务：{current_task}

## 要求
1. {requirement_1}
2. {requirement_2}

## 约束
- 遵循 {style_guide}
- 不修改 {forbidden_areas}

## 输出
{expected_output_format}
"""

    # 3. 迭代反馈
    ITERATION_FEEDBACK = """
每次 AI 输出后：
1. 检查是否符合预期
2. 不符合 → 明确指出问题 + 期望
3. 符合 → 确认 + 下一个任务

示例反馈：
❌ "不对，重写"
✅ "这部分逻辑正确，但命名不够清晰，请将 `getData()` 改为 `fetchUserProfile()`"
```

```

---

### 28. 为什么不用 Claude Code 自带的 Plan 模式做项目规划？

**Plan 模式的局限**：

| 方面 | Claude Code Plan | 外部规划工具 |
|------|------------------|-------------|
| **可控性** | 完全依赖 LLM | 可自定义规则 |
| **可编辑性** | 难以手动调整 | 随时修改 |
| **持久化** | 随会话结束消失 | 文件持久化 |
| **集成** | 封闭 | 可与项目管理工具集成 |
| **版本控制** | 无 | Git 管理 |

**实际选择**：

```python
# 推荐：结合使用
USAGE_PATTERN = {
    "small_task": "直接让 Claude Code 执行",
    "medium_task": "先用 Plan 规划，再执行",
    "large_project": "外部工具（Notion/Obsidian）规划，Claude Code 执行具体模块",
}
```

**不用 Plan 模式的原因**：

1. Plan 模式输出的计划不够结构化
2. 无法在执行过程中调整
3. 和现有项目管理系统不兼容
4. 团队协作场景下不够透明

---

## 附录：面试回答模板

### 项目介绍模板

```
项目背景：
- 是什么类型的项目
- 解决了什么问题

技术架构：
- 整体架构图（口述）
- 核心技术选型
- 关键模块

我的职责：
- 负责哪些模块
- 解决了什么难题
- 取得了什么成果

技术亮点：
- 1. xxx
- 2. xxx
- 3. xxx
```

### 项目难点回答模板

```
项目中最难的点是 xxx，主要体现在：

1. 问题分析
   - 遇到了什么技术挑战
   - 尝试过哪些方案

2. 解决方案
   - 最终采用的方案
   - 为什么这个方案更好

3. 效果
   - 解决了什么问题
   - 性能/效果提升多少
```
