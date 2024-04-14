+++
title = '嵌入'
date = 2024-04-06T23:22:37+08:00
draft = false
weight = 6
categories = ['AI', 'OpenAI', 'Embeddings']
tags = ['AI', 'OpenAI', 'Embeddings']
description = '学习如何将文本转换为数字，解锁搜索等用例。新的嵌入模型。'
keywords = ['嵌入', '搜索', '聚类', '推荐', '文本转换', '零样本分类', '用户嵌入', '产品嵌入', '聚类', 'FAQ']
+++

学习如何将文本转换为数字，解锁搜索等用例。

新的嵌入模型

text-embedding-3-small 和 text-embedding-3-large，我们最新且性能最佳的嵌入模型现已推出，具有更低的成本、更高的多语言性能以及控制总体大小的新参数。

## 什么是嵌入？

OpenAI 的文本嵌入衡量文本字符串的相关性。嵌入通常用于：

- 搜索（根据查询字符串的相关性对结果进行排名）
- 聚类（根据相似性对文本字符串进行分组）
- 推荐（推荐具有相关文本字符串的项目）
- 异常检测（识别相关性较小的异常值）
- 多样性测量（分析相似性分布）
- 分类（根据它们最相似的标签对文本字符串进行分类）

嵌入是一个浮点数向量（列表）。两个向量之间的距离衡量它们的相关性。小距离表示高相关性，大距离表示低相关性。

访问我们的定价页面了解有关嵌入定价的信息。请求根据输入中的令牌数量计费。

## 如何获取嵌入向量

要获取一个嵌入向量，将您的文本字符串发送到嵌入API端点，并附上嵌入模型名称（例如text-embedding-3-small）。响应将包含一个嵌入向量（一组浮点数），您可以提取并保存在向量数据库中，以供许多不同的用例使用：

```python
from openai import OpenAI
client = OpenAI()

response = client.embeddings.create(
    input="Your text string goes here",
    model="text-embedding-3-small"
)

print(response.data[0].embedding)
```

响应将包含嵌入向量以及一些额外的元数据。

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [
        -0.006929283495992422,
        -0.005336422007530928,
        ... (omitted for spacing)
        -4.547132266452536e-05,
        -0.024047505110502243
      ],
    }
  ],
  "model": "text-embedding-3-small",
  "usage": {
    "prompt_tokens": 5,
    "total_tokens": 5
  }
}
```

默认情况下，对于text-embedding-3-small，嵌入向量的长度为1536，对于text-embedding-3-large，则为3072。您可以通过传递维度参数来减少嵌入的维度，而不会丢失其表示概念的属性。在嵌入用例部分，我们会更详细地讨论嵌入维度。

## 嵌入模型

OpenAI提供了两种强大的第三代嵌入模型（在模型ID中标注为-3）。您可以阅读嵌入v3公告博客文章以获取更多详细信息。

使用按输入令牌定价，以下是每美元的文本价格页面示例（假设每页约有800个令牌）：

| 模型                    | 每美元页数（大约） | MTEB评估性能 | 最大输入 |
|-------------------------|-------------------|--------------|----------|
| text-embedding-3-small  | 62,500            | 62.3%        | 8191     |
| text-embedding-3-large  | 9,615             | 64.6%        | 8191     |
| text-embedding-ada-002  | 12,500            | 61.0%        | 8191     |

## 用例

在这里，我们展示一些代表性的用例。我们将使用亚马逊精细食品评论数据集作为以下示例。

### 获取嵌入向量

该数据集包含截至2012年10月，亚马逊用户共留下的568,454条食品评论。我们将使用最近的1,000条评论子集进行说明。这些评论是用英文撰写的，往往是积极的或消极的。每条评论都有一个产品ID、用户ID、评分、评论标题（摘要）和评论正文（文本）。例如：

| 产品ID       | 用户ID         | 评分  | 摘要                | 文本                                                |
|--------------|----------------|-------|----------------------|----------------------------------------------------|
| B001E4KFG0   | A3SGXH7AUHU8GW | 5     | 优质狗粮            | 我买了几罐活力狗粮...                                |
| B00813GRG4   | A1D87F6ZCVE5NK | 1     | 广告不实            | 商品标签上写着巨大的盐味花生...                    |

我们将把评论摘要和评论文本合并成一个单独的文本。模型将对这个合并的文本进行编码，并输出一个单一的向量嵌入。

Get_embeddings_from_dataset.ipynb

```python
from openai import OpenAI
client = OpenAI()

