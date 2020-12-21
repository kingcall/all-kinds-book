



![image-20201221201637792](C:\Users\Wenqliu\AppData\Roaming\Typora\typora-user-images\image-20201221201637792.png)

## JupyterLab简介

JupyterLab是Jupyter主打的最新数据科学生产工具，某种意义上，它的出现是为了取代Jupyter Notebook。不过不用担心Jupyter Notebook会消失，JupyterLab包含了Jupyter Notebook所有功能。

JupyterLab作为一种基于web的集成开发环境，你可以使用它编写notebook、操作终端、编辑markdown文本、打开交互模式、查看csv文件及图片等功能。

你可以把JupyterLab当作一种究极进化版的Jupyter Notebook。原来的单兵作战，现在是空陆空联合协作。

![img](https://pic4.zhimg.com/80/v2-9e8c96c35cfa3e8351a9ec8454452d4b_1440w.jpg)



总之，JupyterLab有以下特点：

- **交互模式：**Python交互式模式可以直接输入代码，然后执行，并立刻得到结果，因此Python交互模式主要是为了调试Python代码用的
- **内核支持的文档：**使你可以在可以在Jupyter内核中运行的任何文本文件（Markdown，Python，R等）中启用代码
- **模块化界面：**可以在同一个窗口同时打开好几个notebook或文件（HTML, TXT, Markdown等等），都以标签的形式展示，更像是一个IDE
- **镜像notebook输出：**让你可以轻易地创建仪表板
- **同一文档多视图：**使你能够实时同步编辑文档并查看结果
- **支持多种数据格式：**你可以查看并处理多种数据格式，也能进行丰富的可视化输出或者Markdown形式输出
- **云服务：**使用Jupyter Lab连接Google Drive等服务，极大得提升生产力

## 安装Jupyter Lab

你可以使用`pip`、`conda`安装Jupyter Lab

**pip**
`pip`可能是大多数人使用包管理工具，如果使用`pip`安装，请在命令行执行：

```text
pip install jupyterlab
```

**conda**
如果你是Anaconda用户，那么可以直接用`conda`安装，请在命令行执行：

```text
conda install -c conda-forge jupyterlab
```

## 运行Jupyter Lab

在安装Jupyter Lab后，接下来要做的是运行它。
你可以在命令行使用`jupyter-lab`或`jupyter lab`命令，然后默认浏览器会自动打开Jupyter Lab。

![img](https://pic3.zhimg.com/80/v2-1acef3c62334a0096269490153a49712_1440w.jpg)



![img](https://pic2.zhimg.com/80/v2-83398e84480a7c07eb2506c2bd214a75_1440w.jpg)



**启动器**
右侧的选项卡称为启动器，你可以新建notebook、console、teminal或者text文本。
当你创建新的notebook或其他项目时，启动器会消失。 如果您想新建文档，只需单击左侧红圈里的“ +”按钮。

![img](https://pic2.zhimg.com/80/v2-5bf604d8a12b4f70b83baa3b178c655d_1440w.jpg)



**打开文档**
在启动器中点击你想要打开的文档类型，即可以打开相应文档。

![img](https://pic2.zhimg.com/80/v2-2850f3244ddf48ea1002bde7e86beda1_1440w.jpg)

单击左侧的“ +”按钮，新建多个文档，你会看到：

![img](https://pic1.zhimg.com/80/v2-bf45563b092a8b86a63944dfa53ef008_1440w.jpg)



你还可以使用顶部的菜单栏创建新项目，步骤：file->new，然后选择要创建的文档类型。这和Jupyter Notebook一样，如果你经常使用Notebook，那么应该不会陌生。

你可以打开多个文档后，任何排版组合，只需按住选项卡拖移即可。

![img](https://pic4.zhimg.com/80/v2-91c98244a413cc36c62815e3ed1028e7_1440w.jpg)

当在一个notebook里面写代码时，如果想要实时同步编辑文档并查看执行结果，可以新建该文档的多个视图。步骤：file->new view for notebook

![img](https://pic1.zhimg.com/80/v2-38d0aa50f61c68c01c433e2332802a3c_1440w.jpg)



**文件浏览器**

左侧一栏是文件浏览器，显示从JupyterLab启动的位置可以使用的文件。

![img](https://pic2.zhimg.com/80/v2-e04a74e3ba704e41dc0c74a455f59df9_1440w.jpg)

你可以创建文件夹、上传文件并、新文件列表

![img](https://pic4.zhimg.com/80/v2-637fa95bc332ab8d73f8c03970e8df43_1440w.jpg)



**预览Markdown文本**

![img](https://pic4.zhimg.com/80/v2-ca840fc97dd3a67806b71d71bac099cb_1440w.jpg)

**编辑代码**

![img](https://pic2.zhimg.com/80/v2-70e9edd7d935232ca3fa1a7916af74c5_1440w.jpg)

**预览csv文件**

![img](https://pic1.zhimg.com/80/v2-54ddbde336faf7af9c97c6b0098897b4_1440w.jpg)

**预览geojson文件**

![img](https://pic1.zhimg.com/80/v2-420c86ca5b675995c29a1637569cf9ec_1440w.jpg)



**打开学习文档**
Jupyter Lab支持打开pandas、numpy、matplotlib、scipy、python、ipython、scipy、markdown、notebook等官方文档。步骤：help->选择相应文档

![img](https://pic1.zhimg.com/80/v2-9ca4b02fd1abe1bb6b0c9bef85d00288_1440w.jpg)



![img](https://pic2.zhimg.com/80/v2-39197f2ff4e379a8c296ac82bd5083b5_1440w.jpg)

**切换背景主题**
Jupyter Lab支持两种背景主题，白色和黑色。步骤：settings->jupyterlab theme

![img](https://pic2.zhimg.com/80/v2-4f7f0c8f405200968415ef9240a29c39_1440w.jpg)

### 结语

本文介绍了JupyterLab的安装和使用方法，并且对其特性做了很多讲解，作为Jupyter主推的下一代数据科学开发工具，JupyterLab有着非常光明的前景！还在使用notebook的你何不试试JupyterLab，体验一下更加贴近IDE的人性化编程环境。