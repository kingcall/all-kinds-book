# Jupyter Lab简介

Jupyter源于Ipython Notebook，是使用Python（也有R、Julia、Node等其他语言的内核）进行代码演示、数据分析、可视化、教学的很好的工具，对Python的愈加流行和在AI领域的领导地位有很大的推动作用。

Jupyter Lab是Jupyter的一个拓展，提供了更好的用户体验，例如可以同时在一个浏览器页面打开编辑多个Notebook，Ipython console和terminal终端，并且支持预览和编辑更多种类的文件，如代码文件，Markdown文档，json，yml，csv，各种格式的图片，vega文件（一种使用json定义图表的语言)和geojson（用json表示地理对象），还可以使用Jupyter Lab连接Google Drive等[云存储](https://cloud.tencent.com/product/cos?from=10680)服务，极大得提升了生产力。

# 体验Jupyter Lab

我们可以通过Try Jupyter网站(https://jupyter.org/try)使用Jupyter Lab。



![img](https://ask.qcloudimg.com/draft/1000830/xmzz8a0e88.png?imageView2/2/w/1620)Try Jupyter Lab

![img](https://ask.qcloudimg.com/draft/1000830/8e50q5ucqj.png?imageView2/2/w/1620)Jupyter Lab

![img](https://ask.qcloudimg.com/draft/1000830/jhmy8ib4jr.png?imageView2/2/w/1620)打开多个Notebook

![img](https://ask.qcloudimg.com/draft/1000830/49f9onr03r.png?imageView2/2/w/1620)新建Ipython Console

![img](https://ask.qcloudimg.com/draft/1000830/g3856kf635.png?imageView2/2/w/1620)预览Markdown

![img](https://ask.qcloudimg.com/draft/1000830/l72wi82qjw.png?imageView2/2/w/1620)编辑代码

![img](https://ask.qcloudimg.com/draft/1000830/wv78vohka5.png?imageView2/2/w/1620)预览CSV

![img](https://ask.qcloudimg.com/draft/1000830/a3ubslu1nr.png?imageView2/2/w/1620)预览Geojson

![img](https://ask.qcloudimg.com/draft/1000830/bmvvjq5i44.png?imageView2/2/w/1620)管理正在运行的实例

# 使用Jupyter Lab

## 安装Jupyter Lab

使用pip安装

```bash
pip install jupyterlab
```

## 运行Jupyter Lab

```bash
jupyter-lab
```

Jupyter Lab会继承Jupyter Notebook的配置（地址，端口，密码等）

## 访问Jupyter Lab

浏览器访问http://localhost:8888

# 结语

本文对Jupyter Lab进行了简单的介绍，希望可以给大家带来新的选择。

祝各位享受生活，享受代码。