def get_embedding(text, model="text-embedding-3-small"):
   text = text.replace("\n", " ")
   return client.embeddings.create(input = [text], model=model).data[0].embedding

df['ada_embedding'] = df.combined.apply(lambda x: get_embedding(x, model='text-embedding-3-small'))
df.to_csv('output/embedded_1k_reviews.csv', index=False)
```

要从已保存的文件加载数据，您可以运行以下命令：

```python
import pandas as pd

df = pd.read_csv('output/embedded_1k_reviews.csv')
df['ada_embedding'] = df.ada_embedding.apply(eval).apply(np.array)
```

### 减少嵌入维度

使用更大的嵌入，例如将它们存储在向量存储中以进行检索，通常会比使用更小的嵌入成本更高，并消耗更多的计算资源、内存和存储空间。

我们的两个新嵌入模型都是使用一种技术进行训练的，该技术允许开发人员在使用嵌入时权衡性能和成本。具体来说，开发人员可以通过传递维度API参数来缩短嵌入（即从序列的末尾删除一些数字），而不会使嵌入失去其表示概念的属性。例如，在MTEB基准测试中，一个text-embedding-3-large嵌入可以缩短到大小为256，同时仍然优于大小为1536的未缩短的text-embedding-ada-002嵌入。您可以在我们的嵌入v3发布博客文章中阅读更多关于如何改变维度影响性能的信息。

总的来说，在创建嵌入时使用维度参数是建议的方法。在某些情况下，您可能需要在生成嵌入后更改嵌入维度。当您手动更改维度时，您需要确保将嵌入的维度标准化，如下所示。

```python
from openai import OpenAI
import numpy as np

client = OpenAI()

def normalize_l2(x):
    x = np.array(x)
    if x.ndim == 1:
        norm = np.linalg.norm(x)
        if norm == 0:
            return x
        return x / norm
    else:
        norm = np.linalg.norm(x, 2, axis=1, keepdims=True)
        return np.where(norm == 0, x, x / norm)


response = client.embeddings.create(
    model="text-embedding-3-small", input="Testing 123", encoding_format="float"
)

cut_dim = response.data[0].embedding[:256]
norm_dim = normalize_l2(cut_dim)

print(norm_dim)
```

动态更改维度使得使用非常灵活。例如，当使用的向量数据存储仅支持长达1024维的嵌入时，开发人员现在仍然可以使用我们最佳的嵌入模型text-embedding-3-large，并为维度API参数指定一个值为1024，这将把嵌入从3072维缩短下来，以换取较小的向量尺寸来权衡一些准确性。

### 使用基于嵌入的搜索进行问答

Question_answering_using_embeddings.ipynb

在许多常见情况下，模型未经过训练的数据中可能不包含您想要在生成用户查询响应时访问的关键事实和信息。解决这个问题的一种方法是将额外的信息放入模型的上下文窗口中，如下所示。这在许多用例中是有效的，但会导致更高的令牌成本。在这个笔记本中，我们探讨了这种方法与基于嵌入的搜索之间的权衡。

```python
query = f"""Use the below article on the 2022 Winter Olympics to answer the subsequent question. If the answer cannot be found, write "I don't know."

Article:
\"\"\"
{wikipedia_article_on_curling}
\"\"\"

Question: Which athletes won the gold medal in curling at the 2022 Winter Olympics?"""

