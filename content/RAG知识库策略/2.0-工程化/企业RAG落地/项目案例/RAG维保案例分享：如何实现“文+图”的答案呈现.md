
RAG一直被看成是大模型在企业应用落地的标准配置，基于企业内部文档的问答，已经解锁出大量使用需求和场景。在这些众多类型的文档中，有相当一部分包含了各类复杂图表，也就是所谓的多模态数据。

本篇以近期实施项目中的一个挖掘机维修场景为例，试图给出一个针对标准化排版PDF 文档（本文演示的固定格式维修手册），使用基于坐标区域截取方法，结合Markdown 语法在回答中显示图片的示例,供大家参考。

**业务背景**

说起挖掘机不禁让人想到了蓝翔搜了下说是截止到 23 年年底，全国范围的液压挖掘机保有量在 200 万台左右。对于一名具体机主而言，在实际干活的过程中，可能碰到的来自发动机、电器、液压、工作装置等大大小小几百个故障，这也让专业的挖机维修需求一直很旺盛。

但对于维修人员而言，显然有几个一直以来的痛点没有被很好解决，比如设备故障类型繁多，单靠个人经验难以覆盖所有问题。再有就是传统老带新的模式下，培训带教周期过长等。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3dSW8Hq8mjUbUYHyPhTMyhQQnKuhzOL9iarj3amx63umBNqgUtU5iaD5Wwm87jD4ia8D6AyqqsB53iarg/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1)

%% 注：维修案例示例内容 %%

后文会介绍一个基于包含 500 多个维修案例合集，开发的挖掘机故障诊断知识库系统，维修人员只需描述故障现象即可获取相关案例，并支持图文结合的答案呈现，直观展示故障部位和维修方法。

**_2_*系统架构

注：本项目扩展自阿里云官方的 local_rag 示例，添加了本地 PDF 图片提取和显示功能，

原项目地址:https://help.aliyun.com/zh/model-studio/use-cases/build-rag-application-based-on-local-retrieval?spm=a2c4g.11186623.help-menu-2400256.d_2_8.5a6771eeJWalDw#a2b0288504ybg 

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3dSW8Hq8mjUbUYHyPhTMyhQyWZKlO6cP8VwKJBKyVdh42ibEbLd9JicjztJwoD3TppiaqNn9rcneabicQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1)

📄 支持多种文档类型（PDF、DOCX、TXT、XLSX、CSV）的上传和处理

🖼️ 智能 PDF 图片提取与显示

🔍 本地知识库构建与管理

🤖 集成阿里云通义千问系列大语言模型

🔄 支持非结构化数据和结构化数据处理

📊 可自定义 RAG 参数（召回数量、相似度阈值等）

#### 3核心技术实现

**_3.1_文档处理与图片提取

系统提供了多种 PDF 图片提取和处理方法，以适应不同场景需求：

***1. 基于坐标的区域截取（推荐方法）***

针对标准化排版的文档（如固定格式的维修手册），使用基于坐标的精确截取：
```
def extract_images_from_maintenance_pdf(pdf_path, label_name):    
image_mapping = {}    
doc = fitz.open(pdf_path)       

# 根据文档格式定义的图片区域坐标    
image_rect = fitz.Rect(400, 160, 750, 320)  # 右侧中间区域  

for page_index, page in enumerate(doc):        
   # 直接从固定区域截取图片        
   pix = page.get_pixmap(matrix=fitz.Matrix(2, 2), clip=image_rect)        
   if is_valid_image(pix):           
      # 保存和映射图片...`
