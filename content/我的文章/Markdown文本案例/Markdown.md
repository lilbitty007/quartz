
能够用简洁的语法生成标题、加粗、列表等排版样式，也可以快速键入代码块、数学公式等功能块。无需手动调整样式，提升书写和排版效率。

Heading level 1  
===============

# Heading level 1 

## Heading level 2

==
##### Heading level 5

###### Heading level 6


## Markdown 标题语法
要创建标题，请在单词或短语前面添加井号 (`#`) 。`#` 的数量代表了标题的级别。例如，添加三个 `#` 表示创建一个三级标题 (`<h3>`) (例如：`### My Header`)。

| Markdown语法              | HTML                       | 预览效果                  |
| ----------------------- | -------------------------- | --------------------- |
| `# Heading level 1`     | `<h1>Heading level 1</h1>` | # Heading level 1     |
| `## Heading level 2`    | `<h2>Heading level 2</h2>` | ## Heading level 2    |
| `### Heading level 3`   | `<h3>Heading level 3</h3>` | ### Heading level 3   |
| `#### Heading level 4`  | `<h4>Heading level 4</h4>` | #### Heading level 4  |
| `##### Heading level 5` | `<h5>Heading level 5</h5>` | ##### Heading level 5 |
|                         |                            |                       |

---

---



### Markdown 标题语法可选语法

| Markdown语法                          | HTML                       | 预览效果 |
| ----------------------------------- | -------------------------- | ---- |
| `Heading level 1   ===============` | `<h1>Heading level 1</h1>` |      |
| `Heading level 2   ---------------` | `<h2>Heading level 2</h2>` |      |


This is the first line.  

And this is the second line.

## Markdown 段落语法

要创建段落  ，请使用空白行将一行或多行文本进行分隔。

|Markdown语法|HTML|预览效果|
|---|---|---|
|`I really like using Markdown.      I think I'll use it to format all of my documents from now on.`|`<p>I really like using Markdown.</p>      <p>I think I'll use it to format all of my documents from now on.</p>`|I really like using Markdown.<br><br>I think I'll use it to format all of my documents from now on.|
#### 段落（Paragraph）用法的最佳实

不要用空格（spaces）或制表符（ tabs）缩进段落。

| ✅  Do this                                                                                            | ❌  Don't do this                                                                                             |
| ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `Don't put tabs or spaces in front of your paragraphs.      Keep lines left-aligned like this.      ` | `This can result in unexpected formatting problems.        Don't add tabs or spaces in front of paragraphs.` |




<br>This is the first line.
   And this is the second line.<br>
   
## Markdown 换行语法

在一行的末尾添加两个或多个空格，然后按回车键,即可创建一个换行(`<br>`)。

| Markdown语法                                                 | HTML                                                                | 预览效果                                                      |
| ---------------------------------------------------------- | ------------------------------------------------------------------- | --------------------------------------------------------- |
| `This is the first line.     And this is the second line.` | `<p>This is the first line.<br>   And this is the second line.</p>` | This is the first line.  <br>And this is the second line. |

### 换行（Line Break）用法的最佳实践

几乎每个 Markdown 应用程序都支持两个或多个空格进行换行，称为 `结尾空格（trailing whitespace)` 的方式，但这是有争议的，因为很难在编辑器中直接看到空格，并且很多人在每个句子后面都会有意或无意地添加两个空格。由于这个原因，你可能要使用除结尾空格以外的其它方式来换行。幸运的是，几乎每个 Markdown 应用程序都支持另一种换行方式：HTML 的 `<br>` 标签。

为了兼容性，请在行尾添加“结尾空格”或 HTML 的 `<br>` 标签来实现换行。

