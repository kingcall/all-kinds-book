- anaconda: 除了Python基本包之外，还自带了多种常用的科学计算Python包
- miniconda：只有Python基本包

## 配置镜像
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

## conda
- conda install numpy=x.x.x 指定版本安装
- conda uninstall numpy 卸载
- conda upgrade --all 更新所有
- conda upgrade numpy 更新指定包

### 创建虚拟环境
创建一个名为test的虚拟环境
conda create -n test

> 创建指定版本的虚拟环境，即便是conda3也可以创建2.x的，反之亦然
conda create -n test python=2.7

> 完整复制某个环境
conda create -n test2 -clone test

> 进入虚拟环境
source activate test

> 退出虚拟环境
source deactiavet