response = client.chat.completions.create(
    messages=[
        {'role': 'system', 'content': 'You answer questions about the 2022 Winter Olympics.'},
        {'role': 'user', 'content': query},
    ],
    model=GPT_MODEL,
    temperature=0,
)

print(response.choices[0].message.content)
```

### 使用嵌入进行文本搜索

Semantic_text_search_using_embeddings.ipynb

为了检索最相关的文档，我们使用查询的嵌入向量与每个文档的嵌入向量之间的余弦相似度，并返回得分最高的文档。

```python
from openai.embeddings_utils import get_embedding, cosine_similarity

def search_reviews(df, product_description, n=3, pprint=True):
   embedding = get_embedding(product_description, model='text-embedding-3-small')
   df['similarities'] = df.ada_embedding.apply(lambda x: cosine_similarity(x, embedding))
   res = df.sort_values('similarities', ascending=False).head(n)
   return res

res = search_reviews(df, 'delicious beans', n=3)
```

### 使用嵌入进行代码搜索

Code_search.ipynb

代码搜索类似于基于嵌入的文本搜索。我们提供了一种方法，可以从给定存储库中的所有Python文件中提取Python函数。然后，将每个函数都由text-embedding-3-small模型索引。

要执行代码搜索，我们使用相同的模型将查询嵌入到自然语言中。然后，我们计算结果查询嵌入和每个函数嵌入之间的余弦相似度。最高的余弦相似度结果最相关。

```python
from openai.embeddings_utils import get_embedding, cosine_similarity

df['code_embedding'] = df['code'].apply(lambda x: get_embedding(x, model='text-embedding-3-small'))

def search_functions(df, code_query, n=3, pprint=True, n_lines=7):
   embedding = get_embedding(code_query, model='text-embedding-3-small')
   df['similarities'] = df.code_embedding.apply(lambda x: cosine_similarity(x, embedding))

   res = df.sort_values('similarities', ascending=False).head(n)
   return res
res = search_functions(df, 'Completions API tests', n=3)
```

### 使用嵌入进行推荐

Recommendation_using_embeddings.ipynb

因为嵌入向量之间的较短距离表示更大的相似性，所以嵌入可以用于推荐。

在下面，我们展示了一个基本的推荐系统。它接受一个字符串列表和一个“源”字符串，计算它们的嵌入，然后返回一个字符串的排名，从最相似到最不相似。作为一个具体的例子，下面的链接笔记本将这个函数的一个版本应用到AG新闻数据集（缩减到2,000个新闻文章描述）上，以返回与任何给定源文章最相似的前5篇文章。

```python
def recommendations_from_strings(
   strings: List[str],
   index_of_source_string: int,
   model="text-embedding-3-small",
) -> List[int]:
   """Return nearest neighbors of a given string."""

   # get embeddings for all strings
   embeddings = [embedding_from_string(string, model=model) for string in strings]

   # get the embedding of the source string
   query_embedding = embeddings[index_of_source_string]

   # get distances between the source embedding and other embeddings (function from embeddings_utils.py)
   distances = distances_from_embeddings(query_embedding, embeddings, distance_metric="cosine")

   # get indices of nearest neighbors (function from embeddings_utils.py)
   indices_of_nearest_neighbors = indices_of_nearest_neighbors_from_distances(distances)
   return indices_of_nearest_neighbors
```

### 二维数据可视化

Visualizing_embeddings_in_2D.ipynb

嵌入的大小随着底层模型的复杂性而变化。为了可视化这个高维数据，我们使用t-SNE算法将数据转换成两个维度。

我们根据评论者给出的星级评分对个别评论进行着色：

- 1星：红色
- 2星：深橙色
- 3星：金色
- 4星：青绿色
- 5星：深绿色

![embeddings-tsne](https://cdn.openai.com/API/docs/images/embeddings-tsne.webp)

这个可视化似乎产生了大致3个聚类，其中一个聚类主要是负面评价。

```python
import pandas as pd
from sklearn.manifold import TSNE
import matplotlib.pyplot as plt
import matplotlib