还有两种其他方式我并不推荐使用。CommonMark 和其它几种轻量级标记语言支持在行尾添加反斜杠 (`\`) 的方式实现换行，但是并非所有 Markdown 应用程序都支持此种方式，因此从兼容性的角度来看，不推荐使用。并且至少有两种轻量级标记语言支持无须在行尾添加任何内容，只须键入回车键（`return`）即可实现换行。

|✅  Do this|❌  Don't do this|
|---|---|
|`First line with two spaces after.     And the next line.      First line with the HTML tag after.<br>   And the next line.      `|`First line with a backslash after.\   And the next line.      First line with nothing after.   And the next line.   `|
First line with the HTML tag after.<br>  
And the next line.










I just love **bold text**.

I just love __bold text__.

Love**is**bold  

Italicized text is the *cat's meow

Italicized text is the _cat's meow_.


## Markdown 强调语法
要加粗文本，请在单词或短语的前后各添加两个星号（asterisks）或下划线（underscores）。如需加粗一个单词或短语的中间部分用以表示强调的话，请在要加粗部分的两侧各添加两个星号（asterisks）。

### 粗体（Blod）

| Markdown语法                   | HTML                                      | 预览效果                       |
| ---------------------------- | ----------------------------------------- | -------------------------- |
| `I just love **bold text**.` | `I just love <strong>bold text</strong>.` | I just love **bold text**. |
| `I just love __bold text__.` | `I just love <strong>bold text</strong>.` | I just love **bold text**. |
| `Love**is**bold`             | `Love<strong>is</strong>bold`             | Love**is**bold             |

### 粗体（Bold）用法最佳实践

Markdown 应用程序在如何处理单词或短语中间的下划线上并不一致。为兼容考虑，在单词或短语中间部分加粗的话，请使用星号（asterisks）。

|✅  Do this|❌  Don't do this|
|---|---|
|`Love**is**bold`|`Love__is__bold`|

### 斜体（Italic）

要用斜体显示文本，请在单词或短语前后添加一个星号（asterisk）或下划线（underscore）。要斜体突出单词的中间部分，请在字母前后各添加一个星号，中间不要带空格。

| Markdown语法                             | HTML                                          | 预览效果                                 |
| -------------------------------------- | --------------------------------------------- | ------------------------------------ |
| `Italicized text is the *cat's meow*.` | `Italicized text is the <em>cat's meow</em>.` | Italicized text is the _cat’s meow_. |
| `Italicized text is the _cat's meow_.` | `Italicized text is the <em>cat's meow</em>.` | Italicized text is the _cat’s meow_. |
| `A*cat*meow`                           | `A<em>cat</em>meow`                           | A_cat_meow                           |

### 斜体（Italic）用法的最佳实践

要同时用粗体和斜体突出显示文本，请在单词或短语的前后各添加三个星号或下划线。要加粗并用斜体显示单词或短语的中间部分，请在要突出显示的部分前后各添加三个星号，中间不要带空格。

|✅  Do this|❌  Don't do this|
|---|---|
|`A*cat*meow`|`A_cat_meow`|

### 粗体





（Bold）和斜体（Italic）

要同时用粗体和斜体突出显示文本，请在单词或短语的前后各添加三个星号或下划线。要加粗并用斜体显示单词或短语的中间部分，请在要突出显示的部分前后各添加三个星号，中间不要带空格。

|Markdown语法|HTML|预览效果|
|---|---|---|
|`This text is ***really important***.`|`This text is <strong><em>really important</em></strong>.`|This text is **_really important_**.|
|`This text is ___really important___.`|`This text is <strong><em>really important</em></strong>.`|This text is **_really important_**.|
|`This text is __*really important*__.`|`This text is <strong><em>really important</em></strong>.`|This text is **_really important_**.|
|`This text is **_really important_**.`|`This text is <strong><em>really important</em></strong>.`|This text is **_really important_**.|
|`This is really***very***important text.`|`This is really<strong><em>very</em></strong>important text.`|This is really**_very_**important text.|

### 粗体（Bold）和斜体（Italic）用法的最佳实践

Markdown 应用程序在处理单词或短语中间添加的下划线上并不一致。为了实现兼容性，请使用星号将单词或短语的中间部分加粗并以斜体显示，以示重要。

|✅  Do this|❌  Don't do this|
|---|---|
|`This is really***very***important text.`|`This is really___very___important text.`|




> Dorothy followed her through many of the beautiful rooms in her castle.

>asdhasdhas
>aasasasasasa

>dasjdhasdas
>
>>>asdasdassd 


> #### ajsda
> 
>- wode 
> - wo 


## 引用语法

要创建块引用，请在段落前添加一个 `>` 符号。

```
> Dorothy followed her through many of the beautiful rooms in her castle.
```

渲染效果如下所示：

> Dorothy followed her through many of the beautiful rooms in her castle.

### 多个段落的块引用

块引用可以包含多个段落。为段落之间的空白行添加一个 `>` 符号。

```
> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

渲染效果如下：

> Dorothy followed her through many of the beautiful rooms in her castle.
> 
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

### [#](https://markdown.com.cn/basic-syntax/blockquotes.html#%E5%B5%8C%E5%A5%97%E5%9D%97%E5%BC%95%E7%94%A8)嵌套块引用

块引用可以嵌套。在要嵌套的段落前添加一个 `>>` 符号。

```
> Dorothy followed her through many of the beautiful rooms in her castle.
>
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

渲染效果如下：

> Dorothy followed her through many of the beautiful rooms in her castle.
> 
> > The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

### [#](https://markdown.com.cn/basic-syntax/blockquotes.html#%E5%B8%A6%E6%9C%89%E5%85%B6%E5%AE%83%E5%85%83%E7%B4%A0%E7%9A%84%E5%9D%97%E5%BC%95%E7%94%A8)带有其它元素的块引用

块引用可以包含其他 Markdown 格式的元素。并非所有元素都可以使用，你需要进行实验以查看哪些元素有效。

```
> #### The quarterly results look great!
>
> - Revenue was off the chart.
> - Profits were higher than ever.
>
>  *Everything* is going according to **plan**.
```

渲染效果如下：

> **The quarterly results look great**
> 
> - Revenue was off the chart.
> - Profits were higher than ever.
> 
> _Everything_ is going according to **plan**.  







## 列表语法

可以将多个条目组织成有序或无序列表。
### 有序列表

要创建有序列表，请在每个列表项前添加数字并紧跟一个英文句点。数字不必按数学顺序排列，但是列表应当以数字 1 起始。

| Markdown语法                                                                                                      | HTML                                                                                                                                                                         | 预览效果                                                                                                               |
| --------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `1. First item   2. Second item   3. Third item   4. Fourth item`                                               | `<ol>   <li>First item</li>   <li>Second item</li>   <li>Third item</li>   <li>Fourth item</li>   </ol>`                                                                     | 1. First item<br>2. Second item<br>3. Third item<br>4. Fourth item                                                 |
| `1. First item   1. Second item   1. Third item   1. Fourth item`                                               | `<ol>   <li>First item</li>   <li>Second item</li>   <li>Third item</li>   <li>Fourth item</li>   </ol>`                                                                     | 1. First item<br>2. Second item<br>3. Third item<br>4. Fourth item                                                 |
| `1. First item   8. Second item   3. Third item   5. Fourth item`                                               | `<ol>   <li>First item</li>   <li>Second item</li>   <li>Third item</li>   <li>Fourth item</li>   </ol>`                                                                     | 1. First item<br>2. Second item<br>3. Third item<br>4. Fourth item                                                 |
| `1. First item   2. Second item   3. Third item       1. Indented item       2. Indented item   4. Fourth item` | `<ol>   <li>First item</li>   <li>Second item</li>   <li>Third item   <ol>   <li>Indented item</li>   <li>Indented item</li>   </ol>   </li>   <li>Fourth item</li>   </ol>` | 1. First item<br>2. Second item<br>3. Third item<br>    1. Indented item<br>    2. Indented item<br>4. Fourth item |

#### [#](https://markdown.com.cn/basic-syntax/lists.html#%E6%9C%89%E5%BA%8F%E5%88%97%E8%A1%A8%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5)有序列表最佳实践

CommonMark and a few other lightweight markup languages let you use a parenthesis (`)`) as a delimiter (e.g., `1) First item`), but not all Markdown applications support this, so it isn’t a great option from a compatibility perspective. For compatibility, use periods only.

| ✅  Do this                       | ❌  Don't do this                 |
| -------------------------------- | -------------------------------- |
| `1. First item   2. Second item` | `1) First item   2) Second item` |

### 无序列表

要创建无序列表，请在每个列表项前面添加破折号 (-)、星号 (*) 或加号 (+) 。缩进一个或多个列表项可创建嵌套列表。

|Markdown语法|HTML|预览效果|
|---|---|---|
|`- First item   - Second item   - Third item   - Fourth item`|`<ul>   <li>First item</li>   <li>Second item</li>   <li>Third item</li>   <li>Fourth item</li>   </ul>`|- First item<br>- Second item<br>- Third item<br>- Fourth item|
|`* First item   * Second item   * Third item   * Fourth item`|`<ul>   <li>First item</li>   <li>Second item</li>   <li>Third item</li>   <li>Fourth item</li>   </ul>`|- First item<br>- Second item<br>- Third item<br>- Fourth item|
|`+ First item   + Second item   + Third item   + Fourth item`|`<ul>   <li>First item</li>   <li>Second item</li>   <li>Third item</li>   <li>Fourth item</li>   </ul>`|- First item<br>- Second item<br>- Third item<br>- Fourth item|
|`- First item   - Second item   - Third item       - Indented item       - Indented item   - Fourth item`|`<ul>   <li>First item</li>   <li>Second item</li>   <li>Third item   <ul>   <li>Indented item</li>   <li>Indented item</li>   </ul>   </li>   <li>Fourth item</li>   </ul>`|- First item<br>- Second item<br>- Third item<br>    - Indented item<br>    - Indented item<br>- Fourth item|

#### [#](https://markdown.com.cn/basic-syntax/lists.html#%E6%97%A0%E5%BA%8F%E5%88%97%E8%A1%A8%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5)无序列表最佳实践

Markdown applications don’t agree on how to handle different delimiters in the same list. For compatibility, don't mix and match delimiters in the same list — pick one and stick with it.

|✅  Do this|❌  Don't do this|
|---|---|
|`- First item   - Second item   - Third item   - Fourth item`|`+ First item   * Second item   - Third item   + Fourth item`|

### 在列表中嵌套其他元素

要在保留列表连续性的同时在列表中添加另一种元素，请将该元素缩进四个空格或一个制表符，如下例所示：

#### [#](https://markdown.com.cn/basic-syntax/lists.html#%E6%AE%B5%E8%90%BD)段落

```
*   This is the first list item.
*   Here's the second list item.

    I need to add another paragraph below the second list item.

