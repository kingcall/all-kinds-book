PyCharm下载地址：*https://www.jetbrains.com/pycharm/download/*

PyCharm社区版功能基本够用，但是作为傲娇的程序员，咱都是上来就专业版，然后各种破解使用，自由的享受编程带来的快乐。这里介绍一下行之有效的激活方法。

# 永久激活

这里主要介绍永久激活的方式，永久激活后，就可以放心使用了，一劳永逸，5分钟就能完成。

# 1 插件下载

PyCharm永久激活需要下载一个插件：`jetbrains-agent.jar`，插件下载地址，关注公众号【Java大数据与数据仓库】，回复关键字：jetbrains，即可获取。是个`.jar`文件，基于`java`的。
将`jetbrains-agent.jar`破解文件放到PyCharm安装目录bin下面，比如我的：`D:\Program Files\JetBrains\PyCharm 2019.1.2\bin`这个目录。

# 2 创建项目

如果你是刚下载的`PyCharm`，则需要点击激活窗口的`“Evaluate for free”`免费试用，然后再点击`Create New Project`创建一个空项目，这样就可以进入到`PyCharm`的工作页面。

# 3 修改配置文件

点击Pycharm最上面的菜单栏中的 `"Help" -> "Edit Custom VM Options ..."`，如果提示是否要创建文件，请点`"Yes"`。
在打开的vmoptions编辑窗口末行添加：`-javaagent:D:\Program Files\JetBrains\PyCharm 2019.1.2\bin\jetbrains-agent.jar`![图片](https://mmbiz.qpic.cn/mmbiz_png/cAoClgx53VuoP8O6QClmFZvCgyY0NLoChsgD7mYsfQM5TL4WCR9usr3NSa35EAAHtazeTFmd3p9Kt15LqeHgow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)请仔细检查补丁路径是否正确，不正确的话会启动PyCharm失败。**重要的事情说3遍**：

- 修改完配置文件之后重启`Pycharm`
- 修改完配置文件之后重启`Pycharm`
- 修改完配置文件之后重启`Pycharm`



如果错误则会出现`PyCharm`打不开的情况，这时候可以删除用户配置目录下的`PyCharm`文件夹：`C:\Users\26015\.PyCharm2019.1`

# 4 输入激活码

重启PyCharm之后，点击菜单栏中的 `“Help” -> “Register …”`，这里有两种激活方式：
选择最后一种`License server`激活方式，地址填入：`http://jetbrains-license-server` （应该会自动填上），或者点击按钮：`”Discover Server”`来自动填充地址，完成激活。

# 5 查看有效期

当你激活完毕后，`PyCharm`右下角会有个`Registration`小长条提示框，大致的内容为：`You copy is Licensed to userName`

![图片](https://mmbiz.qpic.cn/mmbiz_png/cAoClgx53VuoP8O6QClmFZvCgyY0NLoCAibcV7e5A78WtYoDKGNQ9UDWRvNpJsI5RCeg1knbszicfqzTHsHSCf4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



> `jetbrains-agent.jar`插件下载，关注公众号【Java大数据与数据仓库】，回复关键字：jetbrains，即可获取