df = pd.read_csv('output/embedded_1k_reviews.csv')
matrix = df.ada_embedding.apply(eval).to_list()

# Create a t-SNE model and transform the data
tsne = TSNE(n_components=2, perplexity=15, random_state=42, init='random', learning_rate=200)
vis_dims = tsne.fit_transform(matrix)

colors = ["red", "darkorange", "gold", "turquiose", "darkgreen"]
x = [x for x,y in vis_dims]
y = [y for x,y in vis_dims]
color_indices = df.Score.values - 1

colormap = matplotlib.colors.ListedColormap(colors)
plt.scatter(x, y, c=color_indices, cmap=colormap, alpha=0.3)
plt.title("Amazon ratings visualized in language using t-SNE")
```

### 将嵌入作为机器学习算法中的文本特征编码器

Regression_using_embeddings.ipynb

嵌入可以作为机器学习模型中的通用自由文本特征编码器。如果一些相关的输入是自由文本，那么引入嵌入将提高任何机器学习模型的性能。嵌入还可以用作机器学习模型中的分类特征编码器。如果分类变量的名称有意义且数量众多，比如职位名称，这将增加最多的价值。相似度嵌入通常比搜索嵌入在这个任务中表现得更好。

我们观察到，通常情况下，嵌入表示非常丰富且信息密集。例如，使用SVD或PCA将输入的维度降低10％，通常会导致特定任务的下游性能变差。

这段代码将数据分成训练集和测试集，这两个用例将分别用于回归和分类。

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    list(df.ada_embedding.values),
    df.Score,
    test_size = 0.2,
    random_state=42
)
```

#### 使用嵌入特征进行回归分析

嵌入提供了一种优雅的方式来预测数值。在这个示例中，我们基于评论的文本来预测评论者的星级评分。由于嵌入中包含的语义信息很丰富，即使只有很少的评论，预测也是相当不错的。

我们假设得分是1到5之间的连续变量，并允许算法预测任何浮点值。机器学习算法将预测值与真实得分的距离最小化，并实现了平均绝对误差为0.39，这意味着平均预测偏差不到半颗星。

```python
from sklearn.ensemble import RandomForestRegressor

rfr = RandomForestRegressor(n_estimators=100)
rfr.fit(X_train, y_train)
preds = rfr.predict(X_test)
```

### 使用嵌入特征进行分类分析

Classification_using_embeddings.ipynb

这一次，我们不让算法预测在1到5之间的任何值，而是尝试将评论的确切星级分类到5个桶中，范围从1星到5星。

经过训练后，模型学会了更好地预测1星和5星评论，而不是更微妙的评论（2-4星），这可能是由于更极端的情感表达所致。

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, accuracy_score

clf = RandomForestClassifier(n_estimators=100)
clf.fit(X_train, y_train)
preds = clf.predict(X_test)
```

### 零样本分类

Zero-shot_classification_with_embeddings.ipynb

我们可以使用嵌入进行零样本分类，而无需任何标记的训练数据。对于每个类别，我们将类名或类别的简短描述进行嵌入。为了以零样本方式对一些新文本进行分类，我们将其嵌入与所有类别嵌入进行比较，并预测与最高相似度的类别。

```python
from openai.embeddings_utils import cosine_similarity, get_embedding

df= df[df.Score!=3]
df['sentiment'] = df.Score.replace({1:'negative', 2:'negative', 4:'positive', 5:'positive'})

labels = ['negative', 'positive']
label_embeddings = [get_embedding(label, model=model) for label in labels]

def label_score(review_embedding, label_embeddings):
   return cosine_similarity(review_embedding, label_embeddings[1]) - cosine_similarity(review_embedding, label_embeddings[0])

