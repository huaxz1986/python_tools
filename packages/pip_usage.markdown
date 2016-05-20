# pip使用指南（基于pip 8.1.1）

当你使用Python2.7.9以及Python3.4以上的Python版本时，pip已经默认随Python一起安装，但是你可能需要不定期的更新pip。

## 一、更新pip

* 在Linux或者OSX平台上执行命令 `pip install -U pip`
* 在Windows平台上执行命令`python -m pip install -U pip`

  ![Windows下更新pip](./imgs/pip/update.JPG)

## 二、使用pip

### 1.安装package
#### a. 从PyPI安装

pip安装package最常见的就是从PyPI安装，从 [PyPI](http://pypi.python.org/pypi/) 安装package的命令为：

	pip install SomePackage  #安装最新的版本
	pip install SomePackage==1.0.4  #安装指定版本
	pip install 'SomePackage>=1.0.4' #minimum版本

  ![安装package](./imgs/pip/install_package.JPG)

>从PyPI用pip安装时，可能因为网络问题发生超时。此时可以指定国内镜像源，命令为：
>`pip install SomePackage -i http://example.com`
>常用的镜像源有：

>* 清华：http://mirrors.tuna.tsinghua.edu.cn/pypi/simple
>* 豆瓣：http://pypi.douban.com/simple 
	
>如果不希望每次运行pip命令时均手工添加，可以在配置文件中添加
>
>		[global]
>		index-url=http://example.com
>
	
#### b. 从Wheel安装
Whell文件时一个打包的构建好的文件格式，比从源代码build然后install快得多。安装命令：

	pip install SomePackage-1.0-py2.py3-none-any.whl

#### c. 从本地源代码安装
通常源代码中都有`setup.py`文件，从源代码安装的命令为：

	python setup.py install

### 2.查看
#### a. 查看某个已安装package
查看某个package的详细信息命令为：

	pip show SomePackage #查看该package详细信息
	pip show --files SomePackage #查看该package已安装哪些文件

  ![查看已安装package详细信息](./imgs/pip/show_package.JPG)

  ![查看已安装package安装文件](./imgs/pip/show_package_detail.JPG)

#### b. 查看所有已经安装package
查看已经安装了哪些package的命令为：

	pip list #列出所有已安装的package
	pip list --outdated #列出所有outdated package

  ![查看已安装package list](./imgs/pip/list_package.JPG)

  ![查看已安装但是outdated的package list](./imgs/pip/list_outdatedpackage.JPG)


### 3.更新package
更新package命令为：

	pip install --upgrade SomePackage

  ![更新 package](./imgs/pip/update_package.JPG)


### 4.卸载package

卸载package的命令为：

	pip uninstall SomePackage

  ![卸载 package](./imgs/pip/uninstall_package.JPG)

### 5.寻找package
寻找package的命令为：

	pip search "query"

  ![寻找 package](./imgs/pip/search_package.JPG)

### 6. pip freeze
pip freeze命令是输出当前已安装package的依赖列表，方便于复制当前已安装的package。命令为：

		pip freeze

  ![pip freeze](./imgs/pip/freeze_package.JPG)

#### a. 生成requirement文件
可以通过重定向输出从而产生requirement文件，命令为： `pip freeze > requirements.txt`

你也可以从某个`requirements`文件中读取从而叠加到当前的`pip freeze`结果中：
 `pip freeze -r exist_requirements.txt > requirements.txt`

#### b.不输出全局的包列表
如果你是在`virtualenv`环境中，则通过 `pip freeze -l `命令则不会读取`globally-installed`包（即不会读系统取`site-packages`目录中的包）

如果你是使用`pip freeze --user`命令，则只会读取`user-site`目录中的包

#### c.安装`requirements`文件中的包

如果有了`requirements`文件，则通过`pip install -r requirements.txt`命令可以安装所有的这些包
 
## 三、配置文件
配置文件有三个级别：系统级别、用户级别、virtualenv级别。其读取顺序为：首先读取系统级别的配置文件，然后读取用户级别的配置文件，最后读取virtualenv级别的配置文件。

如果同样的值在多个配置文件中设置，则最后读取的值会覆盖早期读取的值。

### 1.系统级的配置文件
系统级的配置文件位于：

* Unix系统中位于：`/etc/pip.conf`。也可以位于由`XDG_CONFIG_DIRS`环境变量（若存在的话）
  指定的目录的名为`pip`的子目录中，如`/etc/xdg/pip/pip.conf`
* OSX系统中位于： `/Library/Application Support/pip/pip.conf`
* XP系统中位于： 
  `C:\Documents and Settings\All Users\Application Data\pip\pip.ini`
* Win7和以后的系统中是hidden的，但是可以写 `C:\ProgramData\pip\pip.ini`

> 系统级的配置文件不支持Windows Vista

### 2.用户级的的配置文件
用户级的pip配置文件位于：

* Unix系统中位于：`$HOME/.config/pip/pip.conf`（由`XDG_CONFIG_HOME`环境变量指定的），或者`$HOME/.pip/pip.conf`（优先级较低）中
* OSX系统中位于：`$HOME/Library/Application Support/pip/pip.conf`
* Windows系统中位于：`%APPDATA%\pip\pip.ini`或者`%HOME%\pip\pip.ini`（优先级较低）中

> 你可以通过环境变量 `PIP_CONFIG_FILE`  来指定这个用户级别的配置文件的位置

### 3.virtualenv中的配置文件
你可以在env中设置不同虚拟环境下的配置文件：

* 在Unix/OSX系统中位于：`$VIRTUAL_ENV/pip.conf`
* 在Windows系统中位于： `%VIRTUAL_ENV%\pip.ini`

### 4.配置文件的内容

		[global]   #针对所有pip命令的配置
		timeout = 60 #超时
		index-url = http://pypi.douban.com/simple #不同的库url
		trusted-host = pypi.douban.com        #添加豆瓣源为可信主机，要不然可能报错
		disable-pip-version-check = true      #取消pip版本检查，排除每次都报最新的pip

		[install] #针对具体的pip命令的配置
		ignore-installed = true
		no-dependencies = yes

## 四、系统site-packages和用户级site-packages

Python支持用户级安装。所谓用户级安装用于以下两种情形：

* 当用户没有写`global site-packages`目录时
* 当用户不想安装在`global site-packages`目录时

用户级安装命令是在常规的`install`命令后添加`--user`选项。
安装的目录由`PYTHONUSERBASE`环境变量决定（该环境变量影响的是`site.USER_BASE`变量的值）。在Windows7下，这个目录是：`C:\Users\huaxz\AppData\Local\pip\cache\wheels`

所有的pip命令，这些命令后面添加`--user`，则会特别针对用户级安装目录。

  ![pip --user选项](./imgs/pip/user_options.JPG) 