```

**优势：**  

对固定格式文档效果极佳

不受 PDF 内部图像对象表示形式限制

可以捕获矢量图形和复合元素

提高图片提取的准确率和质量

***2. 基于对象标记的提取（备选方法）***

使用 PyMuPDF 的内置功能识别 PDF 中的图像对象：

```
def extract_images_from_pdf(pdf_path, label_name):    
doc = fitz.open(pdf_path)   
for page in doc:       
image_list = page.get_images(full=True)        
for img in image_list:       
# 提取和处理图片...`
```

**局限性：**  

仅能提取 PDF 中显式存储的图像对象

无法提取矢量图形或作为背景的图片

可能会提取装饰性元素或无关图形

****3. 其他优化方案****

**基于内容分析的智能提取**：结合文本标记定位图片

**多模态 LLM 辅助**：使用视觉模型辅助识别复杂文档中的图片

**_3.2_**

****图片处理流程****

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3dSW8Hq8mjUbUYHyPhTMyhQ2mUdlywslGBibIzjmosEu7Dia70h8zB3EdCiby9C8IRdhibPgTqYQfHCuQ/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1)

**_3.3_**

****RAG 技术实现****

****分块策略****

系统针对不同类型的数据采用不同的分块策略：

**非结构化文档**：

```
documents = SimpleDirectoryReader(input_files=enhanced_files).load_data()
```

**结构化数据**：

```
nodes = []
```

****嵌入模型****

默认使用通义千问文档嵌入模型：

```
EMBED_MODEL = DashScopeEmbedding(
```

****检索策略****

采用两阶段检索策略：

**向量相似度初筛**：

```
retriever_engine = index.as_retriever(similarity_top_k=20)
```

**语义重排序**：

```
dashscope_rerank = DashScopeRerank(top_n=chunk_cnt)
```

注：比较初步的检索策略，可根据实际情况进行调整

****4. 图片链接处理****

只保留最相关文本块中的图片链接

移除其他文本块的图片链接

使用 Markdown 语法在回答中显示图片

```
prompt_template = """请参考以下内容，仅使用第一个最相关文本块中的图片链接。
```

**_4_**

**使用指南**

**_4.1_**

****上传数据****

系统支持两种文件上传方式：**临时上传**：直接在 RAG 问答页面上传文件，临时使用

**创建知识库**：在"上传数据"页面中上传文件，并在"创建知识库"页面构建永久知识库

****支持的文件类型****

非结构化数据：PDF、DOCX、TXT

结构化数据：XLSX、CSV

![图片](https://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3dSW8Hq8mjUbUYHyPhTMyhQr7fQcUob6icno4TvVxfOk4T8vGavuOECDeF0qYjHCWCgxksoUZ3LXYA/640?wx_fmt=png&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1)

**_4.2_**

****创建知识库****

进入"创建知识库"页面

选择已上传的类目或数据表

设置知识库名称

点击"确认创建知识库"

**_4.3_**

****RAG 问答****

```
flowchart TD
```

**_5_**

**项目结构**

main.py - FastAPI 应用入口和 Gradio 界面定义

chat.py - RAG 问答核心功能和大模型调用

upload_file.py - 文件上传和处理逻辑，包括 PDF 图片提取

create_kb.py - 知识库创建和管理

html_string.py - Web 界面 HTML 模板

File/ - 存放上传的文件

VectorStore/ - 存放向量数据库

static/images/ - 存放提取的图片

images/ - UI 头像图片

注：源码已发布至知识星球，按需加入。

p.s 加入星球后请微信后台私信我加入微信交流群

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/3OZeSOuRw3fjsaiaxuUro0HQYoUygc8wx3GtAXZyUp5icnIDbeIUwFJFQc1Ya94NekfBSLXFiaf97dqdomRhXjbmw/640?wx_fmt=jpeg&from=appmsg&tp=wxpic&wxfrom=5&wx_lazy=1)

**_6_**

**自定义与扩展**

修改嵌入模型

可以使用本地嵌入模型替代云端 API。

在create_kb.py和chat.py中取消相关注释并安装额外依赖。

优化提示词模板

修改chat.py中的prompt_template变量以定制提示词模板。

添加新的文档类型支持

扩展upload_file.py中的处理逻辑以支持更多文件类型。

---

![](http://mmbiz.qpic.cn/mmbiz_png/3OZeSOuRw3f0UfiboA2EooV6gxE23nUmwREHMHWnUmrtWcPHRe5CsMMQwa4c65tXKE39I0n2RKNkwgSZrICrnlg/300?wx_fmt=png&wxfrom=19)

**韦东东**

这个公众号给大家分享我日常大模型应用学习和开发实践，其中涉及Lora微调、RAG、Text2SQL、Multi-Agent等方法，25年会着重关注有行业Know-how的垂直产业场景应用开发和咨询，欢迎大家交流。

41篇原创内容

公众号

下一期，结合 RAGFlow 框架展示下如何实现类似图文问答效果。

![](https://mmbiz.qlogo.cn/mmbiz_jpg/6SRMMU65LTibQQicSuicGmeaKmLlq3aJvpkqHORcxNN4Rn4nrEuB1jQRguNA2gibfNzdHUbgpuauyS1CMuWF23Uuug/0?wx_fmt=jpeg)

韦东东

喜欢作者

RAG · 目录

下一篇用自定义脚本，解锁RAGFlow中Word复杂表格的终极图文问答

阅读 4042

​

**留言 12**

写留言

- ![](https://wx.qlogo.cn/mmopen/ajNVdqHZLLA9oWZg41IHzFQh4sR92fyWSadqiaKTpVdo8ibVhfDWVicAjcxEAoa2Sj5pTPYHy44FWnicem3EAXNC7UiaFdJSbu2yoNmfFVMdXDCvpnmJqzHS4crRYRJsRZAzh/64)
    
    小叮当的铃
    
    上海3月26日
    
    赞3
    
    如果用ragflow实现，应该如何改进呢
    
    ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    韦东东
    
    作者3月26日
    
    赞
    
    后续更新
    
    ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    我有一个朋友叫胖墩妮
    
    上海3月28日
    
    赞
    
    回复 **韦东东**：超级期待！
    
    ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    韦东东
    
    作者4月8日
    
    赞
    
    https://mp.weixin.qq.com/s/yGbMk7gnZ3w2kgV27Xuo0A?payreadticket=HLZpasiz3x9U--pamqIsHbgXCN__YdHMeC3jHwzZaTrNDvZkH86H4jGik3DpcGac6aEofXQ
    
    2条回复
    
- ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    老黄
    
    福建4月19日
    
    赞
    
    作者辛苦
    
- ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    Kylin
    
    四川4月8日
    
    赞
    
    是基于阿里的框架改的吗？
    
    ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    韦东东
    
    作者4月8日
    
    赞
    
    是，文章中有附上原始链接，我最新文章讲了如何在ragflow和dify中实现类似效果
    
- ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    朗朗君浩
    
    重庆3月26日
    
    赞
    
    ![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)
    
    ![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)
    
    终于等到了![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)
    
- ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    韦鹏飞
    
    安徽3月26日
    
    赞
    
    ![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)
    
    ![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)
    
    ![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)![[强]](https://res.wx.qq.com/mpres/zh_CN/htmledition/comm_htmledition/images/pic/common/pic_blank.gif)
    
- ![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBjbGlwLXBhdGg9InVybCgjY2xpcDBfNDIyMF8yNjc0KSI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMGg0MHY0MEgweiIvPjxwYXRoIGZpbGw9IiNFREVERUQiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiIGNsaXAtcnVsZT0iZXZlbm9kZCIgZD0iTTExLjUgMjlhMSAxIDAgMCAxLTEtMXYtLjY4NGMwLS42ODYuNDk4LTEuNDg0IDEuMTE0LTEuNzg1bDUuNjYtMi43NjJjLjgyMS0uNCAxLjAxMi0xLjI4OC40Mi0xLjk5bC0uMzYyLS40MjljLS43MzYtLjg3Mi0xLjMzMi0yLjUtMS4zMzItMy42NFYxNWMwLTIuMjEgMS43OTUtNCA0LTQgMi4yMSAwIDQgMS43OTMgNCA0djEuNzFjMCAxLjE0LS42IDIuNzczLTEuMzMyIDMuNjQybC0uMzYxLjQyOGMtLjU5LjY5OS0uNDA2IDEuNTg4LjQxOSAxLjk5bDUuNjYgMi43NjJjLjYxNS4zIDEuMTE0IDEuMDkzIDEuMTE0IDEuNzg0VjI4YTEgMSAwIDAgMS0xIDFoLTE3eiIgZmlsbD0iIzAwMCIgZmlsbC1vcGFjaXR5PSIuOSIgb3BhY2l0eT0iLjIiLz48L2c+PGRlZnM+PGNsaXBQYXRoIGlkPSJjbGlwMF80MjIwXzI2NzQiPjxwYXRoIGZpbGw9IiNmZmYiIGQ9Ik0wIDBoNDB2NDBIMHoiLz48L2NsaXBQYXRoPjwvZGVmcz48L3N2Zz4=)
    
    风川云啸
    
    湖南3月26日
    
    赞
    
    博主讲的挺详细的
    

已无更多数据