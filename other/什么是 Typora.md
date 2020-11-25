[TOC]

## 什么是 Typora

Typora是一款轻便简洁的Markdown编辑器，支持即时渲染技术，这也是与其他Markdown编辑器最显著的区别。即时渲染使得你写Markdown就想是写Word文档一样流畅自如，不像其他编辑器的有编辑栏和显示栏。

Typora删除了预览窗口，以及所有其他不必要的干扰。取而代之的是实时预览。

所以 Typora 首先是一个 Markdown 文本编辑器，它支持且仅支持 Markdown 语法的文本编辑。在 [Typora 官网](https://typora.io/) 上他们将 Typora 描述为 「A truly **minimal** markdown editor. 」。

##  Markdown介绍

Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档。

Markdown 语言在 2004 由约翰·格鲁伯（英语：John Gruber）创建。

Markdown 编写的文档可以导出 HTML 、Word、图像、PDF、Epub 等多种格式的文档。

Markdown 编写的文档后缀为 .md, .markdown。



## Typora 常用快捷键汇总

```
Ctrl+1  一阶标题   
Ctrl+2  二阶标题   
Ctrl+3  三阶标题   
Ctrl+4  四阶标题   
Ctrl+5  五阶标题   
Ctrl+6  六阶标题   
Ctrl+=  提升标题级别
Ctrl+-  降低标题级别
Ctrl+L  选中某句话  
Ctrl+D  选中某个单词 
Ctrl+B  字体加粗
Ctrl+I  字体倾斜
Ctrl+U  下划线
Ctrl+Home   返回Typora顶部
Ctrl+End    返回Typora底部
Ctrl+ALT+T  创建表格
Ctrl+K  创建超链接
Ctrl+F  搜索
Ctrl+E  选中相同格式的文字  
Ctrl+H  搜索并替换
Alt+Shift+5 删除线
Ctrl+Shift+I    插入图片
Ctrl+Shift+M    公式块 
Ctrl+Shift+Q    引用

注：一些实体符号需要在实体符号之前加”\”才能够显示
```





## Markdown 语法

### 段落结构

#### 标题

\# 用来表示标题就像HTML 的 h 标签 ，这里 # 代表 h1 ,## 代表 h2 ,一直到 ###### 代表h6

![image-20201121180537912](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/21/18:05:38-image-20201121180537912.png)

#### 无序列表

> 使用 * + - 都可以创建一个无序列表

- AAA
- BBB
- CCC

#### 有序列表

使用 1. 2. 3. 创建有序列表

1. AAA
2. BBB
3. CCC

####  任务列表

`- [ ] ` 表示位选中  `- [X]` 表示选中

- [ ] 
- [x] 

- [x] GFM task list 1
- [x] GFM task list 2
- [ ] GFM task list 3
    - [ ] GFM task list 3-1
    - [ ] GFM task list 3-2
    - [ ] GFM task list 3-3
- [ ] GFM task list 4
    - [ ] GFM task list 4-1
    - [ ] GFM task list 4-2



#### 代码块

在Typora中插入程序代码的方式有两种：使用反引号 `（~ 键）、使用缩进（Tab）。

- 插入行内代码输入1个反引号（\`） + 回车，即插入一个单词或者一句代码的情况，使用 `code` 这样的形式插入。

- 插入多行代码输入3个反引号（`） + 回车，**并在后面选择一个语言名称即可实现语法高**, 你可以在3个反引号+回车 敲击结束之后选择一个语言，也可以在只敲击3个反引号 之后输入相关语言名称前缀来选择相应的语言

  ![image-20201121181710436](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/21/18:17:10-image-20201121181710436.png)

![image-20201121181737416](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/21/18:17:37-image-20201121181737416.png)

```java
@SpringBootApplication
@EnableScheduling
public class BlogApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogApplication.class, args);
    }
}

```



#### 表格

输入 `| 表头1 | 表头2 |`并回车。即可创建一个包含2列表。或者按快捷键 `Ctrl + T`弹出对话框，选择属性，点击确定即可生成

![image-20201121182232572](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/21/18:22:33-image-20201121182232572.png)

不管是哪种方式，第一行为表头，第二行为分割表头和主体部分，第三行开始每一行为一个表格行 列与列之间用管道符号`|` 隔开

还可设置对齐方式(表头与内容之间)，如果不使用对齐标记，内容默认左对齐，表头居中对齐

- 左对齐 ：|
- 右对齐 |：
- 中对齐 ：|：



#### 分割线

输入 `***` 或者 `---` 再按回车即可绘制一条水平线，其实可以看到标题标签是自带分割线的

***

---



#### 目录（TOC）

输入 `[ toc ]` 然后回车，即可创建一个“目录”。TOC从文档中提取所有标题，其内容将自动更新。



#### 引用

输入 > + 空格  引用内容 即可

>  我是被引用的人

### 文本修饰

这部分可以在软件的格式菜单中看到

#### 斜体

使用 `*单个星号*` 或者 `_单下划线_` 可以字体倾斜。快捷键 `Ctrl + I`

\*例子\*  效果 *例子*  *你好吗*



#### 加粗

使用 `**两个星号**` 或者 `__两个下划线__` 可以字体加粗。快捷键 `Ctrl + B`

**加粗**



#### 加粗斜体

使用`***加粗斜体***`可以加粗斜体。

***加粗斜体***



#### 删除线

使用`~~删除线~~` 快捷键 `Alt + Shift + 5`

~~删除线~~

~~似懂非懂~~



#### 下划线

通过`<u>下划线的内容</u>` 或者 快捷键`Ctrl + U`可实现下划线

<u>发顺丰的</u>



#### 表情符号

Github的Markdown语法支持添加emoji表情，输入不同的符号码（两个冒号包围的字符）可以显示出不同的表情。

:smile -- 无法显示

😺



#### 高亮

`==高亮==`(需在设置中打开该功能，需要注意的是==和文本之间不能有空格)

==我是自带火花的人==



#### 文本居中

使用 `<center>这是要居中的内容</center>`可以使文本居中

<center>这是要居中的文本内容</center>



#### 脚注

脚注使用这个语法 [\^ 脚注内容]，脚注标识可以为字母数字下划线，但支持中文。脚注内容可为任意字符，包括中文。

你好[^你好]

##### 下标

可以使用 `<sub>文本</sub>`实现下标

H<sub>2</sub>O

##### 上标

可以使用`<sup>文本</sup>`实现上标。

y=X<sup>2</sup>



#### 数学公式

当你需要在编辑器中插入数学公式时，可以使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现。根据需要加载 Mathjax 对数学公式进行渲染。

按下 `$$`，然后按下回车键，即可进行数学公式的编辑

`$$ \mathbf{V}_1\times\mathbf{V}_2 = \mathbf{X}_3 $$`

$$ \mathbf{V}_1\times\mathbf{V}_2 = \mathbf{X}_3 $$

> LaTeX 本身就有自己的语法所以需要使用数学公式的话，需要单独去学



`$$E=mc^2$$` 行内的公式$$E=mc^2$$行内的公式，行内的$$E=mc^2$$公式。



`$$x > y$$`            $$x > y$$

$$\(\sqrt{3x-1}+(1+x)^2\)$$
                    
$$\sin(\alpha)^{\theta}=\sum_{i=0}^{n}(x^i + \cos(f))$$



```
$$
	\begin{matrix}
	1 & x & x^2\\
	1 & y & y^2\\
	1 & z & z^2\\
	\end{matrix}
$$

```


$$
\begin{matrix}
	1 & x & x^2\\
	1 & y & y^2\\
	1 & z & z^2\\
	\end{matrix}
$$

### 多媒体

#### 图片

手动 输入 `![]()`  或者 快捷键 Ctrl+command+i  或者可以直接拖进来

![]()

#### 链接

手动输入 [提示信息](链接)

[百度](www.baidu.com)

#### 视屏

