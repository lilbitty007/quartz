

这个公众号给大家分享我日常大模型应用学习和开发实践，其中涉及Lora微调、RAG、Text2SQL、Multi-Agent等方法，25年会着重关注有行业Know-how的垂直产业场景应用开发和咨询，欢迎大家交流。

40篇原创内容

公众号

上篇文章给大家介绍了一个使用 Ollama+ChromDB+DeepSeek-R1:7b组合，本地部署 RAG 问答的自行开发方案的示例，10 天时间文章视频全网有 3W+阅读/播放，maybe 蹭到了 DS 的热点。

![ppcrao.in](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1Hwk11ibFficvcic4d180XYenib4XfDFFeWJwlxZMbKrUuPibicjt83vOSZB2Ow/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

[无需联网！DeepSeek-R1+本地化RAG，打造私有智能文档助手](https://mp.weixin.qq.com/s?__biz=MzI1ODIxNjk1OQ==&mid=2649609187&idx=1&sn=e0f42140d7045e12dd8c3b44e288d152&scene=21#wechat_redirect)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1HwtDNHWdWvibOvJESiacFN3X79iaMYSAv4qAWCP27D1oLMmB0ae9jztorrQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

Github 仓库也有了 29 个 Star、10 个 forks，刚顺便解决了一个处理函数封装成 API 端点的 Issue。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1Hw5YjFeBNUASFdefPr7GDuiafh9oEMc8UxKxGeZzmibr8waj39XMVrSInw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

这个项目原是春节期间在老家给一个企业做 RAG 项目咨询的精简版本，使用 Gradio 构建 Web 界面供大家测试使用。

本是希望大家在这个基础上根据个人或者企业需求进行二次开发，但是在小红书、微信收到一些后台私信里，在集中咨询关于自行开发和现有主流 RAG 框架的区别。所以，有了这篇。
**自主开发的优缺点**

首先，毋庸置疑的一点是，针对企业级 RAG 部署方案的选择，需结合开发成本、功能需求与运维复杂度综合评估。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1HwGwX8uBdawKJ5RJTD9D6z4ia7opOA1yBGicMVsZzicIR3XGNR6icicuiciaWMw/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

自主开发的明显优势是，可以完全自主掌控检索流程（比如可以定制冲突检测算法与多源排序逻辑等），支持动态调整文本分割策略（chunk_size=800, overlap=50）适配不同文档类型，最后就是轻量化运行，最低配置仅需约 2GB 内存即可运行，适配集成显卡环境。

但问题也很明显，首先是企业级功能缺失，缺乏权限管理体系（如 AD/LDAP 集成），无审计日志与操作追溯模块等。另外扩展性限制也有明显局限性，单机部署架构，无法横向扩展处理高并发请求，也没有增量更新机制（每次需全量更新文档向量，仅指当前项目）。


**主流框架对比分析**

那有哪些现成的框架可以参考呢？

基于低成本、易部署、数据安全三个方面特点，并结合开源特性，经过个人初步测试，选择了AnythingLLM、Cherry Studio和RAGFlow这三个框架为大家举例说明，综合对比如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1HwUSVUHyZQaXicBLsSHT9BjzNMiazWV707qWZk1uy7neIic1HtS25qfvNCA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1Hw48Etzbuxbv97dQictGSa9EVGf9wiarhf76JxQtZibTAs6wHy81ibBibP7wQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

1. Cherry Studio - 轻量原型工具

核心优势：桌面端零配置运行，集成 30+开源模型（含 3B-70B 参数级别），支持离线问答；

适用场景：5 人以下小微团队快速验证创意，如独立设计师的素材灵感库、初创公司的竞品分析。

2. AnythingLLM - 全栈私有化方案

核心优势：MIT协议允许商业闭源二次开发，内置企业级权限体系，支持 200+文档格式解析；

适用场景：10-50 人规模企业构建私有知识库，如法律事务所的案例库、制造企业的工艺文档库。

3. RAGFlow - 深度文档引擎

核心优势：专利级文档语义理解（DeepDoc 技术），支持表格/图表内容提取，准确率超 92%；

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1Hw60pARFsuUJpjqyKBcNkGXPYUqcomsk5zG9LjIvmLNdOS4ffLmib6Z5A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

适用场景：金融/科研机构处理复杂格式文档，如上市公司的财报分析、学术论文知识图谱构建。

**_3_**

  

**关键配置维度推荐**

诚然，每个框架各有其特色和局限，本篇以作者比较熟悉的 AnythingLLM 为例，从**大模型配置**、**向量数据库选择**、**Embedder首选项**、**分块策略**等四方面，介绍下配置维度初步推荐。

需要说明的是，以下只做个人经验总结的泛泛讨论，不涉及具体场景或项目案例，如有明确实施需求的盆友可以评论区讨论操作细节，当然也欢迎找我私聊交流。

**_3.1_**

  

****模型选择配置****

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1HwXpyaBg8Jx9790j45vSxKTuMO5BZDBpNPiaiatibrpuNicE4MFMOibz49fuQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

****关于本地部署模型与商用API的选择需权衡第三方可能缓存请求数据的风险，如OpenAI默认保留API数据30天。But, 如果你不是调用境外LLM api，或者你的数据又不是那么敏感，初期测试阶段个人建议还是尽量使用商业API，比如DeepSeek-r1或者V3，亦或者最新的Qwen 2.5 Max。****

****毕竟，在保证基座模型的推理能力水平的前提下，才能更好控制变量法去耐心做下述几个工程化调优。****

当然还有混合部署方案，对于需兼顾性能与安全的场景，核心业务使用本地模型，边缘场景可审慎评估商用API：

```
# 敏感数据处理流程示例
```

  

**_3.2_**

  

****向量数据库选型****

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1Hwqv30FIbAjvt3W6hMqXDAC8gI5JdmDj2tBSibYbm5JicWLm2R84rq2tnQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

除上述三个本地VC外，还有云端部署场景需要考虑，这里以Pinecone和Qdrant为例：

Pinecone：适合需要弹性扩展的企业级应用，支持自动索引优化，但需注意API调用成本；Qdrant：开源方案中HNSW算法性能最优，支持混合检索（关键词+向量）

有盆友在上篇帖子里问了哪种向量数据库比较好，这个问题当然要取决于特定的业务背景。个人经验有限无法完整回答，就贴一个在reddit上找到的图片，大家可以做个参考：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1Hwy4Per8iaFuIKUnEcjd3CQWibMhW0aUPgoxibmyibLLQXqGIYQEWzc09f8A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

**_3.3_**

  

****Embedder 选择策略****

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1HwDXIgicZ7wHkRVe026kNXiaFtDfEsA9CiaFY7Qjaqyx6Wia3cHk4SNG3EMA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

1. 敏感数据场景:

本地模型优先：ollama部署的nomic-embed-text（4.8GB显存需求）或all-MiniLM-L6-v2（CPU运行）；

性能对比：

```
# 嵌入速度测试（千字/秒）
```

2. 非敏感数据场景：

OpenAI API：text-embedding-3-large在MTEB基准测试中准确率91.2%，但需配置API调用审计策略；

混合部署策略：

```
graph LR
```

**_3.4_**

  

****分开策略优化方案****

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1HwPkVNAsdIHA35yUIMaibMTV3icUcdibRlQicIyVCoH5BzRb9k66ALTklEUA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

文本分块大小和重叠大小直接决定了检索器（Retriever）能够提供给生成器（Generator）的上下文质量：

- 块大小：较大的分块可以保留更多上下文信息，但可能导致信息稀释，降低检索精度；较小的分块则可能导致重叠不足，而易造成上下文断裂。
    

- 重叠大小：适度的重叠有助于保持跨块的语义连贯性，但过多重叠会增加冗余，降低检索效率。
    

上表是根据个人近期实践结合网上搜索做的整理，仅供参考。分块大小与业务场景强相关，没有普适最优解。一般而言，分块策略的调整依据是：

复杂文档（如法律条款）：块大小建议 4096，重叠 512。

短文本（如对话记录）：块大小建议 1024，重叠 256。

在此基础上，还应该根据自定义的质量评估指标设计动态调整机制，例如：检索召回率<85% → 增大块重叠（每次+10%）。生成结果偏离度>30% → 减小块大小（每次-25%）。

**_5_**

  

**核心影响要素分级**

根据 Perplexity 检索的相关实证研究显示（我没看），各参数对输出效果的影响权重可量化如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3d9JZZ9hXIGibxojxOYQG1HwDeeGGSkEhq0TC3lQ7vsojAa0qoRwAbyxiaiagZB1sCBUoMe2ajWgFRuw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

> Towards&nbsp;Understanding&nbsp;Retrieval&nbsp;Accuracy&nbsp;and&nbsp;Prompt&nbsp;Quality&nbsp;in&nbsp;RAG&nbsp;Systems
> 
> https://arxiv.org/html/2411.19463

****分块策略（权重 35%）****

块大小直接影响信息完整性，法律文档建议 4096 字符重叠量优化上下文保留，代码类数据最佳重叠率 12.5%;

****嵌入模型适配（权重 30%）****

领域专用词表覆盖率需>85%混合嵌入方案可提升跨模态检索准确率 23%

****重排序机制（权重 25%）****

BGE 重排器使 MRR@5 提升 41%动态阈值过滤减少噪声文档干扰

****提示工程（权重 10%）****

CoT 提示策略在 QA 任务中提升 F1 值 17%结构化模板降低代码生成错误率 32%

**_6_**

  

后续拟更新

1. RAG 系统效果评估与持续优化体系构建
    
2. 主流 RAG 框架多模态能力深度剖析与选型指南
    
3. 自主开发 RAG 系统：增量更新机制设计与实现
    

  

（完）