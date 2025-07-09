# 1 概述

本文是本系列（使用RAG技术构建企业级文档问答系统）的第二篇，将介绍检索增强生成（Retrieval Augmented Generation，简称RAG）最基础流程。

所谓检索增强生成，是大语言模型兴起之后发展迅速的一个应用领域，简单说就是，这项技术，可以根据用户输入的问题，从文档（如PDF、Word、PPT、TXT、网页等）中自动检索跟问题相关的文本片段（或称为知识片段、上下文），然后将一段指令、用户输入的问题、文本片段拼装成一个Prompt（也就是大语言模型的输入），让大语言模型生成一个回答。

在ChatGPT最初发布的时候，回答问题主要还是依赖ChatGPT训练时的知识，由此导致了三个显著问题：

- 知识陈旧：也就是新发生的事情，它是没办法回答的
    
- 幻觉：也就是编造与事实不符的回答
    
- 没有办法让ChatGPT基于自己独有的，如个人积累的或者企业内部积累的知识文档回答问题，只能基于已经公开的信息回答
    

根据RAG所检索对象的不同，大致可以分成2类，但底层的技术其实是高度相似的：

- 知识库问答：主要是检索企业内部一系列文档，比如Word、PDF、Wiki、Confluence等，或者企业自建的知识管理平台。很多企业其实积累了非常多的内部文档，传统方式只能使用关键词，或者特定类目检索，效率低下，使用RAG后可以高效快速地直接返回答案，当然这个地方也有它自己的坑，先按下不表，后面有机会再细谈
    
- 联网搜索问答：这个主要是检索整个互联网，最典型代表就是Perplexity，国内的典型产品像秘塔AI搜索、天工AI、360AI搜索等，其实也是检索文档，但会首先借助搜索引擎API，获取一个网页列表，然后再对每个网页执行加载、切分、向量化操作，这个之前已经有一篇文章介绍了，感兴趣的朋友可以访问
    