*   And here's the third list item.
```

渲染效果如下：

- This is the first list item.
    
- Here's the second list item.
    
    I need to add another paragraph below the second list item.
    
- And here's the third list item.
    

#### [#](https://markdown.com.cn/basic-syntax/lists.html#%E5%BC%95%E7%94%A8%E5%9D%97)引用块

```
*   This is the first list item.
*   Here's the second list item.

    > A blockquote would look great below the second list item.

*   And here's the third list item.
```

渲染效果如下：

- This is the first list item.
    
- Here's the second list item.
    
    > A blockquote would look great below the second list item.
    
- And here's the third list item.
    

#### [#](https://markdown.com.cn/basic-syntax/lists.html#%E4%BB%A3%E7%A0%81%E5%9D%97)代码块

[代码块](https://markdown.com.cn/basic-syntax/lists.html#code-blocks)通常采用四个空格或一个制表符缩进。当它们被放在列表中时，请将它们缩进八个空格或两个制表符。

```
1.  Open the file.
2.  Find the following code block on line 21:

        &lt;html>
          &lt;head>
            &lt;title>Test&lt;/title>
          &lt;/head>

3.  Update the title to match the name of your website.
```

渲染效果如下：

1. Open the file.
    
2. Find the following code block on line 21:
    
    ```
    &lt;html>
      &lt;head>
        &lt;title>Test&lt;/title>
      &lt;/head>
    ```
    
3. Update the title to match the name of your website.
    

#### [#](https://markdown.com.cn/basic-syntax/lists.html#%E5%9B%BE%E7%89%87)图片

```
1.  Open the file containing the Linux mascot.
2.  Marvel at its beauty.

    ![Tux, the Linux mascot](/assets/images/tux.png)

3.  Close the file.
```

渲染效果如下：

1. Open the file containing the Linux mascot.
    
2. Marvel at its beauty.
    
    
    
3. Close the file.
    

#### [#](https://markdown.com.cn/basic-syntax/lists.html#%E5%88%97%E8%A1%A8)列表

You can nest an unordered list in an ordered list, or vice versa.

```
1. First item
2. Second item
3. Third item
    - Indented item
    - Indented item
4. Fourth item
```

渲染效果如下：

1. First item
2. Second item
3. Third item
    - Indented item
    - Indented item
4. Fourth it


---








