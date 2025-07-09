
# 1 概述

大语言模型虽然已经有了很多的背景知识，但针对模型训练之后新产生的内容，或者领域内的知识进行提问，大模型本身通常无法准确给出回应，一个常用的解决方法是，借助检索增强生成（RAG），将能够用于回答问题的相关上下文给到大模型，利用大模型强大的理解和生成能力，来缓解这个问题。

本文主要介绍如何借助搜索引擎，获取比较新的内容，并对这部分内容的问题进行回答。首先会简单介绍原理，然后是环境准备，代码介绍，最后会通过Chainlit，构造一个完整的可视化Demo。

本文所介绍方法，不需要使用付费大语言模型API，整个流程可以在一台笔记本电脑（8GB以上内存）上运行。效果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPdic9zibgCnXfibHrJAEVCVibALqUKd679NpehgDbPHmc4CbgvPGHOeicypHickVvrnK4JDXEuGexU0rwsGQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

# 2 基本原理

本文所介绍内容，总体依然是RAG，下面是总体处理流程。

AI搜索问答并非在用户输入问题时才去互联网爬取数据，这样会来不及处理，通常都是借助搜索引擎。从搜索引擎获取到相关文档后，后续的所有流程，就跟一般的RAG完全一致了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPdic9zibgCnXfibHrJAEVCVibALqIjSsu0swyJ5ZdoUBb9OchviawELcjiaF88h190IaiaR512loFOVQeA2WA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

image.png

  

# 3 环境准备

## 3.1 操作系统

本文使用的所有组件、库，在Windows、Linux、macOS上都可以安装，理论上在这三个系统上都可以正常运行，但没有对所有系统做兼容性测试，下文以macOS 14.4 Sonoma系统，ARM系列芯片笔记本电脑的环境为例进行介绍。

## 3.2 Python环境准备

推荐使用Anaconda或者Miniconda准备Python环境，具体兼容的Python版本没有做完整测试，本文所使用的是Python 3.11.4。Python安装完成后，安装如下依赖包：

```
pip install -r requirements.txt
```

## 3.3 Ollama安装及模型下载

Ollama是一个能够在本地运行大语言模型的应用，可以直接在命令行中进行问答交互、或者使用相应的API（本文要用到的方式），以及使用第三方GUI工具，如Lobechat等。

从Ollama官网下载并安装对应操作系统的Ollama，Ollama详细的安装配置，请参考Ollama官网。

### 3.3.1 模型下载

Ollama安装好之后，在命令行中，执行如下两条命令，下载相应的大语言模型和向量模型：

```
ollama pull qwen:7bollama pull znbang/bge:large-zh-v1.5-q8_0
```

在Ollama官方的Models页面，提供了非常多支持的模型，如果对相关模型比较熟，可以根据机器的配置选择更大或更小的模型。

下载完成后，执行如下命令，进行二次确认，确保下图中框选的部分在列表中：

```
ollama list
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPdic9zibgCnXfibHrJAEVCVibALq8QIibYCmIeG4gcElC0e7ickmT0bHtiaeicQ4vAmv1iahksYBkicnicZpklHNg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

ollama_list.png

  

使用如下命令，检查大语言模型是否可以正常工作：

```
ollama run qwen:7b
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPdic9zibgCnXfibHrJAEVCVibALqA89cWFuTtp1NB2DBLKRbTxMry1Urq7I0IICiconpPUeiaPS0sPWU7yuA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

ollama_run.png

  

如果输出如上图所示内容，则说明大语言模型工作正常。输入`/exit`退出问答界面。

### 3.3.2 使用API

#### 3.3.2.1 大语言模型

如果是依照本文，在本机安装Ollama，执行如下Python代码：

```python
from langchain_community.llms.ollama import Ollama  
  
model_name = 'qwen:7b'  
model = Ollama(base_url='http://localhost:11434', model=model_name)  
  
print(model('你是谁'))  
```


如果输出如下内容，则表示API调用正常：

我是阿里云研发的大规模语言模型，我叫通义千问。  

如果Ollama安装在其他机器，替换上述代码中的`base_url`

#### 3.3.2.2 向量模型

类似大语言模型的部分，执行如下Python代码：
```python
from langchain_community.embeddings import OllamaEmbeddings  
  
embedding_model = OllamaEmbeddings(  
    base_url='http://localhost:11434',  
    model='znbang/bge:large-zh-v1.5-q8_0'  
)  
print(embedding_model.embed_query('你是谁'))  

如果输出类似如下内容，则表明向量模型API调用正常：

[0.8701383471488953, 0.926769495010376, ...  
```


## 3.4 搜索引擎API准备

许多搜索引擎都有专门的API，只需要两三行代码即可获取结果，本文使用Bing中文版网页请求地址，借助BeautifulSoup库解析结果的方式，获取Bing搜索结果。

