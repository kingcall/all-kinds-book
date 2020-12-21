最近在从Jupyter Notebook向Jupyter Lab转，倍感舒适。

Lab和Notebook是一家人，前者算后者的升级加强版。

Lab相比较Notebook最大的优势在于它的**用户界面集成强，适合多文档协助工作**。

而且Lab是可拓展的，插件丰富，非常像vs code，但又完美地继承了Notebook的所有优点。

![img](https://pic3.zhimg.com/80/v2-ff0fffc43af443624e7156964d7e1a12_1440w.jpg)

之前写过Lab的介绍文档，这次再来聊聊Lab里那些好用到爆炸的**插件**。

[神器 | JupyterLab，极其强大的下一代notebook！](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/U0GtZkc9a7xZhy8EgVXW2w)

在Lab中安装插件并不需要pip，直接在界面侧栏就可以搜索你需要的插件。

当然在此之前，你需要设置显示插件栏，因为默认插件栏不显示。

![img](https://pic2.zhimg.com/80/v2-78c59f45d6b021d345986f0c658282fd_1440w.jpg)默认无插件栏

点击菜单栏`Settings`下拉框中的`Advanced Settings Editor`选项，会出现一个设置页面。

![img](https://pic2.zhimg.com/80/v2-6cbf26a299b6e15ddd86804e98563165_1440w.jpg)设置

接着，点击`Extension Manager`，并且在右边的空白框里填上`{'enabled':true}`，并且按右上角的保存按钮。

![img](https://pic4.zhimg.com/80/v2-7d0877540b94b10b9f8d9665e29f9fcb_1440w.jpg)设置显示插件栏

最后，你会看到Lab右边会出现插件栏的按钮，我已经安装过一些插件。

你可以在搜索栏搜索想要的插件，并直接安装。

![img](https://pic4.zhimg.com/80/v2-a088d6d1240ba2e120003b38bd1caff3_1440w.jpg)插件栏

## 下面就来介绍15款非常nice的Jupyter Lab插件

## 1. github

安装这个插件后，JupyterLab左侧会出现一个github栏按钮，你可以在里面搜索github项目，并且打开项目里面的文件，如果是notebook文件，能够直接运行代码。

这个插件非常适合在Lab上学习github项目，方便高效。
[https://github.com/jupyterlab/jupyterlab-github](https://link.zhihu.com/?target=https%3A//github.com/jupyterlab/jupyterlab-github)

![img](https://pic2.zhimg.com/80/v2-b70f93ed2267a99923a31a25b562df51_1440w.jpg)github插件

## 2. toc

这是一个Lab的目录插件，安装后就能很方便地在Lab上展示notebook或者markdown的目录。
目录可以滚动，并且能展示或隐藏子目录。
[https://github.com/jupyterlab/jupyterlab-toc](https://link.zhihu.com/?target=https%3A//github.com/jupyterlab/jupyterlab-toc)

![img](https://pic4.zhimg.com/80/v2-c8371f49fd1910d53ca08883dac0bbf3_1440w.jpg)

## 3. LaTeX

支持在线编辑并预览LaTeX文档。
[https://github.com/jupyterlab/jupyterlab-latex](https://link.zhihu.com/?target=https%3A//github.com/jupyterlab/jupyterlab-latex)

![img](https://pic2.zhimg.com/80/v2-50271619f157e9d4b5ddf00a60d2bd19_1440w.jpg)

## 4. HTML

该插件允许你在Jupyter Lab内部呈现HTML文件，这在打开例如d3可视化效果时非常有用。
[https://github.com/mflevine/jupyterlab_html](https://link.zhihu.com/?target=https%3A//github.com/mflevine/jupyterlab_html)

![img](https://pic3.zhimg.com/v2-f3641c9ebbe114d95aa9189c8b5c078e_b.jpg)

## 5. plotly

该插件可以在Lab中展示plotly可视化效果。
[https://github.com/jupyterlab/jupyter-renderers](https://link.zhihu.com/?target=https%3A//github.com/jupyterlab/jupyter-renderers)

![img](https://pic3.zhimg.com/v2-31cb7b9257ebcb7e9ba235dfe3ddd06e_b.jpg)

## 6. bokeh

该插件可以在Lab中展示bokeh可视化效果。
[https://github.com/bokeh/jupyter_bokeh](https://link.zhihu.com/?target=https%3A//github.com/bokeh/jupyter_bokeh)

![img](https://pic1.zhimg.com/v2-8fdf2cef245c5a9df0a9a2ea7c172580_b.jpg)

## 7. matplotlib

该插件可以在Lab中启用matplotlib可视化交互功能。
[https://github.com/matplotlib/jupyter-matplotlib](https://link.zhihu.com/?target=https%3A//github.com/matplotlib/jupyter-matplotlib)

![img](https://pic1.zhimg.com/v2-64b4c510b1b8d6316ff300bae05a2d68_b.jpg)

## 8. drawio

该插件可以在Lab中启用drawio绘图工具，drawio是一款非常棒的流程图工具。
[https://github.com/QuantStack/jupyterlab-drawio](https://link.zhihu.com/?target=https%3A//github.com/QuantStack/jupyterlab-drawio)

![img](https://pic2.zhimg.com/80/v2-37988ff1b9f9e330b73c74f55ab023e5_1440w.jpg)

## 9. sql

该插件可以在Lab中连接数据库，并进行sql查询和修改操作。
[https://github.com/pbugnion/jupyterlab-sql](https://link.zhihu.com/?target=https%3A//github.com/pbugnion/jupyterlab-sql)

![img](https://pic1.zhimg.com/v2-c1854188bb3c82ef23d188dbee190380_b.jpg)

## 10. variableinspector

该插件可以在Lab中展示代码中的变量及其属性，类似RStudio中的变量检查器。你可以一边撸代码，一边看有哪些变量。
[https://github.com/lckr/jupyterlab-variableInspector](https://link.zhihu.com/?target=https%3A//github.com/lckr/jupyterlab-variableInspector)

![img](https://pic1.zhimg.com/v2-3db9cf697a172e63fe0270b963972f30_b.jpg)

## 11. dash

该插件可以在Lab中展示plotly dash交互式面板。
[https://awesomeopensource.com/project/plotly/jupyterlab-dash](https://link.zhihu.com/?target=https%3A//awesomeopensource.com/project/plotly/jupyterlab-dash)

![img](https://pic4.zhimg.com/v2-965d73b987888d56cc64696213c39533_b.jpg)

## 12. gather

在Lab中清理代码，恢复丢失的代码以及比较代码版本的工具。
[https://github.com/microsoft/gather](https://link.zhihu.com/?target=https%3A//github.com/microsoft/gather)

![img](https://pic4.zhimg.com/v2-eb2a45e367173b62953842a4bcd8e61b_b.jpg)

## 13. go to Definition

该插件用于在Lab笔记本和文件编辑器中跳转到变量或函数的定义。
[https://github.com/krassowski/jupyterlab-go-to-definition](https://link.zhihu.com/?target=https%3A//github.com/krassowski/jupyterlab-go-to-definition)

![img](https://pic4.zhimg.com/v2-ce141a5fced260c9806fde414693e95b_b.jpg)

## 14. lsp

该插件用于自动补全、参数建议、函数文档查询、跳转定义等。
[https://github.com/krassowski/jupyterlab-lsp](https://link.zhihu.com/?target=https%3A//github.com/krassowski/jupyterlab-lsp)

![img](https://pic3.zhimg.com/80/v2-b127db98b06fc3d668f038faac4435f2_1440w.jpg)

![img](https://pic4.zhimg.com/80/v2-39803361fc1917a3de9b35e430f049fb_1440w.jpg)

![img](https://pic1.zhimg.com/80/v2-1064bcb905ed1b61fcbc973dc588d300_1440w.jpg)

![img](https://pic3.zhimg.com/80/v2-f63c3e4e0dbca69173eab3b6a24d36ca_1440w.jpg)

## 15. spreadsheet

该插件用于在Lab上显示excel表格，只读模式。
[https://github.com/quigleyj97/jupyterlab-spreadsheet](https://link.zhihu.com/?target=https%3A//github.com/quigleyj97/jupyterlab-spreadsheet)

![img](https://pic3.zhimg.com/80/v2-f737429db282d822b2338cee10fa7bc2_1440w.jpg)

## 小结

Jupyter Lab还有很多强大的拓展插件，这里也没办法一一列举。感兴趣的去github找找，提供一个项目供参考。
[https://github.com/mauhai/awesome-jupyterlab](https://link.zhihu.com/?target=https%3A//github.com/mauhai/awesome-jupyterlab)

[
  ](https://union-click.jd.com/jdc?e=jdext-1196936992187953152-0&p=AyIGZRtTHAUSAVEYWRAyEgddE1kVABc3EUQDS10iXhBeGlcJDBkNXg9JHUlSSkkFSRwSB10TWRUAFxgMXgdIMhMCUn88ZnRpZT5THElBdlsCHw1tdkQLWStbHAIQD1QaWxIBIgdUGlsRBxEEUxprJQIXNwd1g6O0yqLkB4%2B%2FjcePwitaJQIWAVwbXxYDEgFUGlglAhoDZc31gdeauIyr%2FsOovNLYq46cqca50ytrJQEiXABPElAeEgddHFMXBRoPVhtSEwoWBVUfWAkDIgdUGlsdBRYEURo1FGwSD1YcUxILFA5VK1slAiJYEUYGJQATBlcZ)