[使用Ollama和Langchain动手开发AI搜索问答助手](http://mp.weixin.qq.com/s?__biz=MjM5NTQ3NTg4MQ==&mid=2257496693&idx=1&sn=6ce58b0a5c95ae2ceab9f915533c7ada&chksm=a58df2b392fa7ba50e2c6a6edc7b6564e4845a7a99a7a07202a889418edf65c20fcfef917271&scene=21#wechat_redirect)  
上面反复提到了知识库，在RAG的流程中，知识库会经历下面4个步骤处理，如下图所示：

- 加载：可以简单理解成把文档读取成字符串
    
- 切分：按照特定长度，把文档切分成文本片段，做这一步是因为，后面要使用向量模型将切分后的文本片段（其实就是段落或者句子）转换成向量，由于向量模型输入长度限制，所以这一步必须按照特定长度切分
    
- 向量化：这一步会使用一个向量模型，将一个**句子**转换成一个向量，跟word2vec模型其实不是一个东西，word2vec模型是把一个字符或者一个词，转换成一个向量，而在RAG中说的向量模型，是把句子转换成向量，这样后续就可以使用向量计算，来比较句子之间的相似性，所谓RAG中的检索，很大程度是依赖向量，所以这块很重要
    
- 向量存储：这一步一般会使用向量库存储向量化好的文本片段，以及一些元数据信息，如文件名、ID之类的，向量库是类似MySQL、PostgreSQL一样的一个数据库，只不过它专注于存储向量，典型的有Milvus、FAISS、Chroma、Qdrant、Pinecone、Weatiate、PGVector等
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPd8u6YvwndWw6lNcISicicyAVTNUuwZJt3tKQt5yJOWzUNpPMYnicthPOcWGYcEHSPicvlrUrfhh7VQSibw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

(https://python.langchain.com/v0.2/docs/tutorials/rag/)

知识库处理好，保存到向量库之后，当用户提问时，会将用户问题也进行向量化，然后拿用户问题向量，去向量库中，使用余弦相似度（只是原理，后续后再详细展开），检索到最相似的一些句子，然后将用户问题、检索到的相似句子，一同组成一个Prompt，输入大模型，生成答案，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPd8u6YvwndWw6lNcISicicyAVTicJjlxTJoEav1cP24GibjoBicXLed0f5u0zDibQibdEqaNzP08Q42muwmpw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

(https://python.langchain.com/v0.2/docs/tutorials/rag/)

下面将构建这个完整流程。

本文代码已经开源，地址在：https://github.com/Steven-Luo/MasteringRAG/blob/main/01_baseline.ipynb

# 2 环境准备

下面代码中所使用到的数据，可以在代码仓库中找到，其中的“问题-答案”对的构建方法，在上一篇文章中有完整说明：

[使用RAG技术构建企业级文档问答系统之Q[[RAG的QA抽取]]
其余部分主要分3步：

- 安装Python包
    
- 准备Ollama，安装好Ollama之后，使用`ollama pull qwen2:7b-instruct`下载模型
    
- 下载向量模型`BAAI/bge-large-zh-v1.5`，这步可选，也可以在执行代码时自动下载，但需要确保能够访问到HuggingFace
    

上面的模型，都可以在本地运行，建议至少预留8GB的内存。

代码在Google Colab环境下进行了测试，正常情况下，安装Anaconda基本上会包含大部分所用到的包，再安装如下包即可：

pip install -U langchain langchain_community pypdf sentence_transformers chromadb  

所安装包的版本

import langchain, langchain_community, pypdf, sentence_transformers, chromadb  
  
for module in (langchain, langchain_community, pypdf, sentence_transformers, chromadb):  
    print(f"{module.__name__:<30}{module.__version__}")  

langchain                     0.2.10  
langchain_community           0.2.9  
pypdf                         4.3.1  
sentence_transformers         3.0.1  
chromadb                      0.5.4  

import os  
import pandas as pd  
  
from langchain_community.vectorstores import Chroma  
  
# 如果已经下载到本地，可以替换为本地路径  
EMBEDDING_MODEL_PATH = 'BAAI/bge-large-zh-v1.5'  
dt = '20240713'  
version = 'v1'  
  
output_dir = os.path.join('outputs', f'{version}_{dt}')  

加载数据集，包含问题、回答、所使用的文档片段，因此，使用这个数据集，可以对检索、生成效果进行测试

qa_df = pd.read_excel(os.path.join(output_dir, 'question_answer.xlsx'))  

# 3 文档处理

## 3.1 文档加载

此处使用PyPDF这个库进行加载，处理PDF的库还有很多，后面会专门出一篇文章进行介绍。

from langchain_community.document_loaders import PyPDFLoader  
  
loader = PyPDFLoader("data/2024全球经济金融展望报告.pdf")  
documents = loader.load()  

## 3.2 文档切分

在企业内部，一般知识库会比较庞大，每次都重新切分会比较耗时，因此，对切分后的文档片段也可以保存，方便下次再加载

from uuid import uuid4  
import os  
import pickle  
  
from langchain.text_splitter import RecursiveCharacterTextSplitter  
  
def split_docs(documents, filepath, chunk_size=400, chunk_overlap=40, seperators=['\n\n\n', '\n\n'], force_split=False):  
    if os.path.exists(filepath) and not force_split:  
        print('found cache, restoring...')  
        return pickle.load(open(filepath, 'rb'))  
  
    splitter = RecursiveCharacterTextSplitter(  
        chunk_size=chunk_size,  
        chunk_overlap=chunk_overlap,  
        separators=seperators  
    )  
    split_docs = splitter.split_documents(documents)  
    for chunk in split_docs:  
        chunk.metadata['uuid'] = str(uuid4())  
  
    pickle.dump(split_docs, open(filepath, 'wb'))  
  
    return split_docs  
      
splitted_docs = split_docs(documents, os.path.join(output_dir, 'split_docs.pkl'), chunk_size=500, chunk_overlap=50)  

## 3.3 向量化

加载向量模型

from langchain.embeddings import HuggingFaceBgeEmbeddings  
import torch  
  
device = 'cuda' if torch.cuda.is_available() else 'cpu'  
print(f'device: {device}')  
  
embeddings = HuggingFaceBgeEmbeddings(  
    model_name=EMBEDDING_MODEL_PATH,  
    model_kwargs={'device': device},  
    encode_kwargs={'normalize_embeddings': True}  
)  

将文档向量化，并使用Chroma持久化

from tqdm.auto import tqdm  
  
def get_vector_db(docs, store_path, force_rebuild=False):  
    if not os.path.exists(store_path):  
        force_rebuild = True  
  
    if force_rebuild:  
        vector_db = Chroma.from_documents(  
            docs,  
            embedding=embeddings,  
            persist_directory=store_path  
        )  
    else:  
        vector_db = Chroma(  
            persist_directory=store_path,  
            embedding_function=embeddings  
        )  
    return vector_db  

跟上方的文档切分类似，企业知识库通常会比较庞大，如果每次都重新向量化，会非常耗时，因此，可以将向量化后的文档片段持久化

vector_db = get_vector_db(splitted_docs, store_path=os.path.join(output_dir, 'chromadb', 'bge_large_v1.5'))  

# 4 检索

Langchain提供了比较方便的API，使用下方的函数即可完成检索

def retrieve(vector_db, query: str, k=5):  
    return vector_db.similarity_search(query, k=k)  

为了方便后续对文档问答效果进行优化，此处对中间环节——检索，进行评估。

注意，一般这一步评估也是比较麻烦的，因为文档问答，答案来源于文档片段，如果回答错误，不能说明检索一定错误，反过来，如果答案正确，那么在检索环节，只要正确回答的文本“来自”所检索的文档片段，就应该算检索正确，但具体回答是否“来自”文档片段时，有技术上的问题，具体来说，有以下几点：

- 不能直接拿字符串匹配，因为生成的答案经过了大模型的加工，不能保整与检索的文档片段中的文字一字不差
    
- 使用向量模型，将两者转换成向量，计算向量相似度，但这样面临卡阈值的问题，到底阈值多少算是答案参考了知识片段
    
- 使用字符串模糊匹配的方式，也有跟计算向量相似度类似的卡阈值问题
    
- 最终答案可能来源于原本相连的段落，但由于文档切分，将整个段落切分到了两个文档片段，这样虽然可能最后回答正确，但单独拿出每一个片段来，跟答案的相似度可能都不高
    

后面会出一篇专门的文章，专门介绍文档问答的检索、回答的性能。

回到本文，由于在构造“问题-回答”对时，特意记录了所使用的文档片段，这样就可以直接用这个文档片段的UUID计算，避免了上面的问题。

具体到检索的性能，一般使用HitRate进行评估，其中为测试集总数，第条数据检索命中时为1，否则为0。

由于知识片段本身的相似性比较高，因此，只检索一条一般是没法回答问题的。一般会检索Top-K个知识片段。具体到指标计算，就是对于每一条测试数据，检索个知识片段，只要有一个检索命中，那就为1，否则为0。

下面是Top1~Top8的HitRate计算：

test_df = qa_df[(qa_df['dataset'] == 'test') & (qa_df['qa_type'] == 'detailed')]  
  
# 计算Top1~Top8的HitRate  
top_k_arr = list(range(1, 9))  
hit_stat_data = []  
  
for idx, row in tqdm(test_df.iterrows(), total=len(test_df)):  
    question = row['question']  
    true_uuid = row['uuid']  
    chunks = retrieve(vector_db, question, k=max(top_k_arr))  
    retrieved_uuids = [doc.metadata['uuid'] for doc in chunks]  
  
    for k in top_k_arr:  
        hit_stat_data.append({  
            'question': question,  
            'top_k': k,  
            'hit': int(true_uuid in retrieved_uuids[:k])  
        })  
          
hit_stat_df = pd.DataFrame(hit_stat_data)  
hit_stat_df.sample(5)  

||question|top_k|hit|
|---|---|---|---|
|489|美元的走势如何变化？|2|1|
|682|美联储加息对美国房地产市场风险排名产生了什么影响？|3|0|
|344|预计2023年欧元区的经济增速大概是多少?|1|0|
|230|2023年前8个月全球货物贸易量指数的变化情况如何？|7|1|
|444|美联储在2月1日的基点变动了多少?|5|1|

检索HitRate计算

import seaborn as sns  
  
hit_stat_df.groupby('top_k')['hit'].mean().reset_index()  

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPd8u6YvwndWw6lNcISicicyAVTpuzwem36RHiabWk08RR24qul3oh9gtbATQI3asSVbYboY4eVocApicyA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

sns.barplot(x='top_k', y='hit', data=hit_stat_df.groupby('top_k')['hit'].mean().reset_index())  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPd8u6YvwndWw6lNcISicicyAVTnQx9uqggic4a1wFSHezf2aCIL91GWEMjQ5N2sTJptmeu3WmCxFjqf2A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

  

  

大家可以稍微留意一下这个指标，后续会陆续对检索进行优化，大家到时可以直观地观察到检索性能的提升。

# 5 问答

下面就是综合向量库、检索的完整问答流程了。

## 5.1 使用LCEL

这一步演示如何使用Langchain Expression Language，这种方式整个代码会相对简洁，但如果对流程不熟悉，遇到bug不好调试。

### 5.1.1 流式输出

from langchain.llms import Ollama  
from langchain_core.output_parsers import StrOutputParser  
from langchain_core.runnables import RunnablePassthrough  
from langchain_core.prompts import PromptTemplate  
  
def format_docs(docs):  
    return "\n\n".join(doc.page_content for doc in docs)  
  
llm = Ollama(  
    model='qwen2:7b-instruct',  
    base_url="http://localhost:11434"  
)  
  
prompt_tmpl = """  
你是一个金融分析师，擅长根据所获取的信息片段，对问题进行分析和推理。  
你的任务是根据所获取的信息片段（<<<<context>>><<<</context>>>之间的内容）回答问题。  
回答保持简洁，不必重复问题，不要要添加描述性解释和与答案无关的任何内容。  
已知信息：  
<<<<context>>>  
{context}  
<<<</context>>>  
  
问题：{question}  
请回答：  
"""  
prompt = PromptTemplate.from_template(prompt_tmpl)  
retriever = vector_db.as_retriever(search_kwargs={'k': 4})  
  
rag_chain = (  
    {"context": retriever | format_docs, "question": RunnablePassthrough()}  
    | prompt  
    | llm  
    | StrOutputParser()  
)  
  
for chunk in rag_chain.stream("2023年10月美国ISM制造业PMI指数较上月有何变化？"):  
    print(chunk, end="", flush=True)  

输出

2023年10月美国ISM制造业PMI指数较上个月大幅下降2.3个百分点。  

### 5.1.2 非流式输出

print(rag_chain.invoke('2023年10月美国ISM制造业PMI指数较上月有何变化？'))  

输出

2023年10月美国ISM制造业PMI指数较上个月大幅下降2.3个百分点。  

## 5.2 流程拆解

下面对整个过程，拆解成常规的Python代码：

def rag(query, n_chunks=5):  
    prompt_tmpl = """  
你是一个金融分析师，擅长根据所获取的信息片段，对问题进行分析和推理。  
你的任务是根据所获取的信息片段（<<<<context>>><<<</context>>>之间的内容）回答问题。  
回答保持简洁，不必重复问题，不要要添加描述性解释和与答案无关的任何内容。  
已知信息：  
<<<<context>>>  
  
<<<</context>>>  
  
问题：  
请回答：  
""".strip()  
  
    chunks = retrieve(vector_db, question, k=n_chunks)  
    prompt = prompt_tmpl.replace('', '\n\n'.join([doc.page_content for doc in chunks])).replace('', query)  
  
    return llm(prompt), [doc.page_content for doc in chunks]  

prediction_df = qa_df[qa_df['dataset'] == 'test'][['uuid', 'question', 'qa_type', 'answer']]  
  
answer_dict = {}  
for idx, row in tqdm(prediction_df.iterrows(), total=len(prediction_df)):  
    uuid = row['uuid']  
    question = row['question']  
    answer, context = rag(question, n_chunks=4)  
    answer_dict[question] = {  
        'uuid': uuid,  
        'ref_answer': row['answer'],  
        'gen_answer': answer,  
        'context': context  
    }  
      
prediction_df.loc[:, 'gen_answer'] = prediction_df['question'].apply(lambda q: answer_dict[q]['gen_answer'])  
prediction_df.loc[:, 'context'] = prediction_df['question'].apply(lambda q: answer_dict[q]['context'])  
prediction_df.sample(5)  

![图片](https://mmbiz.qpic.cn/mmbiz_png/pJvU3tDvPd8u6YvwndWw6lNcISicicyAVTU2oaneGrW6xde0JqqtDpSBciaCg7ByHwk7JiaTlhTPcJkLbNLmIicR7zA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)