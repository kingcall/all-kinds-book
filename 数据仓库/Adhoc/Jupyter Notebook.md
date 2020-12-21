# 一、什么是Jupyter Notebook？

## 1. 简介

> Jupyter Notebook是基于网页的用于交互计算的应用程序。其可被应用于全过程计算：开发、文档编写、运行代码和展示结果。——[Jupyter Notebook官方介绍](https://link.jianshu.com?t=https%3A%2F%2Fjupyter-notebook.readthedocs.io%2Fen%2Fstable%2Fnotebook.html)

简而言之，Jupyter Notebook是以网页的形式打开，可以在网页页面中**直接**编写代码和运行代码，代码的运行结果也会直接在代码块下显示。如在编程过程中需要编写说明文档，可在同一个页面中直接编写，便于作及时的说明和解释。

## 2. 组成部分

### ① 网页应用

网页应用即基于网页形式的、结合了编写说明文档、数学公式、交互计算和其他富媒体形式的工具。**简言之，网页应用是可以实现各种功能的工具。**

### ② 文档

即Jupyter Notebook中所有交互计算、编写说明文档、数学公式、图片以及其他富媒体形式的输入和输出，都是以文档的形式体现的。

这些文档是保存为后缀名为`.ipynb`的`JSON`格式文件，不仅便于版本控制，也方便与他人共享。

此外，文档还可以导出为：HTML、LaTeX、PDF等格式。

## 3. Jupyter Notebook的主要特点

1. 编程时具有**语法高亮**、*缩进*、*tab补全*的功能。
2. 可直接通过浏览器运行代码，同时在代码块下方展示运行结果。
3. 以富媒体格式展示计算结果。富媒体格式包括：HTML，LaTeX，PNG，SVG等。
4. 对代码编写说明文档或语句时，支持Markdown语法。
5. 支持使用LaTeX编写数学性说明。

# 二、安装Jupyter Notebook

## 0. 先试用，再决定

如果看了以上对Jupyter Notebook的介绍你还是拿不定主意究竟是否适合你，那么不要担心，你可以先**免安装试用体验**一下，[戳这里](https://link.jianshu.com?t=https%3A%2F%2Ftry.jupyter.org%2F)，然后再做决定。

值得注意的是，官方提供的同时试用是有限的，如果你点击链接之后进入的页面如下图所示，那么不要着急，过会儿再试试看吧。



![img](https:////upload-images.jianshu.io/upload_images/5101171-852de98b32232333?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

试用失败

如果你足够幸运，那么你将看到如下界面，就可以开始体验啦。

主界面



![img](https:////upload-images.jianshu.io/upload_images/5101171-f578a4a9de4af239?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

试用成功

编辑页面



![img](https:////upload-images.jianshu.io/upload_images/5101171-3ca8bce68afed86b?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

编辑页面

## 1. 安装

### ① 安装前提

安装Jupyter Notebook的前提是需要安装了Python（3.3版本及以上，或2.7版本）。

### ② 使用Anaconda安装

如果你是小白，那么建议你通过安装Anaconda来解决Jupyter Notebook的安装问题，因为Anaconda已经自动为你安装了Jupter Notebook及其他工具，还有python中超过180个科学包及其依赖项。

你可以通过进入Anaconda的[官方下载页面](https://link.jianshu.com?t=https%3A%2F%2Fwww.anaconda.com%2Fdownload%2F%23macos)自行选择下载；如果你对阅读**英文文档**感到头痛，或者对**安装步骤**一无所知，甚至也想快速了解一下**什么是Anaconda**，那么可以前往我的另一篇文章[Anaconda介绍、安装及使用教程](https://link.jianshu.com?t=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F32925500)。你想要的，都在里面！

常规来说，安装了Anaconda发行版时已经自动为你安装了Jupyter Notebook的，但如果没有自动安装，那么就在终端（Linux或macOS的“终端”，Windows的“Anaconda Prompt”，以下均简称“终端”）中输入以下命令安装：



```undefined
conda install jupyter notebook
```

### ③ 使用pip命令安装

如果你是有经验的Python玩家，想要尝试用pip命令来安装Jupyter Notebook，那么请看以下步骤吧！接下来的命令都输入在终端当中的噢！

1. 把pip升级到最新版本

   - Python 3.x

   

   ```undefined
   pip3 install --upgrade pip
   ```

   - Python 2.x

   

   ```undefined
   pip install --upgrade pip
   ```

- 注意：老版本的pip在安装Jupyter Notebook过程中或面临依赖项无法同步安装的问题。因此**强烈建议**先把pip升级到最新版本。

1. 安装Jupyter Notebook

   - Python 3.x

   

   ```undefined
   pip3 install jupyter
   ```

   - Python 2.x

   

   ```undefined
   pip install jupyter
   ```

# 三、运行Jupyter Notebook

## 0. 帮助

如果你有任何jupyter notebook命令的疑问，可以考虑查看官方帮助文档，命令如下：



```bash
jupyter notebook --help
```

或



```undefined
jupyter notebook -h
```

## 1. 启动

### ① 默认端口启动

在终端中输入以下命令：



```undefined
jupyter notebook
```

执行命令之后，在终端中将会显示一系列notebook的服务器信息，同时浏览器将会自动启动Jupyter Notebook。

启动过程中终端显示内容如下：



```csharp
$ jupyter notebook
[I 08:58:24.417 NotebookApp] Serving notebooks from local directory: /Users/catherine
[I 08:58:24.417 NotebookApp] 0 active kernels
[I 08:58:24.417 NotebookApp] The Jupyter Notebook is running at: http://localhost:8888/
[I 08:58:24.417 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```

- 注意：之后在Jupyter Notebook的所有操作，都请保持终端**不要关闭**，因为一旦关闭终端，就会断开与本地服务器的链接，你将无法在Jupyter Notebook中进行其他操作啦。

浏览器地址栏中默认地将会显示：`http://localhost:8888`。其中，“localhost”指的是本机，“8888”则是端口号。

![img](https:////upload-images.jianshu.io/upload_images/5101171-54e6e9a6b932e371?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

URL



如果你**同时**启动了多个Jupyter Notebook，由于默认端口“8888”被占用，因此地址栏中的数字将从“8888”起，每多启动一个Jupyter Notebook数字就加1，如“8889”、“8890”……

### ② 指定端口启动

如果你想自定义端口号来启动Jupyter Notebook，可以在终端中输入以下命令：



```xml
jupyter notebook --port <port_number>
```

其中，“<port_number>”是自定义端口号，直接以数字的形式写在命令当中，数字两边不加尖括号“<>”。如：`jupyter notebook --port 9999`，即在端口号为“9999”的服务器启动Jupyter Notebook。

### ③ 启动服务器但不打开浏览器

如果你只是想启动Jupyter Notebook的服务器但不打算立刻进入到主页面，那么就无需立刻启动浏览器。在终端中输入：



```undefined
jupyter notebook --no-browser
```

此时，将会在终端显示启动的服务器信息，并在服务器启动之后，显示出打开浏览器页面的链接。当你需要启动浏览器页面时，只需要复制链接，并粘贴在浏览器的地址栏中，轻按回车变转到了你的Jupyter Notebook页面。



![img](https:////upload-images.jianshu.io/upload_images/5101171-50d747de0b61bbca?imageMogr2/auto-orient/strip|imageView2/2/w/894/format/webp)

no_browser

例图中由于在完成上面内容时我同时启动了多个Jupyter Notebook，因此显示我的“8888”端口号被占用，最终分配给我的是“8889”。

## 2. 主页面

### ① 主页面内容

当执行完启动命令之后，浏览器将会进入到Notebook的主页面，如下图所示。



![img](https:////upload-images.jianshu.io/upload_images/5101171-15a4d4600b0e75c6?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Notebook Dashboard

如果你的主页面里边的文件夹跟我的不同，或者你在疑惑为什么首次启动里边就已经有这么多文件夹，不要担心，这里边的文件夹全都是你的家目录里的目录文件。你可以在终端中执行以下2步来查看：

① `cd` 或 `cd -` 或 `cd ~` 或`cd /Users/<user_name>`

- 这个命令将会进入你的家目录。
- “<user_name>” 是用户名。用户名两边不加尖括号“<>”。

② `ls`

- 这个命令将会展示你家目录下的文件。

### ② 设置Jupyter Notebook文件存放位置

如果你不想把今后在Jupyter Notebook中编写的所有文档都直接保存在家目录下，那你需要修改Jupyter Notebook的文件存放路径。

#### ⑴ 创建文件夹/目录

- Windows用户在想要存放Jupyter Notebook文件的**磁盘**中**新建文件夹**并为该文件夹命名；双击进入该文件夹，然后复制地址栏中的路径。
- Linux/macOS用户在想要存放Jupyter Notebook文件的位置**创建目录**并为目录命名，命令为：`mkdir <directory_name>`；进入目录，命令为：`cd <directory_name>`；查看目录的路径，命令为：`pwd`；复制该路径。
- 注意：“<directory_name>”是自定义的目录名。目录名两边不加尖括号“<>”。

#### ⑵ 配置文件路径

- 一个便捷获取配置文件所在路径的命令：



```undefined
jupyter notebook --generate-config
```

- 注意： 这条命令虽然可以用于查看配置文件所在的路径，但主要用途是是否将这个路径下的配置文件**替换**为**默认配置文件**。
   如果你是第一次查询，那么**或许**不会出现下图的提示；若文件已经存在或被修改，使用这个命令之后会出现询问“Overwrite /Users/raxxie/.jupyter/jupyter_notebook_config.py with default config? [y/N]”，即“用默认配置文件覆盖此路径下的文件吗？”，如果按“y”，则完成覆盖，那么之前所做的修改都将失效；如果只是为了查询路径，那么一定要输入“N”。

![img](https:////upload-images.jianshu.io/upload_images/5101171-97d327bdfbd3e8fb?imageMogr2/auto-orient/strip|imageView2/2/w/719/format/webp)

命令

常规的情况下，Windows和Linux/macOS的配置文件所在路径和配置文件名如下所述：

- Windows系统的配置文件路径：`C:\Users\<user_name>\.jupyter\`
- Linux/macOS系统的配置文件路径：`/Users/<user_name>/.jupyter/` 或 `~/.jupyter/`
- 配置文件名：`jupyter_notebook_config.py`
- 注意：

① “<user_name>”为你的用户名。用户名两边不加尖括号“<>”。

② Windows和Linux/macOS系统的配置文件存放路径其实是相同的，只是系统不同，表现形式有所不同而已。

③ Windows和Linux/macOS系统的配置文件也是相同的。文件名以“.py”结尾，是Python的可执行文件。

④ 如果你不是通过一步到位的方式前往配置文件所在位置，而是一层一层进入文件夹/目录的，那么当你进入家目录后，用`ls`命令会发现找不到“.jupyter”文件夹/目录。这是因为凡是以“.”开头的目录都是隐藏文件，你可以通过`ls -a`命令查看当前位置下所有的隐藏文件。

#### ⑶ 修改配置文件

- Windows系统的用户可以使用文档编辑工具或IDE打开“jupyter_notebook_config.py”文件并进行编辑。常用的文档编辑工具和IDE有记事本、Notepad++、vim、Sublime Text、PyCharm等。其中，vim是没有图形界面的，是一款学习曲线较为陡峭的编辑器，其他工具在此不做使用说明，因为上手相对简单。通过vim修改配置文件的方法请继续往下阅读。
- Linux/macOS系统的用户建议直接通过终端调用vim来对配置文件进行修改。具体操作步骤如下：

##### ⒜ 打开配置文件

打开终端，输入命令：



```jsx
vim ~/.jupyter/jupyter_notebook_config.py
```

![img](https:////upload-images.jianshu.io/upload_images/5101171-e486b634051347bd?imageMogr2/auto-orient/strip|imageView2/2/w/811/format/webp)

vim打开配置文件

执行上述命令后便进入到配置文件当中了。

##### ⒝ 查找关键词

进入配置文件后查找关键词“c.NotebookApp.notebook_dir”。查找方法如下：

进入配置文件后不要按其他键，用**英文半角**直接输入`/c.NotebookApp.notebook_dir`，这时搜索的关键词已在文档中高亮显示了，按回车，光标从底部切换到文档正文中被查找关键词的首字母。

##### ⒞ 编辑配置文件

按**小写i**进入编辑模式，底部出现“--INSERT--”说明成功进入编辑模式。使用方向键把光标定位在第二个单引号上（光标定位在哪个字符，就在这个字符前开始输入），把“⑴ 创建文件夹/目录”步骤中复制的路径粘贴在此处。

##### ⒟ 取消注释

把该行行首的**井号（#）**删除。因为配置文件是Python的可执行文件，在Python中，井号（#）表示注释，即在编译过程中不会执行该行命令，所以为了使修改生效，需要删除井号（#）。

![img](https:////upload-images.jianshu.io/upload_images/5101171-797ace48ff698880?imageMogr2/auto-orient/strip|imageView2/2/w/922/format/webp)

config



##### ⒠ 保存配置文件

先按`ESC`键，从编辑模式退出，回到命令模式。

再用**英文半角**直接输入`:wq`，回车即成功保存且退出了配置文件。

注意：

- **冒号（:）** 一定要有，且也是**英文半角**。
- w：保存。
- q：退出。

##### ⒡ 验证

在终端中输入命令`jupyter notebook`打开Jupyter Notebook，此时你会看到一个清爽的界面，恭喜！

![img](https:////upload-images.jianshu.io/upload_images/5101171-1f4f722f5e5f354c?imageMogr2/auto-orient/strip|imageView2/2/w/1170/format/webp)

modified



##### ⒢ 注意

- 以上所有命令均以**英文半角**格式输入，若有报错，请严格检查这两个条件，**英文**且**半角**。
- 这里仅介绍了vim编辑器修改配置文件的方法，没有对vim编辑器的详细使用进行讲解，所以无需了解vim编辑器的具体使用方法，只需要按照上述步骤一定可以顺利完成修改！
- 推荐有时间和经历时学习一下vim编辑器的使用。这款强大的编辑器将会成为你未来工作中的利器。

# 四、Jupyter Notebook的基本使用

## 1. Files页面

![img](https:////upload-images.jianshu.io/upload_images/5101171-1ff2dee783b7ac11?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Files页面

此时你的界面当中应该还没有“Conda”和“Nbextensions”类目。不要着急，这两个类目将分别在“五、拓展功能”中的“[1.关联Jupyter Notebook和conda的环境和包——‘nb_conda’](#conda)”和“[2.Markdown生成目录](#nbextensions)”中安装。

Files页面是用于管理和创建文件相关的类目。

对于现有的文件，可以通过勾选文件的方式，对选中文件进行复制、重命名、移动、下载、查看、编辑和删除的操作。

同时，也可以根据需要，在“New”下拉列表中选择想要创建文件的环境，进行创建“ipynb”格式的笔记本、“txt”格式的文档、终端或文件夹。如果你创建的环境没有在下拉列表中显示，那么你需要依次前往“五、拓展功能”中的“[1.关联Jupyter Notebook和conda的环境和包——‘nb_conda’](#conda)”和“[六、增加内核——‘ipykernel’](#ipykernel)”中解决该问题。

### ① 笔记本的基本操作

![img](https:////upload-images.jianshu.io/upload_images/5101171-d0731dccc60e4f24?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

笔记本的使用

上图展示的是笔记本的基本结构和功能。根据图中的注解已经可以解决绝大多数的使用问题了！

工具栏的使用如图中的注解一样直观，在此不过多解释。需要特别说明的是“单元格的状态”，有Code，Markdown，Heading，Raw NBconvert。其中，最常用的是前两个，分别是代码状态，Markdown编写状态。Jupyter Notebook已经取消了Heading状态，即标题单元格。取而代之的是Markdown的一级至六级标题。而Raw NBconvert目前极少用到，此处也不做过多讲解。

菜单栏涵盖了笔记本的所有功能，即便是工具栏的功能，也都可以在菜单栏的类目里找到。然而，并不是所有功能都是常用的，比如Widgets，Navigate。Kernel类目的使用，主要是对内核的操作，比如中断、重启、连接、关闭、切换内核等，由于我们在创建笔记本时已经选择了内核，因此切换内核的操作便于我们在使用笔记本时切换到我们想要的内核环境中去。由于其他的功能相对比较常规，根据图中的注解来尝试使用笔记本的功能已经非常便捷，因此不再做详细讲解。

### ② 笔记本重命名的两种方式

#### ⑴  笔记本内部重命名

在使用笔记本时，可以直接在其内部进行重命名。在左上方“Jupyter”的图标旁有程序默认的标题“Untitled”，点击“Untitled”然后在弹出的对话框中输入自拟的标题，点击“Rename”即完成了重命名。

#### ⑵  笔记本外部重命名

若在使用笔记本时忘记了重命名，且已经保存并退出至“Files”界面，则在“Files”界面勾选需要重命名的文件，点击“Rename”然后直接输入自拟的标题即可。

#### ⑶ 演示

![img](https:////upload-images.jianshu.io/upload_images/5101171-ebc5c06becc04dfe?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

重命名

## 2. Running页面

Running页面主要展示的是当前正在运行当中的终端和“ipynb”格式的笔记本。若想要关闭已经打开的终端和“ipynb”格式的笔记本，仅仅关闭其页面是无法彻底退出程序的，需要在Running页面点击其对应的“Shutdown”。更多关闭方法可以查阅“八、关闭和退出”中的“[1.关闭笔记本和终端](#quit)”

![img](https:////upload-images.jianshu.io/upload_images/5101171-d129a2a91f87e6e8?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Running

## 3. Clusters页面

> Clusters tab is now provided by IPython parallel. See '[IPython parallel](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fipython%2Fipyparallel)' for installation details.

Clusters类目现在已由IPython parallel对接，且由于现阶段使用频率较低，因此在此不做详细说明，想要了解更多可以访问[IPython parallel的官方网站](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fipython%2Fipyparallel)。

## 4. Conda页面

Conda页面主要是Jupyter Notebook与Conda关联之后对Conda环境和包进行直接操作和管理的页面工具。详细信息请直接查阅“五、拓展功能”中的“[1.关联Jupyter Notebook和conda的环境和包——‘nb_conda’](#conda)”。这是目前使用Jupyter Notebook的必备环节，因此请务必查阅。

## 5. Nbextensions页面

![img](https:////upload-images.jianshu.io/upload_images/5101171-e53ab207a96c980c?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

nbextensions

Nbextensions页面提供了多个Jupyter Notebook的插件，使其功能更加强大。该页面中主要使用的插件有nb_conda，nb_present，Table of Contents(2)。这些功能我们无需完全掌握，也无需安装所有的扩展功能，根据本文档提供的学习思路，我们只需要安装Talbe of Contents(2)即可，该功能可为Markdown文档提供目录导航，便于我们编写文档。该安装指导请查阅“五、拓展功能”中的“[2.Markdown生成目录](#nbextensions)”。

# 五、拓展功能

<a id=conda></a>

## 1. 关联Jupyter Notebook和conda的环境和包——“nb_conda”☆

### ① 安装



```undefined
conda install nb_conda
```

执行上述命令能够将你conda创建的环境与Jupyter Notebook相关联，便于你在Jupyter Notebook的使用中，在不同的环境下创建笔记本进行工作。

### ② 使用

- 可以在Conda类目下对conda环境和包进行一系列操作。

  ![img](https:////upload-images.jianshu.io/upload_images/5101171-80f141edb2bac9d5?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  conda类目

- 可以在笔记本内的“Kernel”类目里的“Change kernel”切换内核。

  ![img](https:////upload-images.jianshu.io/upload_images/5101171-2cb5c4ec387ca814?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  切换内核

### ③ 卸载



```csharp
canda remove nb_conda
```

执行上述命令即可卸载nb_conda包。

<a id=nbextensions></a>

## 2. Markdown生成目录

- 不同于有道云笔记的Markdown编译器，Jupyter Notebook无法为Markdown文档通过特定语法添加目录，因此需要通过安装扩展来实现目录的添加。



```swift
conda install -c conda-forge jupyter_contrib_nbextensions
```

- 执行上述命令后，启动Jupyter Notebook，你会发现导航栏多了“Nbextensions”的类目，点击“Nbextensions”，勾选“Table of Contents ⑵”

  ![img](https:////upload-images.jianshu.io/upload_images/5101171-1d2c050b8d54fdb0?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  nbextensions

- 之后再在Jupyter Notebook中使用Markdown，点击下图的图标即可使用啦。

  ![img](https:////upload-images.jianshu.io/upload_images/5101171-5871d68688547f5e?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  添加目录

## 3. Markdown在文中设置链接并定位

在使用Markdown编辑文档时，难免会遇到需要在文中设定链接，定位在文档中的其他位置便于查看。因为Markdown可以完美的兼容html语法，因此这种功能可以通过html语法当中“a标签”的索引用法来实现。

语法格式如下：



```xml
[添加链接的正文](#自定义索引词)
<a id=自定义索引词>跳转提示</a>
```

- 注意：

  1. 语法格式当中所有的符号均是**英文半角**。
  2. “自定义索引词”最好是英文，较长的词可以用下划线连接。
  3. “a标签”出现在想要被跳转到的文章位置，html标签除了单标签外均要符合“有头（`<a>`）必有尾（`</a>`）”的原则。头尾之间的“跳转提示”是可有可无的。
  4. “a标签”中的“id”值即是为正文中添加链接时设定的“自定义索引值”，这里通过“id”的值实现从正文的链接跳转至指定位置的功能。

- 例：

  1. 有跳转提示语

     ![img](https:////upload-images.jianshu.io/upload_images/5101171-c958912184ce3d2a?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

     有提示语

  2. 无跳转提示语

     ![img]()

     无提示语

## 4. 加载指定网页源代码

### ① 使用场景

想要在Jupyter Notebook中直接加载指定网站的源代码到笔记本中。

### ② 方法

执行以下命令:



```undefined
%load URL
```

其中，URL为指定网站的地址。

### ③ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-8fe1e7f4d433006a?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

加载指定网站源代码

## 5. 加载本地Python文件

### ① 使用场景

想在Jupyter Notebook中加载本地的Python文件并执行文件代码。

### ② 方法

执行以下命令：



```undefined
%load Python文件的绝对路径
```

### ③ 注意

1. Python文件的后缀为“.py”。
2. “%load”后跟的是Python文件的**绝对路径**。
3. 输入命令后，可以按`CTRL 回车`来执行命令。第一次执行，是将本地的Python文件内容加载到单元格内。此时，Jupyter Notebook会自动将“%load”命令注释掉（即在前边加井号“#”），以便在执行已加载的文件代码时不重复执行该命令；第二次执行，则是执行已加载文件的代码。

### ④ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-5b6ad15ba1a1f29b?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

加载本地Python文件

## 6. 直接运行本地Python文件

### ① 使用场景

不想在Jupyter Notebook的单元格中加载本地Python文件，想要直接运行。

### ② 方法

执行命令：



```undefined
%run Python文件的绝对路径
```

或



```undefined
!python3 Python文件的绝对路径
```

或



```undefined
!python Python文件的绝对路径
```

### ③ 注意

1. Python文件的后缀为“.py”。
2. “%run”后跟的是Python文件的**绝对路径**。
3. “!python3”用于执行Python 3.x版本的代码。
4. “!python”用于执行Python 2.x版本的代码。
5. “!python3”和“!python”属于 `!shell命令` 语法的使用，即在Jupyter Notebook中执行shell命令的语法。
6. 输入命令后，可以按 `CTRL 回车` 来执行命令，执行过程中将不显示本地Python文件的内容，直接显示运行结果。

### ④ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-a185ba9fdac862ba?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

直接运行Python文件

## 7. 在Jupyter Notebook中获取当前位置

### ① 使用场景

想要在Jupyter Notebook中获取当前所在位置的**绝对路径**。

### ② 方法



```bash
%pwd
```

或



```bash
!pwd
```

### ③ 注意

1. 获取的位置是当前Jupyter Notebook中创建的笔记本所在位置，且该位置为**绝对路径**。
2. “!pwd”属于 `!shell命令` 语法的使用，即在Jupyter Notebook中执行shell命令的语法。

### ④ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-570307c5bacab4ea?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

获取当前路径

## 8. 在Jupyter Notebook使用shell命令

### ① 方法一——在笔记本的单元格中

#### ⑴ 语法



```undefined
!shell命令
```

- 在Jupyter Notebook中的笔记本单元格中用英文感叹号“!”后接shell命令即可执行shell命令。

#### ⑵ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-75788ac3d5029d7d?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

shell命令

### ② 方法二——在Jupyter Notebook中新建终端

#### ⑴ 启动方法

在Jupyter Notebook主界面，即“File”界面中点击“New”；在“New”下拉框中点击“Terminal”即新建了终端。此时终端位置是在你的家目录，可以通过`pwd`命令查询当前所在位置的绝对路径。

#### ⑵ 关闭方法

在Jupyter Notebook的“Running”界面中的“Terminals”类目中可以看到正在运行的终端，点击后边的“Shutdown”即可关闭终端。

#### ⑶ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-6be67157a70cb517?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

终端

## 9. 隐藏笔记本输入单元格

### ① 使用场景

在Jupyter Notebook的笔记本中无论是编写文档还是编程，都有输入（In []）和输出（Out []）。当我们编写的代码或文档使用的单元格较多时，有时我们只想关注输出的内容而暂时不看输入的内容，这时就需要隐藏输入单元格而只显示输出单元格。

### ② 方法一

#### ⑴ 代码



```python
from IPython.display import display
from IPython.display import HTML
import IPython.core.display as di # Example: di.display_html('<h3>%s:</h3>' % str, raw=True)

# 这行代码的作用是：当文档作为HTML格式输出时，将会默认隐藏输入单元格。
di.display_html('<script>jQuery(function() {if (jQuery("body.notebook_app").length == 0) { jQuery(".input_area").toggle(); jQuery(".prompt").toggle();}});</script>', raw=True)

# 这行代码将会添加“Toggle code”按钮来切换“隐藏/显示”输入单元格。
di.display_html('''<button onclick="jQuery('.input_area').toggle(); jQuery('.prompt').toggle();">Toggle code</button>''', raw=True)
```

在笔记本第一个单元格中输入以上代码，然后执行，即可在该文档中使用“隐藏/显示”输入单元格功能。

- 缺陷：此方法不能很好的适用于Markdown单元格。

#### ⑵ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-0988fe0b394e7604?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

隐藏/显示方法一

### ③ 方法二

#### ⑴ 代码



```xml
from IPython.display import HTML

HTML('''<script>
code_show=true; 
function code_toggle() {
 if (code_show){
 $('div.input').hide();
 } else {
 $('div.input').show();
 }
 code_show = !code_show
} 
$( document ).ready(code_toggle);
</script>
<form action="javascript:code_toggle()"><input type="submit" value="Click here to toggle on/off the raw code."></form>''')
```

在笔记本第一个单元格中输入以上代码，然后执行，即可在该文档中使用“隐藏/显示”输入单元格功能。

- 缺陷：此方法不能很好的适用于Markdown单元格。

#### ⑵ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-900f734f7e1d7095?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

隐藏/显示方法二

## 10. 魔术命令

由于目前暂时用不到过多的魔术命令，因此暂时先参考[官网的文档](https://link.jianshu.com?t=http%3A%2F%2Fipython.readthedocs.io%2Fen%2Fstable%2Finteractive%2Fmagics.html)。

<a id=ipykernel></a>

# 六、增加内核——“ipykernel” ☆

## 1. 使用场景

1. 场景一：同时用不同版本的Python进行工作，在Jupyter Notebook中无法切换，即“New”的下拉菜单中无法使用需要的环境。
2. 场景二：创建了不同的虚拟环境（或许具有相同的Python版本但安装的包不同），在Jupyter Notebook中无法切换，即“New”的下拉菜单中无法使用需要的环境。

接下来将分别用“命令行模式”和“图形界面模式”来解决以上两个场景的问题。顾名思义，“命令行模式”即在终端中通过执行命令来一步步解决问题；“图形界面模式”则是通过在Jupyter Notebook的网页中通过鼠标点击的方式解决上述问题。

其中，“图形界面模式”的解决方法相对比较简单快捷，如果对于急于解决问题，不需要知道运行原理的朋友，可以直接进入“[3. 解决方法之图形界面模式](#gui)”来阅读。

“命令行模式”看似比较复杂，且又划分了使用场景，但通过这种方式来解决问题可以更好的了解其中的工作原理，比如，每进行一步操作对应的命令是什么，而命令的执行是为了达到什么样的目的，这些可能都被封装在图形界面上的一个点击动作来完成了。对于想更深入了解其运作过程的朋友，可以接着向下阅读。

## 2. 解决方法之命令行模式

### ① 同时使用不同版本的Python

#### ⑴ 在Python 3中创建Python 2内核

##### ⒜ pip安装

- 首先安装Python 2的ipykernel包。



```undefined
python2 -m pip install ipykernel
```

- 再为**当前用户**安装Python 2的内核（ipykernel）。



```undefined
python2 -m ipykernel install --user
```

- 注意：“--user”参数的意思是针对当前用户安装，而非系统范围内安装。

##### ⒝ conda安装

- 首先创建Python版本为2.x且具有ipykernel的新环境，其中“<env_name>”为自定义环境名，环境名两边不加尖括号“<>”。



```xml
conda create -n <env_name> python=2 ipykernel
```

- 然后切换至新创建的环境。



```xml
Windows: activate <env_name>
Linux/macOS: source activate <env_name>
```

- 为**当前用户**安装Python 2的内核（ipykernel）。



```undefined
python2 -m ipykernel install --user
```

- 注意：“--user”参数的意思是针对当前用户安装，而非系统范围内安装。

#### ⑵ 在Python 2中创建Python 3内核

##### ⒜ pip安装

- 首先安装Python 3的ipykernel包。



```undefined
python3 -m pip install ipykernel
```

- 再为**当前用户**安装Python 2的内核（ipykernel）。



```undefined
python3 -m ipykernel install --user
```

- 注意：“--user”参数的意思是针对当前用户安装，而非系统范围内安装。

##### ⒝ conda安装

- 首先创建Python版本为3.x且具有ipykernel的新环境，其中“<env_name>”为自定义环境名，环境名两边不加尖括号“<>”。



```xml
conda create -n <env_name> python=3 ipykernel
```

- 然后切换至新创建的环境。



```xml
Windows: activate <env_name>
Linux/macOS: source activate <env_name>
```

- 为**当前用户**安装Python 3的内核（ipykernel）。



```undefined
python3 -m ipykernel install --user
```

- 注意：“--user”参数的意思是针对当前用户安装，而非系统范围内安装。

### ② 为不同环境创建内核

#### ⑴ 切换至需安装内核的环境



```xml
Windows: activate <env_name>
Linux/macOS: source activate <env_name>
```

- 注意：“<env_name>”是需要安装内核的环境名称，环境名两边不加尖括号“<>”。

#### ⑵ 检查该环境是否安装了ipykernel包



```cpp
conda list
```

执行上述命令查看当前环境下安装的包，若没有安装ipykernel包，则执行安装命令；否则进行下一步。



```undefined
conda install ipykernel
```

#### ⑶ 为当前环境下的当前用户安装Python内核

- 若该环境的Python版本为2.x，则执行命令：



```xml
python2 -m ipykernel install --user --name <env_name> --display-name "<notebook_name>"
```

- 若该环境的Python版本为3.x，则执行命令：



```xml
python3 -m ipykernel install --user --name <env_name> --display-name "<notebook_name>"
```

- 注意:
  1. “<env_name>”为当前环境的环境名称。环境名两边不加尖括号“<>”。
  2. “<notebook_name>”为自定义显示在Jupyter Notebook中的名称。名称两边不加尖括号“<>”，但**双引号必须加**。
  3. “--name”参数的值，即“<env_name>”是Jupyter内部使用的，其目录的存放路径为`~/Library/Jupyter/kernels/`。如果定义的名称在该路径已经存在，那么将自动覆盖该名称目录的内容。
  4. “--display-name”参数的值是显示在Jupyter Notebook的菜单中的名称。

#### ⑷ 检验

使用命令`jupyter notebook`启动Jupyter Notebook；在“Files”下的“New”下拉框中即可找到你在第⑶步中的自定义名称，此时，你便可以尽情地在Jupyter Notebook中切换环境，在不同的环境中创建笔记本进行工作和学习啦！

<a id=gui></a>

## 3. 解决方法之图形界面模式

① 你创建了一个新的环境，但却发现在Jupyter Notebook的“New”中找不到这个环境，无法在该环境中创建笔记本。



![img](https:////upload-images.jianshu.io/upload_images/5101171-cfe83caf4c637f86?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

图形界面_问题

② 进入Jupyter Notebook → Conda → 在“Conda environment”中点击你要添加ipykernel包的环境 → 左下方搜索框输入“ipykernel” → 勾选“ipykernel” → 点击搜索框旁的“→”箭头 → 安装完毕 → 右下方框内找到“ipykernel”说明已经安装成功。

![img](https:////upload-images.jianshu.io/upload_images/5101171-1044c7dafd64c290?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

图形界面_解决

③ 在终端`CTRL C`关闭Jupyter Notebook的服务器然后重启Jupyter Notebook，在“File”的“New”的下拉列表里就可以找到你的环境啦。

![img](https:////upload-images.jianshu.io/upload_images/5101171-431fd2f7fd36c123?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

图形界面_验证

# 七、Jupyter Notebook快捷键

## 1. Mac与Windows特殊按键对照表

| 符号 |  Mac按键   | Windows按键 |
| :--: | :--------: | :---------: |
|  ⌘   |  command   |     无      |
|  ⌃   |  control   |    ctrl     |
|  ⌥   |   option   |     alt     |
|  ⇧   |   shift    |    shift    |
|  ↩   |   return   |   return    |
|  ␣   |   space    |    space    |
|  ⇥   |    tab     |     tab     |
|  ⌫   |   delete   |  backspace  |
|  ⌦   | fn  delete |   delete    |
|  -   |     -      |      -      |

## 2. Jupyter Notebook笔记本的两种模式

### ① 命令模式

- 命令模式将键盘命令与Jupyter Notebook笔记本命令相结合，可以通过键盘不同键的组合运行笔记本的命令。
- 按`esc`键进入命令模式。
- 命令模式下，单元格边框为灰色，且左侧边框线为蓝色粗线条。

![img](https:////upload-images.jianshu.io/upload_images/5101171-fa250ffc77636245?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

命令模式

### ② 编辑模式

- 编辑模式使用户可以在单元格内编辑代码或文档。
- 按`enter`或`return`键进入编辑模式。
- 编辑模式下，单元格边框和左侧边框线均为绿色。

![img](https:////upload-images.jianshu.io/upload_images/5101171-7bb9c93d888e9f82?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

编辑模式

## 3. 两种模式的快捷键

### ① 命令模式

| 快捷键 | 用途                                             |
| :----: | ------------------------------------------------ |
|   F    | 查找和替换                                       |
|   ↩    | 进入编辑模式                                     |
|  ⌘⇧F   | 打开命令选项板                                   |
|  ⌘⇧P   | 打开命令选项板                                   |
|   P    | 打开命令选项板                                   |
|   ⇧↩   | 运行当前单元格并选中下一个单元格                 |
|   ⌃↩   | 运行选中单元格                                   |
|   ⌥↩   | 运行当前单元格并且在下方插入单元格               |
|   Y    | 将单元格切换至code状态                           |
|   M    | 将单元格切换至markdown状态                       |
|   R    | 将单元格切换至raw状态                            |
|   1    | 将单元格设定一级标题                             |
|   2    | 将单元格设定二级标题                             |
|   3    | 将单元格设定三级标题                             |
|   4    | 将单元格设定四级标题                             |
|   5    | 将单元格设定五级标题                             |
|   6    | 将单元格设定六级标题                             |
|   ↑    | 选中上方单元格                                   |
|   K    | 选中上方单元格                                   |
|   ↓    | 选中下方单元格                                   |
|   J    | 选中下方单元格                                   |
|   ⇧K   | 向上多选单元格                                   |
|   ⇧↑   | 向上多选单元格                                   |
|   ⇧J   | 向下多选单元格                                   |
|   ⇧↓   | 向下多选单元格                                   |
|   A    | 在上方插入单元格                                 |
|   B    | 在下方插入单元格                                 |
|   X    | 剪切选中单元格                                   |
|   C    | 复制选中单元格                                   |
|   ⇧V   | 粘贴到上方单元格                                 |
|   V    | 粘贴到下方单元格                                 |
|   Z    | 撤销删除                                         |
|  D, D  | 删除选中单元格                                   |
|   ⇧M   | 合并选中单元格，若直选中一个则与下一个单元格合并 |
|   ⌘S   | 保存                                             |
|   S    | 保存                                             |
|   L    | 转换行号                                         |
|   O    | 转换输出                                         |
|   ⇧O   | 转换滚动输出                                     |
|   H    | 显示快捷键帮助                                   |
|  I, I  | 中断Notebook内核                                 |
|  O, O  | 重启Notebook内核                                 |
|  esc   | 关闭页面                                         |
|   Q    | 关闭页面                                         |
|   ⇧L   | 转换所有单元格行号且设置持续有效                 |
|   ⇧␣   | 向上滚动                                         |
|   ␣    | 向下滚动                                         |

### ② 编辑模式

| Mac快捷键 | Windows快捷键 | 用途                               |
| :-------: | :-----------: | ---------------------------------- |
|     ⇥     |       ⇥       | 代码补全或缩进                     |
|    ⇧⇥     |      ⇧⇥       | 提示                               |
|    ⌘]     |      ⌃]       | 向后缩进                           |
|    ⌘[     |      ⌃[       | 向前缩进                           |
|    ⌘A     |      ⌃A       | 全选                               |
|    ⌘Z     |      ⌃Z       | 撤销                               |
|    ⌘/     |               | 注释                               |
|    ⌘D     |               | 删除该行内容                       |
|    ⌘U     |               | 撤销                               |
|    ⌘↑     |      ⌃↑       | 光标跳转至单元格起始位置           |
|    ⌘↓     |      ⌃↓       | 光标跳转至单元格最终位置           |
|    ⌥←     |      ⌃←       | 光标位置左移一个单词               |
|    ⌥→     |      ⌃→       | 光标位置右移一个单词               |
|    ⌥⌫     |      ⌃⌫       | 删除前边一个单词                   |
|    ⌥⌦     |      ⌃⌦       | 删除后边一个单词                   |
|    ⌘⇧Z    |      ⌃Y       | 重做                               |
|    ⌘⇧U    |      ⌃⇧Z      | 重做                               |
|    ⌘⌫     |      ⌃⌫       | 删除该行光标左边内容               |
|    ⌘⌦     |      ⌃⌦       | 删除该行光标右边内容               |
|    ⌃M     |      ⌃M       | 进入命令模式                       |
|    esc    |      esc      | 进入命令模式                       |
|    ⌘⇧F    |               | 打开命令选项板                     |
|    ⌘⇧P    |               | 打开命令选项板                     |
|    ⇧↩     |      ⇧↩       | 运行当前单元格并选中下一个单元格   |
|    ⌃↩     |      ⌃↩       | 运行选中单元格                     |
|    ⌥↩     |      ⌥↩       | 运行当前单元格并且在下方插入单元格 |
|    ⌃⇧-    |      ⌃⇧-      | 以光标所在位置分割单元格           |
|    ⌘S     |      ⌃S       | 保存                               |
|     ↓     |       ↓       | 下移光标                           |
|     ↑     |       ↑       | 上移光标                           |

## 4. 查看和编辑快捷键

### ① 查看快捷键

① 进入Jupyter Notebook主界面“File”中。

② 在“New”的下拉列表中选择环境创建一个笔记本。

③ 点击“Help”。

④ 点击“Keyboard Shortcuts”。

### ② 编辑快捷键

#### ⑴ 方法一

① 进入Jupyter Notebook主界面“File”中。

② 在“New”的下拉列表中选择环境创建一个笔记本。

③ 点击“Help”。

④ 点击“Keyboard Shortcuts”。

⑤ 弹出的对话框中“Command Mode (press Esc to enable)”旁点击“Edit Shortcuts”按钮。

#### ⑵ 方法二

① 进入Jupyter Notebook主界面“File”中。

② 在“New”的下拉列表中选择环境创建一个笔记本。

③ 点击“Help”。

④ 点击“Edit Keyboard Shortcuts”。

### ③ 例

![img](https:////upload-images.jianshu.io/upload_images/5101171-3b9eb407f463e3bd?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

查看和编辑快捷键

# 八、关闭和退出

<a id=quit></a>

## 1. 关闭笔记本和终端

当我们在Jupyter Notebook中创建了终端或笔记本时，将会弹出新的窗口来运行终端或笔记本。当我们使用完毕想要退出终端或笔记本时，仅仅**关闭页面**是无法结束程序运行的，因此我们需要通过以下步骤将其完全关闭。

### ① 方法一

⑴ 进入“Files”页面。

⑵ 勾选想要关闭的“ipynb”笔记本。正在运行的笔记本其图标为绿色，且后边标有“Running”的字样；已经关闭的笔记本其图标为灰色。

⑶ 点击上方的黄色的“Shutdown”按钮。

⑷ 成功关闭笔记本。

- 注意：此方法只能关闭笔记本，无法关闭终端。

### ② 方法二

⑴ 进入“Running”页面。

⑵ 第一栏是“Terminals”，即所有正在运行的终端均会在此显示；第二栏是“Notebooks”，即所有正在运行的“ipynb”笔记本均会在此显示。

⑶ 点击想要关闭的终端或笔记本后黄色“Shutdown”按钮。

⑷ 成功关闭终端或笔记本。

- 注意：此方法可以关闭任何正在运行的终端和笔记本。

### ③ 注意

⑴ 只有“ipynb”笔记本和终端需要通过上述方法才能使其结束运行。

⑵ “txt”文档，即“New”下拉列表中的“Text File”，以及“Folder”只要关闭程序运行的页面即结束运行，无需通过上述步骤关闭。

### ④ 演示

![img](https:////upload-images.jianshu.io/upload_images/5101171-a17e58d46b1d30ca?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

关闭笔记本和终端

## 2. 退出Jupyter Notebook程序

如果你想退出Jupyter Notebook程序，仅仅通过关闭网页是无法退出的，因为当你打开Jupyter Notebook时，其实是启动了它的服务器。

你可以尝试关闭页面，并打开新的浏览器页面，把之前的地址输进地址栏，然后跳转页面，你会发现再次进入了刚才“关闭”的Jupyter Notebook页面。

如果你忘记了刚才关闭的页面地址，可以在启动Jupyter Notebook的终端中找到地址，复制并粘贴至新的浏览器页面的地址栏，会发现同样能够进入刚才关闭的页面。

因此，想要彻底退出Jupyter Notebook，需要关闭它的服务器。只需要在它启动的终端上按：

- Mac用户：`control c`
- Windows用户：`ctrl c`

然后在终端上会提示：“Shutdown this notebook server (y/[n])?”输入`y`即可关闭服务器，这才是彻底退出了Jupyter Notebook程序。此时，如果你想要通过输入刚才关闭网页的网址进行访问Jupyter Notebook便会看到报错页面。

