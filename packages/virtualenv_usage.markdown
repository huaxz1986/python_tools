# VirtualEnv
VirtualEnv用于创建独立的Python虚拟环境，这些独立的Python虚拟环境中有独立的安装目录。

## 一、安装VirtualEnv
通过`pip install virtualenv`命令可以安装VirtualEnv

## 二、创建独立的Python虚拟环境
创建独立的Python虚拟环境命令为：

	virtualenv ENV

>如果你使用`virtualenv --system-site-packages ENV`命令创建Python虚拟环境，则该虚拟环境会继承来自`global site-packages`中的所有 packages.

其中`ENV`是新的虚拟环境的目录，该命令做了下面的事情：

* 创建`ENV/lib/`目录以及`ENV/include/`目录，它们包含用于支持Python虚拟环境所必须的
  library files。虚拟环境中安装的所有 package都位于 
  `ENV/lib/pythonX.X/site-packages/` 目录下
* 
	* 在Unix/OSX系统上创建`ENV/bin/`目录，其中存放着Python虚拟环境所必须的可执行文件以及脚本。
	* 在Windows系统上创建`ENV\Scripts\`目录，其中存放着Python虚拟环境所必须的可执行文件以及脚本。
* 安装关键的 pip 以及 setuptools 这两个 package。`ENV/bin/pip` 是与当前虚拟环境中相关的 pip

## 三、 激活Python虚拟环境
在新创建的Python虚拟环境中，有一个激活脚本。

* 在 Posix系统上，该激活脚本位于`ENV/bin/activate`
* 在 Windows系统中，激活脚本位于`ENV\Scripts\activate。bat`

激活Python虚拟环境命令为：	

* 在 Posix 系统中，执行`source ENV/bin/activate`命令
* 在 Windows 系统中，在CMD中执行`ENV\Scripts\activate.bat`脚本

>该脚本本质上是修改`$PATH`环境变量，使得环境变量的第一项为`ENV/bin/`目录。
>同时修改你的SHELL命令提示符，从而提示你这是在一个Python虚拟环境中
>
>你也可以不使用该激活脚本，而是直接调用`bin/`目录中的脚本和python解释器(Unix/OSX环境下)
>，如`ENV/bin/pip`以及 `ENV/bin/python-script.py`

## 四、退出虚拟环境
退出虚拟环境，在shell中键入`deactivate`命令即可

## 五、移除Python虚拟环境
移除之前首先退出Python虚拟环境，然后删除ENV文件夹即可
