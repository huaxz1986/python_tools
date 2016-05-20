# Sphinx笔记（基于Sphinx 1.3.1）

## 一、入门

1.Sphinx支持HTML、LaTex、man、纯文本输出。它使用结构化文本标记(reStructuredText)作为标记语言

2.Sphinx内置了一个`sphinx-quickstart`脚本用于自动生成默认的`conf.py`配置文件

* 包含所有原始文本的目录称为资源目录。该目录必须包含Sphinx配置文件`conf.py`
* Sphinx根据`conf.py`的变量值来查阅和生成文档
* Sphinx的`sphinx-quickstart`脚本除了自动生成`conf.py`之外还会生成一个主文档`index.rst`。主文档是欢迎页，它包含了`toctree`（即内容属索引）。主文档的初始内容为：

  ```
	.. toctree ::
	   :maxdepth: 2
  ```
3.编译文档的命令：`sphinx-build -b html sourcedir  builddir`

* `sourcedir`为源代码所在的目录，`builddir`为目标目录，通常该目录都命名为`doc`
* `-b`指定了输出格式：
	* `html`：输出为HTML格式（默认行为）
	* `dirhtml`：输出为HTML格式，但是输出到单一目录中
	* `singlehtml`：输出为一个单页的HTML页面
	* `htmlhelp`、`qthelp`、`devhelp`、`epub`	:构建为HTML文件并配合附加信息打包为上述指定的格式
	* `latex`：输出为Latex源码
	* `man`：输出为手册页面
	* `text`：输出为纯文本
	* `linkcheck`：检查所有外部链接的完整性，没有输出
	* `gettext`：输出为`.pot`文档
	* `doctest`：若`doctest`扩展已经激活，则执行所有文档中的`doctests`
* `sphinx-quickstart`脚本已经创建了`Makefile`以及`make.bat`（Windows环境中），可以简单的使用命令`make html`构建

4.`conf.py`已经写入了相关配置项，你可以取消某些行的注释并修订参数值。
> `conf.py`文件默认使用UTF-8编码保存。如果有参数值使用非ASCII字符，则必须用UNICODE形式

## 二、Autodoc
1.对Python进行文档化时，常规做法是把大量内容直接填入源码各个字符串位置。Sphinx支持直接从模块中提取这种类型的文档，这通过Autodoc扩展来实现。

* 要使用autodoc扩展，必须在`conf.py`的`extensions`列表中包含`sphinx.ext.autodoc`
* autodoc需要导入你的模块来加载文档字符串，因此你必须在配置文件`conf.py`的`sys.path`列表中追加目标模块所在的路径（如果该路径已存在与`sys.path`中，则不需要此步骤）


2.添加docstring：在`data member`以及`class attribute`之后你都可以用`'''   ...'''`添加文档字符串

```
class Foo:
	'''
	This is docstring for Foo
	'''
	bar=1
	'''
	This is description for bar
	'''
	def __init__(self):
	'''
	This is description for __init__
	'''
		self.param=4
		'''
		This is  description for param
		'''
```
* 在注释文字中，可以用reST语法、Markdown语法
* 模块的文档字符串必须放在文档的首行
* 你也可以通过添加注释来完成文档化：注释要么在定义语句同一行，位于定义语句之后；要么在定义语句之前的一行或者几行
	
```
class Foo: #This is comments for Foo ,must be one line
   	# This is comment for bar in front of bar definine ,can bu multi line
	# This is comment for bar in front of bar definine ,can bu multi line
	bar=1	
	def __init__(self):#This is comments for __init__ ,must be one line
		self.param=4#This is comments for param ,must be one line		
```

3.必须要用`sphinx-apidoc`脚本将python的docstring转换成`.rst`文档：

```
sphinx-apidoc [options] -o outputdir packagedir [pathnames]
```
然后再运行`sphinx-build`才能生成`autodoc`

* `outputdir`：输出`.rst`文档的目录；`packagedir`：Python源文件的根目录
* 通常把所有文档放在一个叫`doc`的目录中

4.autodoc生成的文件内容为：` .. automodule :: `：为`module`生成自动化文档； ` .. autoclass :: `：为`class`生成自动化文档； ` .. autoexception :: `：为`exception`生成自动化文档

* 其语义为：

	```
	.. automodule :: modname
  	   :members:
	```
  会对`modname`这个模块生成自动化文档，并对其所有`member`（递归地）打文档（通过查看模块的`__all__`属性查找`member`）

	```
	.. autoclass:: classname
  	   :members:
	```
	会对`classname`这个类生成自动化文档，会对`classname`类的所有非私有成员（即不是以`_`开头的成员）打文档
* 你也可以显式指定`member`，此时只有指定的`member`打文档，其他的`member`不打文档：

	```
	.. automodule :: modname
  	   :members: mod_member1,mod_member2
	```
* 没有`docstring`的`member`默认不会打文档。若想增加对这些不带`docstring`的`member`打文档，可以用：
	
	```
	.. automodule :: modname
  	   :members:
  	   :undoc-members:	
	```

* 若添加`:private-members：`则会对私有成员（以`_`或者`__`开头的成员）打文档  
  若添加`:special-members：`则会对特殊成员（以`__`开头且以`__`结尾的成员）打文档 
* 对于`class`与`exception`，`member`默认不包含基类的成员。若想添加基类的成员文档，
  则添加`:inherited-members:`
* 若想排除某个指定`member`不打文档，则添加`:exclude-members: member1,member2`
* `:show-inheritance:`选项会在每个`class`、`exception`文档后插入基类的信息

5.`autodoc`通过导入模块，然后观察`.__doc__`属性来生成docstring。因此如果你对使用了装饰器的函数、类打文档，必须在装饰器中拷贝被装饰函数、类的`.__doc__`属性


				