prediction = 'positive' if label_score('Sample Review', label_embeddings) > 0 else 'negative'
```

### 获取用户和产品嵌入以进行冷启动推荐

User_and_product_embeddings.ipynb

我们可以通过对用户的所有评论进行平均来获得用户嵌入。类似地，我们可以通过对关于该产品的所有评论进行平均来获得产品嵌入。为了展示这种方法的有用性，我们使用了50,000条评论的子集，以覆盖更多用户和产品的评论。

我们在一个单独的测试集上评估了这些嵌入的有用性，其中我们将用户和产品嵌入的相似度作为评分的函数进行绘制。有趣的是，基于这种方法，即使在用户收到产品之前，我们也可以比随机更好地预测他们是否会喜欢这个产品。

![embeddings-boxplot](https://cdn.openai.com/API/docs/images/embeddings-boxplot.webp)

```python
user_embeddings = df.groupby('UserId').ada_embedding.apply(np.mean)
prod_embeddings = df.groupby('ProductId').ada_embedding.apply(np.mean)
```

### 聚类

Clustering.ipynb

聚类是理解大量文本数据的一种方式。嵌入对于这个任务是有用的，因为它们提供了每个文本的语义上有意义的向量表示。因此，在无监督的情况下，聚类将揭示数据集中的隐藏分组。

在这个例子中，我们发现了四个不同的聚类：一个聚焦于狗粮，一个聚焦于负面评论，以及两个聚焦于正面评论。

![embeddings-cluster](https://cdn.openai.com/API/docs/images/embeddings-cluster.webp)

```python
import numpy as np
from sklearn.cluster import KMeans

matrix = np.vstack(df.ada_embedding.values)
n_clusters = 4

kmeans = KMeans(n_clusters = n_clusters, init='k-means++', random_state=42)
kmeans.fit(matrix)
df['Cluster'] = kmeans.labels_
```

## 常见问题

### 我如何在对字符串进行嵌入之前确定它包含多少个标记？

在Python中，您可以使用OpenAI的标记器tiktoken将字符串分割为标记。

示例代码：

```python
import tiktoken

def num_tokens_from_string(string: str, encoding_name: str) -> int:
    """Returns the number of tokens in a text string."""
    encoding = tiktoken.get_encoding(encoding_name)
    num_tokens = len(encoding.encode(string))
    return num_tokens

num_tokens_from_string("tiktoken is great!", "cl100k_base")
```

对于像text-embedding-3-small这样的第三代嵌入模型，请使用cl100k_base编码。

有关更多详细信息和示例代码，请参阅OpenAI Cookbook指南中的如何使用tiktoken计数标记。

### 如何快速检索K个最近的嵌入向量？

为了快速搜索许多向量，我们建议使用向量数据库。您可以在我们在GitHub上的Cookbook中找到使用向量数据库和OpenAI API的示例。

### 我应该使用哪种距离函数？

我们推荐使用余弦相似度。距离函数的选择通常并不重要。

OpenAI的嵌入已经归一化为长度1，这意味着：

- 使用点积可以稍微更快地计算余弦相似度
- 余弦相似度和欧几里得距离将得到相同的排名

### 我可以在线分享我的嵌入吗？

是的，客户拥有我们模型的输入和输出，包括嵌入的情况。您有责任确保您输入到我们API的内容不违反任何适用的法律或我们的使用条款。

### V3嵌入模型了解最近的事件吗？

不，text-embedding-3-large和text-embedding-3-small模型缺乏对2021年9月之后发生事件的了解。这通常不会像对文本生成模型那样限制严重，但在某些边缘情况下可能会降低性能。

---

- [官网](https://platform.openai.com/docs/guides/embeddings)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/embeddings/)
    - [微信 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649976821&idx=1&sn=5f6df5599387fc2caf44470feb7d3d3f&chksm=86c7d530b1b05c26828a9da07a59a13701586866b6d444134da8d0834cb6e50df177aded36b0#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/137750246)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/692425478)
