
这篇试图说清楚：

###RAGFlow 与 MinerU 在复杂表格处理下的局限性、如何使用 Python-docx 等库实现把每一行表格数据都转化为一个独立且富含上下文的“事实”句子，以及如何可靠的提取单元格图片和存储实现。
==关于文章中的方案，在不同类型word和pdf下的适用问题说明==
![671b65df452a1d483bbc6400c37c4873.jpg](https://raw.githubusercontent.com/lilbitty007/obs_img/%E5%88%86%E6%94%AF1/img/20250620221735048.jpg)



以下，enjoy:

**_1_****案例材料说明**

这次演示所用的文档，依然是历史文章中经常使用的工程机械维保材料。鉴于原始文档中并没有标准的复杂表格结构，我手动做了下预处理，其中包含了合并单元格和单元格嵌入图片的用例。



##### 第一页
标准图文对照表。这是一个相对规整的表格，但其关键在于将“相关图片”作为了表格的一列。这代表了产品手册、物料清单、故障图例等场景，即图像本身就是结构化数据的一部分。
![image.png](https://raw.githubusercontent.com/lilbitty007/obs_img/%E5%88%86%E6%94%AF1/img/20250620154437297.png)
##### **第二页**：

多级合并单元格表。这个表格的复杂度高了很多，同时包含了横向合并（如顶部的“案例基础信息”横跨多列）和纵向合并（如左侧的“故障系统分类”纵跨多行）。这种多级表头和行列合并的结构，在各类报告、技术规格书和复杂的流程记录中非常普遍。
![2604f03545e63d8b5824c5831a2e6152.jpg](https://raw.githubusercontent.com/lilbitty007/obs_img/%E5%88%86%E6%94%AF1/img/20250620160549942.jpg)
**_1.2_**

****解析难点****

**单行信息的完整性至关重要**：

在进行向量化切分时，必须将同一行的所有信息作为一个完整的、富含上下文的知识块（Chunk）来处理。如果简单地按单元格或固定长度进行切分，就会彻底破坏这种内在逻辑，导致模型在检索时只能找到零碎、残缺的信息片段。

**上下文补全是必要前提**：

对于第二页中被垂直合并的单元格，必须把上级标题（如“发动机故障”）自动填充到后续的逻辑行中，以确保每一行知识都具备完整的上下文，例如“（发动机故障下的）动臂侧摆油缸自动伸出”。

### **_2三种非预处理的对比测试

在正式开始介绍自定义脚本的预处理方案前，先快速过下三种更简单直接的方式，看下对比测试效果如何，或者说看下标准方案下的局限性如何。

#### _2-1直接上传 Word 到 RAGFlow

作为基线测试，我选择直接把.docx 格式的维修案例源文件直接上传至 RAGFlow 知识库。测试之后发现，RAGFlow 对 Word 文档中的表格结构有着相当不错的解析能力。它能够正确地保留原始的表格样式，并且智能地处理了合并单元格，将主标题（如“故障系统分类”）自动填充到了被合并的单元格中，保证了每一行信息的上下文完整性。在纯文本问答测试中，这种处理方式能够返回准确的答案。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgiclaya4fpJaUhxLib0RbYSXLZ3vAkYC7YPzsRMoGZZ3ZO0PFogicuRkB7Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicDDr3s53K2aicOXmLYzKCIyn0DqJXsjbM5gxOGMZa4icnMlvLWEHrWxKQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

然而，有个明显的短板在于其完全无法处理文档中的图片。上传后，所有与故障案例相关的图片信息都丢失了。当然根本原因在于，RAGFlow v0.19 的版本的图片处理流程目前并不支持从.docx 文件中直接提取图像，目前主要是针对 PDF、PPT 等格式。


### **_2.2把 Word 另存为 PDF 后上传 RAGFlow***_



为了触发 RAGFlow 原生的图片处理能力，我尝试了一个看似直接的变通方法：把Word 文档另存为 PDF 格式后再进行上传。但是结果证明，这种做法是个很不明智的选择。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicBeibaQ4sDwtcGQYB43rK1RIRxDcgaLBCzcfmVztia7VppQibyg2cvB3tg/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

虽然，RAGFlow 确实启动了图片处理模块，但提取出的“图片”内容完全不对。它把整个页面，甚至是部分不相关的图文组合错误地识别为了单一的图片块。与此同时，文本分块也变得极度混乱，原有的表格结构被彻底打碎，失去了任何逻辑关联。

造成这种现象的可能原因是，Word 在“另存为 PDF”的过程中，主要关注的是视觉保真度，而非逻辑结构的传递。生成的 PDF 很可能是一个“非结构化”或“无标签”的 PDF，它虽然看起来和原文一样，但已经丢失了关于哪些是文本、哪些是表格、哪些是图片的底层元信息。RAGFlow 的 PDF 解析器在面对这种只有视觉布局、没有逻辑结构的文档时，无法准确地分割内容边界，从而导致了错误的区块识别和混乱的文本分块。

**_2.3_**

******使用 MinerU 解析 Word 源文件******

作为对比，我也测试了近期较受欢迎的解析工具 MinerU，对同一 Word 源文件的处理效果。结果同样低于预期。MinerU 不仅与 RAGFlow 直传 Word 一样，没法处理任何图片信息，它在文本内容的提取上甚至也出现了明显的缺失和遗漏。原文档中部分表格的行内容未能被完整解析出来，这表明其内置的解析算法同样难以适应这种包含多层合并单元格的复杂表格布局。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicEgQWBGe6pPnTlJicWC08BXyfEL6FXgTUzWTcBFgAfsobfvv5fFbS2IQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

---

通过以上对比可以看出，无论是 RAGFlow 还是 MinerU，其内置的通用解析器在面对包含复杂表格、合并单元格及图文混排的 Word 文档时，都表现出明显的局限性。这些通用工具为了兼容更广泛的文档类型，其解析策略往往是“最大公约数”式的，难以针对特定格式的复杂布局进行深度优化。

当然，这也恰恰凸显了，面向特定场景的自定义预处理脚本的核心价值。**==通过使用 python-docx 等库深入 docx 文档的底层 XML 结构，从而实现精确地解析包括合并单元格在内的复杂表格，以及可靠地提取图片二进制数据并实现自动保存到 MiniO 中，有目的地将非结构化的图文信息转化为对大型语言模型最友好的、富含上下文的“事实语句”。==**

#### **_3_**解决方案框架****

整个系统通过模块化的设计，实现了从原始.docx 文件到 RAGFLow 聊天助手的端到端部署。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicN6aJOk9PiaoOGaZhNX5ARaMwfVMdg4jRTMeMdSfyduaPrIsWX5y8p9Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicR34icVMMRJAkGVhZPYRX1ONx5C202mddicQRhHrAQkcRMu4zhhMTzsfA/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

核心系统分为三大模块：

##### **_3.1_******调度模块 (process_docx_for_ragflow.py)****

作为流程的入口和编排角色，它负责读取.env 文件中的配置，接收用户输入的.docx 文件，并按顺序调用其他功能模块。

#####  **_3.2_******文档处理模块 (docx_processor.py)****

这是数据预处理的核心，其功能特色在于：

**深度解析**：能智能处理包含合并单元格的复杂表格。

**MiniO 存储**：自动提取文档中的图片并上传至 MinIO 对象存储。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgic4kFzPPlVdsJS1kcnP8pJoN6WwMqBmatmPLnjzbibWKanRkIVabY1e2g/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

**格式优化**：将每一行表格数据转化为对 RAG 模型友好的独立句子，并将图片 URL 封装成可在 RAGFlow 中直接渲染的 HTML <img>标签，这是确保可用性和可读性的关键。

##### **_3.3 _******RAGFlow 构建模块 (ragflow_build.py)****

负责与 RAGFlow 平台的所有 API 交互，实现完全自动化部署，包括：

- 创建知识库并上传处理好的文本。
    
- 主动触发并等待文档解析完成。
    
- 创建聊天助手，并为其配置指定 LLM 和提示词。
    

总结来说，原始文档经过处理模块的深度加工，生成包含 HTML 图片标签的结构化文本；随后，这份优化后的文本被构建模块无缝对接到 RAGFlow 平台。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicHYEgibAYHjOyhRrkdR9P0tSW6pAldmOBSClibbYcEO6h51bkGiahj1qDg/640?wx_fmt=jpeg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)
#### **_4_****实现原理解析**

要让 RAG 系统能精准地理解表格内容，首要挑战是必须先将 Word 中那些视觉上不规则、包含大量合并单元格的表格，转化为程序可以理解的、规则的结构化数据。这个自定义脚本通过一个名为 get_table_as_grid 的函数，以一种精巧的算法很好地解决了这个问题。其核心原理可以概括为：在内存中重建一个与视觉布局完全一致的“虚拟网格”，并将原始单元格内容“投影”到这个网格的正确位置上。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicLZa6FicFzsZokgpI5RUPPTrjo0iaRmMNTt5of3icicqhWoG8KwZ0AImmsg/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

%% 注：下面内容偏技术向，不感兴趣的可以跳过，不过还是但当涉猎下为好。
 %%
整个过程主要分为以下几个关键步骤：

**_4.1_**

****第一步：构建标准化的“虚拟网格”****

算法的第一步不是直接读取内容，而是先创建一个空的二维列表（即矩阵），一般称之为“虚拟网格”。这个网格的尺寸是严格按照表格的实际视觉行列数来定义的（例如，一个 5 行 4 列的表格）。这一步至关重要，因为它给后续不规则数据的“归位”提供了一个规整的、标准化的“画布”。（这点其实有些像工业物联网中的数字孪生）

**_4.2_**

****第二步：维护一个“已处理坐标集”****

在遍历原始表格之前，脚本初始化了一个集合（Set）数据结构，用于实时记录虚拟网格中已经被内容填充的坐标 (行号, 列号)。这个集合类似一个“遮罩层”，是整个算法能够正确处理合并单元格的关键。它的作用是确保一旦某个单元格因合并而被填充，就不会再被后续的单元格错误地覆盖。

**_4.3_**

****第三步：遍历并“解码”单元格的合并属性****

接下来，脚本会逐行、逐单元格地遍历原始 Word 表格。对于每一个单元格，不只是简单地读取文本，而是深入其底层的 XML 属性，重点解码两个核心属性：

**gridSpan (水平合并):** 这个属性直接告诉我们当前单元格在水平方向上占据了多少列。

**vMerge (垂直合并)**: 这个属性相对复杂。如果值是'restart'，则表明这是垂直合并区域的起始单元格；如果属性不存在或值为 None，则表明它是一个普通单元格，或者是被上方单元格所覆盖的“后续单元格”。

**_4.4_**

****第四步：智能填充与坐标标记****

在解码了每个单元格的合并信息后，算法执行最核心的填充操作：

**定位**：对于当前遍历到的单元格，算法首先在“虚拟网格”的对应行中，从左到右查找第一个未被“已处理坐标集”标记的位置。这个位置就是当前单元格内容应该被填充的左上角起点。

**投影**：根据上一步解码出的 gridSpan（宽度）和 vMerge（高度，如果为 restart 则计算其跨度），脚本将当前单元格的内容（包括文本和提取出的图片 URL）“投影”或“绘制”到虚拟网格中对应大小的矩形区域内。

**标记**：完成投影后，脚本立即将这个矩形区域内**所有的坐标**都添加到“已处理坐标集”中。

通过这个“定位 → 投影 → 标记”的循环，即使原始表格的结构再复杂，脚本也能确保每个单元格的内容都被不多不少、不重不漏地放置到虚拟网格的正确位置。最终，get_table_as_grid 函数返回的，就是一个与 Word 文档视觉效果完全一致、数据完整的二维矩阵，为后续的“知识语句化”处理提供了可靠的数据基础。

#### **_5**最终实现效果**

从知识库后台的“数据分块”视图中可以清晰地看到，与之前所有方案的混乱分块不同，经过自定义脚本处理后，知识库中的每一个分块（Chunk）都精准地对应了原始表格中的一个完整的逻辑行。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicZfWQfq4ldDY0qe58pbJTyugbdqLdPW9SO44o227DoL6wBQ98scJmFQ/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

这种分块的好处体现在两个方面：

##### **_5.1_******信息的完整性****

每一行数据，如“发动机冒蓝烟”，其对应的“机型”、“故障原因”、“维修方案”以及“相关图片”等所有信息，都完整地封装在同一个知识块中。这确保了在检索的时候，可以一次性获取关于该故障的全部上下文。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicFAzibAPb6ecwY1icxO2qxyibBRFskjZYlBVTe30IUqsfUc1lic0XtcN6ibA/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

##### **_5.2_******图文的原生绑定****

最关键的一点是，脚本将图片上传至 MinIO 后，直接将返回的 URL 包装成 HTML 的<img>标签，并作为文本内容的一部分嵌入到知识块中。当检索系统命中这段文本时，图片的 URL 也被无缝地继承了过来。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicF719ia9WGt1he9fb8ey468Dziay0PhMLdQ9Qf789CLEf7w1xFdjMDuMg/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

需要特别说明的是，这里的图片显示方案依然是沿用了直接的 http url 的直接渲染方式，不是 RAGFlow V0.19 的这种内生方案。关于历史文章中提到的图片 URL 可能会被 LLM 在回答输出时"自作聪明"的篡改问题，实测只要生成图片名称时保证命名的合理性，这种被修改的概率会降低很多。

##### **_5.3_******整体流程****

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3cvo3HwEFxG49B8mykEjwgicrBticDibU0IBnYP7Q8GkUzjvhRQewpyGw3HKXt59wcUibjzIX58WyO4Vw/640?wx_fmt=png&from=appmsg&watermark=1&tp=wxpic&wxfrom=5&wx_lazy=1)

**用户提问**：用户输入问题“发动机冒蓝烟的原因”。

**精准检索**：RAGFlow 的检索系统在向量数据库中进行搜索，命中了之前构建的那个关于“发动机冒蓝烟”的、包含了完整图文信息的知识块。

**智能生成**：这个知识块被完整地提交给 LLM 作为上下文。根据预设的“工程机械专家”提示词，LLM 从中提炼并总结出关键的故障原因：“1. 喷油器故障 2. 气门间隙异常...”。

**图文并茂**：由于预设的提示词中明确要求“对于知识库信息中包含 url 链接...请你务必也把链接信息不要做任何修改的显示在回答中”，LLM 在生成文本答案的同时，也忠实地将知识块中携带的那个<img>标签一并放入了最终的回答里。

**前端渲染**：RAGFlow 的前端界面在收到包含<img>标签的回答后，自动将其渲染为可见的图片，从而实现了图文并茂的最终效果。_**