执行如下代码：
```python 
def search_with_bing(query):  
    import requests  
    from bs4 import BeautifulSoup  
    from urllib.parse import quote   
    url = f'https://cn.bing.com/search?q={quote(query)}'  
    headers = {  
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36'  
    }  
    resp = requests.get(url, headers=headers)  
    soup = BeautifulSoup(resp.text, 'html.parser')  
      
    result_elements = soup.select('#b_results > li')  
    data = []  
  
    for parent in result_elements:  
        if parent.select_one('h2') is None:  
            continue  
        data.append({  
            'title': parent.select_one('h2').text,  
            'abstract': parent.select_one('div.b_caption > p').text.replace('\u2002', ' '),  
            'link': parent.select_one('div.b_tpcn > a').get('href')  
        })  
    return data  
  
search_with_bing('大语言模型')  
```


如果结果类似如下所示，则表明执行成功：

[{'title': '什么是LLM大语言模型？Large Language Model，从量变到质变',  
  'abstract': '网页2023年4月17日 · 大语言模型（英文：Large Language Model，缩写LLM），也称大型语言模型，是一种人工智能模型，旨在理解和生成人类语言。. 它们在大量的文本数据上进行训练，可以执行广泛的任务，包括文本总结、翻译、情感分析等等。. LLM的特点是 规模庞大，包含数十亿的参数 ...',  
  'link': 'https://zhuanlan.zhihu.com/p/622518771'},  
 {'title': '什么是大模型（LLMs）？一文读懂大型语言模型（Large ...',  
  'abstract': '网页2 天之前 · 大模型是指具有大规模参数和复杂计算结构的机器学习模型。这些模型通常由 深度神经网络 构建而成，拥有数十亿甚至数千亿个参数。大模型的设计目的是为了提高模型的表达能力和预测性能，能够处理更加复杂的任务和数据。大模型在各种领域都有广泛的应用，包括自然语言处理、计算机视觉、语音识别和 推荐系统 等。大模型通过训练海量数据 …',  
  'link': 'https://www.aigc.cn/large-models'},  
 {'title': '一文读懂“大语言模型” - 知乎',  
  'abstract': '网页2023年7月17日 · 谷歌的 Gen AI 开发工具介绍. 2、大语言模型介绍. 2.1 大语言模型的定义. 大语言模型是深度学习的分支. 深度学习是机器学习的分支，大语言模型是深度学习的分支。机器学习是人工智能（AI）的一个子领域，它的核心是让计算机系统能够通过对数据的学习来提高性能。在机器学习中，我们不是直接编程告诉计算机如何完成任务，而是提供大量 …',  
  'link': 'https://zhuanlan.zhihu.com/p/644183721'},  
...  

# 4 主要流程

## 4.1 使用搜索引擎检索互联网内容

使用上文提到的search_with_bing函数，直接调用即可

```
...  
search_results = search_with_bing('大语言模型')  
...  
```



## 4.2 获取网页全文

此处简洁起见，使用requests库发送GET请求，获取网页全文。

## 4.3 文档解析、切片、向量化及检索

本文使用BeautifulSoup解析上文获取到原始HTML对应的文本html。通常使用这种方式解析的HTML效果比较差，可以使用Jina Reader、Firecrawl等库，获得更高质量的解析结果。
```
...  
  
soup = BeautifulSoup(html, 'html.parser')  
item['body'] = soup.get_text()  
  
...  
```



>下方的代码，会对文本进行切片，进行向量化，并使用query获取检索结果：

```
...  
text_splitter = RecursiveCharacterTextSplitter(  
    ["\n\n\n", "\n\n", "\n"],  
    chunk_size=400,  
    chunk_overlap=50  
)  
documents = [Document(  
    item['body'],  
    metadata={'href': item['href'], 'title': item['title']}  
) for item in search_results.values()]  
split_docs = text_splitter.split_documents(documents)  
vectorstore = Chroma.from_documents(split_docs, embedding_model)  
retriever = vectorstore.as_retriever(search_args={'k': 6})  
retrieved_docs = retriever.get_relevant_documents(query)  
context = '\n\n'.join([doc.page_content for doc in retrieved_docs])  

```

## 4.4 Prompt构造

使用Prompt如下：

`prompt = """请使用下方的上下文（<<<context>>><<</context>>>之间的部分）回答用户问题，如果所提供的上下文不足以回答问题，请回答“我无法回答这个问题”  
`<<<context>>>  
`{context}  
`<<</context>>>  
  
`用户提问：{query}  
`请回答：  
`""".format(query=query, context=context)  

# 5运行

完整代码点击此处获取。

首先完成第3节中的环境准备，然后执行如下命令：

`sh start.sh  

出现类似如下的界面，表明启动成功

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPdic9zibgCnXfibHrJAEVCVibALq4VaBZteOabCk2icMzibRLoFnfC5GsUia7oMxJAZzWbzHWM8OuKNrmcA8Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)
