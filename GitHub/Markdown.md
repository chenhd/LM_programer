#Markdown

link：    [markdown-basics](https://help.github.com/articles/markdown-basics/)
PS：规则比较复杂，多用空行隔开

##Paragraphs(段落)
如果想在Markdown中使用段落只需要在文本后面跟进一个或多个的空行

##Headings（标题）
你可以在你的文本前面加一个或多个的``#``符号来将其变为标题，``#``的数量代表标题的大小
```
# The largest heading (an <h1> tag)
## The second largest heading (an <h2> tag)
…
###### The 6th largest heading (an <h6> tag)
```

##Blockquotes（引用）
你可以使用``>``表示引用
> this is the Blockquotes

##Styling text
``你可以使文本*倾斜*, _倾斜_和**粗体**``
>你可以使文本*倾斜*, _倾斜_和**粗体**

``**Everyone _must_ *attend* the meeting at 5 o'clock today.**``
>**Everyone _must_ *attend* the meeting at 5 o'clock today.**

##Lists
###Unordered lists
你可以在列表元素之前使用`*`或`-`来制作无序列表(后需接空格)
* item
* item
* item
* item
- item

or
- qwe
- qwe
- qwe

###Ordered lists
你可以在列表元素之前使用一个数字来制作有序列表(后需接点和空格)

1. item1
2. item2
3. item3
4. qwer

###Nested lists
你可以通过用2个空格缩进来使用嵌套List(可嵌入多层)

1. item1
  1. nested1
  2. nested2
    - qwe
    * qwe
    - qwe
2. item2
3. item3

##Code formatting
###Inline formats
使用单倒引号（`）来将文本格式化成特定的格式，所有被格式化的文本将不再转换（好像不能转义本身）
>Here's an idea: why don't we take `SuperiorProject` and turn it into `**Reasonable**Project`.

###Multiple lines
使用三倒引号（```）来将文本格式化成独特的块
Check out this neat program I wrote:

```
x = 0
x = 2 + 2
what is x
```

##Links
你可以通过`[]`包装文本，`()`包装连接来创建一个链接
>`[markdown-basics](https://help.github.com/articles/markdown-basics/)`

[markdown-basics](https://help.github.com/articles/markdown-basics/)
