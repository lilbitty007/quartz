---
标题: "(2) johann.GPT 在 X 上的帖子: \"Agentic RAG + MCP 完整流程图解\" / X-Thread by @ProgramerJohann"
链接: "https://x.com/ProgramerJohann/status/1932246723214352460"
作者: "[[@ProgramerJohann]]"
创建时间: "2025-06-15T15:38:30+08:00"
摘要: "该推文通过流程图解释了 Agentic RAG（检索增强生成）与 MCP（多通道处理器）结合的完整流程，包括查询分析、Agent检索、数据重排和答案生成等步骤，旨在提高数据检索和问题解答的效率和准确性。"
tags:
  - "clippings"
  - "AI"
  - "Agentic RAG"
  - "MCP"
  - "LLM Agent"
  - "数据检索"
---
**johann.GPT** @ProgramerJohann [2025-06-10](https://x.com/ProgramerJohann/status/1932246723214352460)

Agentic RAG + MCP 完整流程图解：

核心流程：

1️⃣ 查询分析：LLM Agent分析用户查询，可重写查询，判断是否需要额外数据源

2️⃣ Agent检索：触发检索时，MCP发挥关键作用 - 各数据域管理独立的MCP服务器，提供标准化数据访问

3️⃣ 数据重排：强大模型对检索数据进行整合和重排，精准筛选

4️⃣ 答案生成：基于整合数据生成回答

5️⃣ 分析答案：分析答案质量，不满足则重写查询重新生成

![Image](https://pbs.twimg.com/media/GtC4BemagAA8h3Z?format=jpg&name=large)

---

