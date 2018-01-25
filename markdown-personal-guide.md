# 楔子
既然决定要用 Markdown 来记录，那么首先复习一遍都会使用到 Markdown 哪些相关的内容。
## 注意事项
### 反斜杠
使用反斜杠 \ 可以转义字符以插入普通符号，具体如下表：

    \   反斜线
    `   反引号
    *   星号
    _   底线
    {}  花括号
    []  方括号
    ()  括弧
    #   井字号
    +   加号
    -   减号
    .   英文句点
    !   惊叹号

# 正文
## 段落和换行
在需要**强行**换行的地方键入两个以上的空格然后回车  
基本上 markdown 中的大部分区块元素前后都有一个空格

## 标题
选用类 Atx 形式，在行首插入1到6个 # ，对应到标题1到6阶。

    # 这是 H1
    ## 这是 H2
    ###### 这是 H6

## 区块引用 Blockquotes
区块引用使用 `>` 标记
* 每行都添加标记
* 标记可以嵌套
* 该标记中可以使用其他 markdown 语法

        > This is the first level of quoting.
        >
        > > This is nested blockquote.
        >
        > Back to the first level.

## 列表
* 无序列表：*
* 有序列表: 1.

列表固定缩进对齐
列表项目可以包含多个段落，每个项目下的段落都必须缩进 4 个空格或是 1 个制表符

    1.  This is a list item with two paragraphs. Lorem ipsum dolor

        Vestibulum enim wisi, viverra nec, fringilla in, laoreet

    2.  Suspendisse id sem consectetuer libero luctus adipiscing.

如果要放代码区块的话，该区块就需要缩进两次，也就是 8 个空格或是 2 个制表符：

    *   一列表项包含一个列表区块：

            <代码写在这>

## 代码区块
缩进 4 个空格或是 1 个制表符就可以

## 分隔线
    第一部分
    ***
    第二部分

## 链接
    This is [an example](http://example.com/ "Title") inline link.

## 强调
    *single asterisks*
    **double asterisks**

## 代码
    Use the `printf()` function.

## 图片
    ![Alt text](/path/to/img.jpg "Optional title")

# 参考链接
[Markdown 语法说明 (简体中文版)](http://wowubuntu.com/markdown/)