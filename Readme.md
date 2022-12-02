# Markdown基本语法

## **1. 标题语法**

根据前边#号的多少定义标题的等级，分别从一个#到六个#，对应一级标题到六级标题
![标题](./Md_images/Readme/%E6%A0%87%E9%A2%98.jpg "标题")

## **2. 段落语法**

段落的创建是使用空白行对文本进行分割，需注意的是不要在段落行前方使用**空格**或者**制表符**缩进段落
![](./Md_images/Readme/%E6%AE%B5%E8%90%BD.jpg "段落")

## **3. 换行语法**

在每一行的末尾添加两个或多个**空格**然后**回车**便可以创建一个换行即(space+space+enter)  
此行便是换行实例，当然了，也可以使用HTML语法规则中的**br**标签进行换行
![](./Md_images/Readme/%E6%8D%A2%E8%A1%8C.jpg)

## **4. 强调语法**

主要是通过将文本设置为粗体或者斜体强调其重要性

### **粗体**

若需加粗文本，需在想要强调的文本前后分别添加两个**星号**(**)或者两个 __下划线__(_)

（自行总结，不一定准确）需注意，下划线的用法需要在前边的两个下划线前添加符号，即(symbol+\_\_+文本+\_\_)

### *斜体*

若需用斜体显示文本，在文本前后添加一个*星号*(*)或者一个 _下划线_(_)  
对于下划线的用法同粗体

### ***粗斜体***

在文本前后添加三个***星号***(*)或者 ___下划线___(_)

![下划线说明](./Md_images/Readme/%E4%B8%8B%E5%88%92%E7%BA%BF.png "下划线说明")

## **5. 引用语法**

若需创建块引用，请在块前添加一个\>符号，效果如下：

> 这是个块引用的演示效果
>> 这是第二层嵌套
>>> 这是第三层嵌套引用
>>>> 这是第四层嵌套引用
>
> 这是个多段落块引用

引用可以结合其他的markdown格式元素
![](./Md_images/Readme/%E5%BC%95%E7%94%A8.png)

## **6.列表语法**

包括有序列表和无序列表两种，由多个条目组成

### 有序列表

创建有序列表，只需在每个列表项前添加数字并紧跟一个英文句点，数字不必按照数学顺序排列，但应从1开始
1. first
2. second
    1. second-first
2. third

![](./Md_images/Readme/%E6%9C%89%E5%BA%8F%E5%88%97%E8%A1%A8.jpg)

### 无序列表

要创建无序列表，请在每个列表项前面添加破折号 (-)、星号 (*) 或加号 (+) 。缩进一个或多个列表项可创建嵌套列表。
- \-无序列表
* \*无序列表
+ \+无序列表
    + 嵌套无序列表
        + 再嵌套列表

![](./Md_images/Readme/%E6%97%A0%E5%BA%8F%E5%88%97%E8%A1%A8.jpg)

列表语法较为灵活，可以嵌套多种其他的markdown规则（段落，引用块，代码块，图片，列表）

## 7. 代码语法

要将文本表示为代码，应将其包裹在反引号\`中。  

`python matlab java`

代码块

`
    python
    matlab
    java
`
![](./Md_images/Readme/%E4%BB%A3%E7%A0%81%E5%9D%97.jpg)

## 8. 分隔线语法

要创建分隔线，请在单独一行上使用三个或多个星号 (***)、破折号 (---) 或下划线 (___) ，并且不能包含其他内容。

***
*分隔线

---
-分隔线

___
_分隔线

![](./Md_images/Readme/%E5%88%86%E9%9A%94%E7%BA%BF.png)

## 9. 链接语法

链接文本放在中括号内，链接地址放在后面的括号中，链接title可选。

超链接Markdown语法代码：\[超链接显示名\]\(超链接地址 "超链接title"\)

例：
[Markdown官方教程](https://markdown.com.cn/basic-syntax/links.html "Markdown")

[hobbit-hole][1]  

[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle

![](./Md_images/Readme/%E9%93%BE%E6%8E%A51.jpg)
![](./Md_images/Readme/%E9%93%BE%E6%8E%A52.jpg)

## 10. 图片语法

要添加图像，请使用感叹号 (!), 然后在方括号增加替代文本，图片链接放在圆括号里，括号里的链接后可以增加一个可选的图片标题文本。

插入图片Markdown语法代码：\!\[图片alt]\(图片链接 "图片title"\)。


![纯图片链接](./Md_images/Readme/wallhaven-3l9zj3.jpg)

可以给图片添加链接，将图像的Markdown 括在方括号中，然后将链接添加在圆括号中。
[![带链接图片](./Md_images/Readme/wallhaven-zym92v.jpg "例图2")](https://wallhaven.cc/)

## 11. 转义字符语法

要显示原本用于格式化 Markdown 文档的字符，请在字符前面添加反斜杠字符 \ 。

[![](./Md_images/Readme/%E8%BD%AC%E4%B9%89.jpg)](https://markdown.com.cn/basic-syntax/escaping-characters.html)

## 12. 内嵌HTML标签

暂时不用，用得上再看。[链接](https://markdown.com.cn/basic-syntax/htmls.html)


# [扩展语法](https://markdown.com.cn/extended-syntax/)

## 1. 表格

![](./Md_images/Readme/%E6%89%A9%E5%B1%95%E8%A1%A8%E6%A0%BC.jpg)

## 2. 围栏代码块

![](./Md_images/Readme/%E6%89%A9%E5%B1%95%E4%BB%A3%E7%A0%81%E5%9D%97.jpg)

## 3. 脚注

![](./Md_images/Readme/%E6%89%A9%E5%B1%95%E8%84%9A%E6%B3%A8.jpg)

## 4. 标题编号

![](./Md_images/Readme/%E6%89%A9%E5%B1%95%E6%A0%87%E9%A2%98%E7%BC%96%E5%8F%B7.jpg)

## 5. 定义列表

![](./Md_images/Readme/%E6%89%A9%E5%B1%95%E5%AE%9A%E4%B9%89%E5%88%97%E8%A1%A8.jpg)

## 6. 删除线

![](./Md_images/Readme/%E6%89%A9%E5%B1%95%E5%88%A0%E9%99%A4%E7%BA%BF.jpg)

## 7. 任务列表

![](./Md_images/Readme/%E6%89%A9%E5%B1%95%E4%BB%BB%E5%8A%A1%E5%88%97%E8%A1%A8.jpg)

## 8. Emoji

![](./Md_images/Readme/%E6%89%A9%E5%B1%95emoji.jpg)

[Emojipedia](https://emojipedia.org/)  

[表情符号简码列表](https://gist.github.com/rxaviers/7360908)

## 9. 自动网址

![](./Md_images/Readme/%E8%87%AA%E5%8A%A8%E7%BD%91%E5%9D%80.jpg)