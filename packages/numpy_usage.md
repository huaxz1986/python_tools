<!--
    作者：华校专
    email: huaxz1986@163.com
**  本文档可用于个人学习目的，不得用于商业目的  **
-->
# Numpy学习笔记 （基于Numpy 1.11.0）

- `ndarray`：`NumPy`最重要的对象就是`ndarray`（也可以称它为`array`），它是一个多维的容器。其存放元素通常是相同的类型和大小的。
- `axes`：`Numpy`的维度称之为`axes`
- `rank`：维度数量称之为`rank`

比如`ndarray:[[1,0,0],[0,1,2]]`的`rank`为2，第一维的长度为2（行），第二维的长度为3（列）.
>- `numpy.ndarray`与`Standard Python Library`的`array.array`不同，二者并不是同一个类
>- `ndarray`的维度可以为0，此时为标量；若维度为1维则为向量；若维度为2维则为矩阵....

## 一、 `ndarray`

`ndarray`比较重要的属性有：

- `ndarray.ndim`：`ndarray`的`rank`，即有几个维度
- `ndarray.shape`：`dimensions of the array`。它是`N`个正整数组成的元组，用于指示容器每一个维度的大小。对于一个`n`行`m`列的矩阵，`.shape`为`(n,m)`。因此`.shape`的`length`等于`ndarray.ndim`
- `ndarray.size`：`ndarray`中`elements`的总数量。它等于`.shape`中各分量的乘积
- `ndarray.dtype`：一个`object`用于描述`array`中`elements`的`type`。
	>你可以创建标准的`Python type`，也可以用`Numpy`提供的`type`，如`numpy.int32`、`numpy.float64`等
- `ndarray.itemsize`：`array`中每个`element`的字节大小。它也等于`ndarray.dtype.itemsize`
- `ndarray.data`：存放真正数据的`buffer`。通常我们不会使用这个属性，因为我们从不通过这个属性来访问数据。
	> 通常都是用索引来访问数据
- `ndarray.real`：返回`ndarray`的实部
- `ndarrray.imag`：返回`ndarray`的虚部
- `ndarray.flat`：返回`ndarray`上的一个一维的`iterator`
- `ndarray.T`：返回`ndarray`的转置。如果`ndarray`维度小于1，则返回它本身

> 如果每一行的列数不是完全相同，那么`.shape`属性只会显示`(rows,)`这种格式。

  ![ndarray基本属性](./imgs/numpy/ndarray_attribute.JPG) 

### 1. 创建`ndarray`

创建`ndarray`对象有很多方法：

#### a. 创建全1或者全0

- `empty(shape[,type,order])`：返回一个新的`ndarray`，指定了`shape`和`type`，但是没有初始化元素。
  因此其内容是随机的。
- `empty_like(a[,type,order,subok])`：返回一个新的`ndarray`，
  `shape`与`a`相同，但是没有初始化元素。因此其内容是随机的。

  ![empty](./imgs/numpy/empty.JPG) 

- `eye(N[, M, k, dtype])`：返回一个二维矩阵，对角线元素为1，其余元素为0。`M`默认等于`N`。`k`默认为0表示对角线元素为1，如为正数则表示对角线上方一格的元素为1，如为负数表示对角线下方一格的元素为1.
- `identity(n[, dtype])` ：返回一个单位矩阵

  ![eye_identity](./imgs/numpy/eye_identity.JPG) 

- `ones(shape[, dtype, order])`：返回一个新的`ndarray`，指定了`shape`和`type`，每个元素初始化为1.	- `ones_like(a[, dtype, order, subok])` ：返回一个新的`ndarray`，
  `shape`与`a`相同，每个元素初始化为1。

  ![ones](./imgs/numpy/ones.JPG) 

- `zeros(shape[, dtype, order])` ：返回一个新的`ndarray`，指定了`shape`和`type`，每个元素初始化为0.
- `zeros_like(a[, dtype, order, subok])`：返回一个新的`ndarray`，
  `shape`与`a`相同，每个元素初始化为0。

  ![zeros](./imgs/numpy/zeros.JPG)

- `full(shape, fill_value[, dtype, order])`：返回一个新的`ndarray`，指定了`shape`和`type`，
  每个元素初始化为`fill_value`。
- `full_like(a, fill_value[, dtype, order, subok])`：返回一个新的`ndarray`，
  `shape`与`a`相同，每个元素初始化为`fill_value`。

 ![full](./imgs/numpy/full.JPG)

这里有几个共同的参数：

- `a`：一个`array-like`类型的实例，它不一定是`ndarray`，可以为`list`、`tuple`、`list of tuple`、`list of list`、`tuple of list`、`tuple of tuple`等等。
- `dtype`：结果的元素的类型，默认为`float`。你可以指定`Python`的标准数值类型，也可以使用`numpy`的数值类型如：`numpy.int32`或者`numpy.float64`等等。
- `order`：指定存储多维数据的方式。可以为`'C'`，表示按行优先存储（C风格）；可以为`'F'`，表示按列优先存储（Fortran风格）
	> 对于`**_like()`函数，`order`可以为:`'C'`，`'F'`，`'A'`（表示结果的`order`与`a`相同），`'K'`（表示结果的`order`与`a`尽可能）
- `subok`：`bool`值，如果为`True`则如果`a`为`ndarray`的子类（如`matrix`类），则结果类型与`ndarray`类型相同。如果为`False`则结果类型始终为`ndarray`。默认为`True`。

#### b. 从现有数据创建

- `array(object[, dtype, copy, order, subok, ndmin])`:从`object`创建。
	- `object`可以是一个`ndarray`，也可以是一个`array_like`的对象，
	  也可以是一个含有返回一个序列或者`ndarray`的`__array__`方法的对象，或者一个序列。
	- `copy`：默认为`True`，表示拷贝对象
	- `order`可以为`'C'、'F'、'A'`。默认为`'A'`。
	- `subok`默认为`False`
	- `ndmin`：指定结果`ndarray`最少有多少个维度。
- `asarray(a[, dtype, order])`：将`a`转换成一个`ndarray`。其中`a`是`array_like`的对象，
  可以是`list`、`list of tuple`、`tuple`、`tuple of list`、`ndarray`类型。`order`默认为`C`。
- `asanyarray(a[, dtype, order])`:将`a`转换成`ndarray`。
- `ascontiguousarray(a[, dtype])` ：返回C风格的连续`ndarray`
- `asmatrix(data[, dtype]) `：返回`matrix`
- `copy(a[, order])`:返回`ndarray`，从输入中拷贝元素
- `frombuffer(buffer[, dtype, count, offset])`：从输入数据中返回一维`ndarray`。`count`指定读取的数量，`-1`表示全部读取；`offset`指定从哪里开始读取，默认为0
- `fromfile(file[, dtype, count, sep])` :从二进制文件或者文本文件中读取数据返回`ndarray`.`sep`：当从文本文件中读取时，数值之间的分隔字符串，如果`sep`是空字符串则表示文件应该作为二进制文件读取；如果`sep`为`" "`表示可以匹配0个或者多个空白字符。
- `fromfunction(function, shape, **kwargs)`:返回一个`ndarray`。从函数中获取每一个坐标点的数据。假设`shape`的`rank`为N，那么`function`带有`N`个参数，`fn(x1,x2,...x_N)`，其返回值就是该坐标点的值。
- `fromiter(iterable, dtype[, count])`：从可迭代对象中迭代获取数据创建一维`ndarray`。
- `fromstring(string[, dtype, count, sep]) `：从字符串或者`raw binary`中创建一维`ndarray`。如果`sep`为空字符串则`string`将按照二进制数据解释（即每个字符作为`ASCII`码值对待）。
- `loadtxt(fname[, dtype, comments, delimiter, ...])`:从文本文件中加载数据创建`ndarray`，要求文本文件每一行都有相同数量的数值。`comments`：指示注释行的起始字符，可以为单个字符或者字符列表（默认为`#`）.`delimiter`:指定数值之间的分隔字符串，默认为空白符。`converters`：将指定列号(0,1,2...)的列数据执行转换，是一个`map`，如`{0:func1}`表示对第一列数据执行`func1(val_0)`。`skiprows`：指定跳过开头的多少行。`usecols`：指定读取那些列（0表示第一列）。

  ![array_from_data](./imgs/numpy/ndarray_from_data.JPG)

#### c. 从数值区间创建

- `arange([start,] stop[, step,][, dtype])`:返回均匀间隔的值组成的一维`ndarray`。区间是半闭半开的`[start,stop)`，其采样行为类似Python的`range`函数。`start`为开始点，`stop`为终止点，`setp`为步长，默认为1。这几个数可以为整数可以为浮点数。注意如果`setp`为浮点数，则结果可能有误差，因为浮点数相等比较不准确。
- `linspace(start, stop[, num, endpoint, ...])` ：返回`num`个均匀采样的数值组成的一维`ndarray`（默认为50）。区间是比区间`[start,stop]`。`endpoint`为布尔值，如果为真则表示`stop`是最后采样的值（默认为`True`），否则结果不包含`stop`。`retstep`如果为`True`则返回结果包含采样步长`step`，默认为`True`。
- `logspace(start, stop[, num, endpoint, base, ...])`：返回对数级别上均匀采样的数值组成的一维`ndarray`。采样点开始于`base^start`，结束于`base&stop`。`base`为对数的基。
	>它逻辑上相当于先执行`arange`获取数组`array`，然后再执行`base^array[i]`获取采样点

  ![array_from_data](./imgs/numpy/ndarray_from_range.JPG) 

### 2. 打印`ndarray`

当打印`ndarray`时，`numpy`按照Python的嵌套`list`的格式打印输出，但是按照以下顺序打印：

- 最底层的`axis`按照从左到右的顺序输出
- 次底层的`axis`按照从上到下的顺序输出
- 其他层的`axis`也是按照从上到下的顺序输出，但是每个`slice`中间间隔一条空行

如： 一维的`ndarray`按行打印；二维的`ndarray`按照矩阵打印；三维的`ndarray`按照矩阵的`list`打印

如果`ndarray`太大，那么`numpy`默认跳过中间部分的数据而只是输出四个角落的数据。
>要想任何时候都打印全部数据，可以在`print(array)`之前设置选项`numpy.set_printoptions(threshold='nan')`。这样后续的打印`ndarray`就不会省略中间数据。

### 3. `ndarray`算术运算和比较运算

`ndarray`的算术运算、比较操作都是元素级的操作并且产生一个`ndarray`对象作为结果。`ndarray`的算术运算符`+, -, *, /, //, %, divmod(), ** or pow(), <<, >>, &, ^, |, ~`以及比较运算符`==, <, >, <=, >=, !=`等价于相应的`numpy`中的`universal function`。 

注意：
- 如果两个`ndarray`的元素类型不相同，则四则运算结果的类型由精度最高的那个`ndarray`决定。
- `ndarray`支持原地算术操作，如`+=`、`*=`等，他会原地修改已存在的数组而不是创建一个新的数组。
- `ndarry`的矩阵`*`运算执行的是元素级相乘，而不是数学上的矩阵乘法。如果你想执行矩阵乘法，
  可以使用`numpy.dot(array1,array2)`实现两个`ndarray`的矩阵乘法。你也可以通过`array1.dot(array2)`
  实现同样的效果

  ![ndarray四则运算](./imgs/numpy/arithmetic.JPG) 

  ![ndarray比较运算](./imgs/numpy/compare.JPG)

### 4. 归一化操作

`ndarray`支持若干归一化方法：

- `ndarray.sum()`：累加`ndarray`所有元素的，并返回最终的和
- `ndarray.min()`：返回所有元素的最小值
- `ndarray.max()`：返回所有元素的最大值
- `ndarray.cumsum()`：返回所有元素的累加和，返回累加和序列

对于这些方法你如果想在指定`axis`上操作，则可以提供`axis`关键字参数，其参数值从（0到`ndim`）。默认情况下`axis=None`，表示将`ndarray`看作是一个一维的`ndarray`来操作。

  ![三维ndarray示意图](./imgs/numpy/dim_3.JPG) 

  ![ndarray比较运算](./imgs/numpy/unary_operation.JPG) 

### 5. 通用函数

`numpy`提供常见的数学函数如`sin`、`cos`、`exp`等，这些函数作用于`ndarray`的每一个元素，因此称为通用函数(`ufunc`)。所有的函数参考 https://docs.scipy.org/doc/numpy/reference/routines.html。

  ![ndarray通用函数](./imgs/numpy/universal_function.JPG)

### 6. 索引、切片、迭代

- 一维数组可以被索引、切片、迭代，就像Python 列表一样操作
- 多维数组可以每个轴一个索引。这些索引由一个逗号分割的元组给出。
	- 当提供少于轴数的索引时，缺失的索引被认为是整个切片（必须提供前面的索引，可以缺失末尾的索引），如二维数组的`a[1]`
	- 如果想让缺失索引发生在前面，必须用`...`表示缺失索引。如二维数组的`a[...,1]`；而`a[1,...]`等价于`a[1]`
	- 你也可以显式的用`:`指定整个切片
- 多维数组的迭代仅仅只是迭代第一个轴。如果想迭代多维数组的每一个元素，可以用`ndarray.flat`属性，它是一个迭代器对象，迭代结果为`ndarray`的所有元素

  ![ndarray索引、切片、迭代](./imgs/numpy/index_slice.JPG)

### 7. `ndarray`的变形

`ndarray`的形状由`ndarray.shape`属性给出，它是一个元组，指出每个轴上的元素个数。你可以通过以下函数改变一个`ndarray`的形状：

- `numpy.ravel(a,order='C')`：将`a`展平，返回一个连续的一维`ndarray`。`a`是一个`array-like`对象，它的读取顺序为`order`（默认为C风格表示行优先读取）。
	- 如果`a`为`ndarray`，则可以用`ndarray.ravel(...)`方法，它等价于本函数
- `numpy.resize(a,new_shape)`：返回一个新`ndarray`，它是`a`满足`new_shape`的部分。其中`a`为一个`array-like`对象，而`new_shape`为新数组的形状。如果`new_shape`大于`a`的形状，则新`ndarray`复制`a`来填充新的部分。
	> 所谓的复制就是：首先展开，然后循环复制，然后变形
- `numpy.reshape(a,newshape,order='C')`:返回`a`的形状改变后的新`ndarray`。`a`为一个`array-like`对象。`newshape`必须与旧形状兼容（满足元素数量前后相同）。`a`的读取顺序为`order`（默认为C风格表示行优先读取）。
	- 如果`a`为`ndarray`，则可以用`ndarray.reshape(..)`方法，它等价于本函数
	>如果想改变`a`的形状，则直接修改`a.shape`属性即可

  ![ndarray变形](./imgs/numpy/reshape.JPG)

### 8. `ndarray`的拼接

有几种办法将`ndarray`拼接起来：

-  `numpy.hstack(seq)`：水平拼接（即其他维度不变，维度1拼接），返回新创建的拼接后的`ndarray`。`seq`是一个序列，每个元素为一个`array-like`对象或者`ndarray`，且要求每个元素满足形状条件：除了维度1长度，其他维度长度必须相同。
-  `numpy.vstack(seq)`：垂直拼接（即其他维度不变，维度0拼接），返回新创建的拼接后的`ndarray`。`seq`是一个序列，每个元素为一个`array-like`对象或者`ndarray`，且要求每个元素满足形状条件：除了维度0长度，其他维度长度必须相同。
- `numpy.column_stack(seq)`：将一维数组按列拼接成二维数组，返回新创建的拼接后的`ndarray`。`seq`是一个序列，每个元素为一个`array-like`对象或者`ndarray`，且要求是一维的。拼接规则为：第一维长度为原数组长度，第二维长度为2，结果数组第二维每个元素为原来的数组序列中各取一个值的组成的数组。
- `numpy.concatenate((a1, a2, ...), axis=0)`：将一系列数组按照指定轴线拼接，返回新创建的拼接后的`ndarray`。`a1、a2、....`都是`array_like`对象，且要求每个元素满足形状条件：除了维度`axis`长度，其他维度长度必须相同。`axis`为拼接轴，默认为0（以哪个轴为拼接轴，就变化该轴的长度）

  ![ndarray拼接](./imgs/numpy/concatenate.JPG)

### 9. `ndarray`的拆分

有几种办法将`ndarray`拆分：

- `numpy.hsplit(array, indices_or_sections)`：将`array`按照水平拆分（沿着1轴），返回一个`ndarray`的列表。`array`为一个`ndarray`对象。	
- `numpy.vsplit(array, indices_or_sections)`：将`array`按照垂直拆分（沿着0轴），返回一个`ndarray`的列表。`array`为一个`ndarray`对象。`indices_or_sections`：
- `numpy.array_split(array, indices_or_sections,axis=0)`：将`array`按照指定轴线`axis`拆分，返回一个`ndarray`的列表。`array`为一个`ndarray`对象。`indices_or_sections`：

其共同的参数`indices_or_sections`：为一个整数标量或者一个一维数组。

- 如果是个整数`N`，则表示`array`沿着均匀拆分轴拆成`N`份。如果不能均匀拆分，则抛出异常
- 如果是个一维数组，则指示了拆分点。如`indices_or_sections`为`[2,3]`，则表示拆分段为
  `[0：2]`、`[2:3]`、`[3:]`。如果该一维数组中拆分点超过了轴线长度，则返回空的数组

  ![ndarray拆分](./imgs/numpy/split.JPG)

### 10. `ndarray`的拷贝和视图

当处理`ndarray`时，它的基础数据有时被拷贝，但有时并不被拷贝。有三种情况。

#### a. 完全不拷贝

简单的赋值操作并不拷贝`ndarray`的数据，这种情况下是新的变量引用`ndarray`对象（类似于列表的简单赋值）

  ![完全不拷贝](./imgs/numpy/no_copy.JPG)

#### b. 视图和浅拷贝

不同的`ndarray`可能共享相同的基础数据。`ndarray.view()`方法创建一个新的`ndarray`但是与旧`ndarray`共享相同的数据（即新`ndarray`并不拥有数据）。

- 对于视图，`ndarray.base`返回的是视图的底层`ndarray`。而非视图`ndarray.base`返回`None`
- `ndarray.flags.owndata`返回`ndarray`是否拥有基础数据

对于`ndarray`的分片操作，它返回的是一个`ndarray`的视图。对`ndarray`的索引返回的不是视图，而是含有基础数据。

  ![ndarray视图](./imgs/numpy/ndarray_view.JPG)

#### c. 深拷贝

`ndarray.copy()`操作会返回一个完全的拷贝，不仅拷贝`ndarray`也拷贝基础数据。

  ![深拷贝](./imgs/numpy/deep_copy.JPG)

### 11. 广播法则

当通用函数处理多个不同形状的`ndarray`时，需要使用广播法则来处理。使用广播法则的前提条件：两个`ndarray`的对应维度的长度，要么相等；如果不等则必须有一个长度为1。不满足条件则不能使用广播法则。广播法则的条款是：

- 如果所有的输入数组维度不是完全相同，则某种规则将会被重复地添加数据到维度较小的`ndarray`的数组上直到所有的`ndarray`拥有一样的维度。
- 添加规则：确定`ndarray`的维度为长度为1的轴的方向扩张，且扩张时填充的数据都相同，就是该轴的原始数据（因为该维度长度为1）

  ![广播_上](./imgs/numpy/broadcasting_1.JPG)
  ![广播_下](./imgs/numpy/broadcasting_2.JPG)

  ![广播示例](./imgs/numpy/broadcasting_example.JPG)

### 12. 索引技巧

除了利用Python中的索引和分片规则之外，`ndarray`还支持整数数组索引以及布尔数组索引。

#### a. 整数数组索引

索引数组每个整数是对`ndarray`的索引下标。其结果是一个`ndarray`的视图，外层维度由索引数组的维度，内层维度由`ndarray`的维度决定。

  ![数组索引](./imgs/numpy/index_is_array.JPG)

- 当被索引的`ndarray`是一个多维数组时，对它的索引是第0维的索引

  ![多维数组的索引](./imgs/numpy/mul_dimension_array_index_array.JPG)

- 当索引数组是一个多维数组时，每一维的索引数组必须有相同的形状
	- 这里你可以将索引数组放入序列中，如`a[[index1,index2]]`，它等价于`a[index1,index2]`。
		>注意这里的`index1、index2`均为索引数组。如果都是整数则`a[[inti,intj]]`（返回`ndarray`）
		  不等于`a[inti,intj]`（返回标量）。

  ![多维数组的多维索引](./imgs/numpy/mul_array_mul_index.JPG)

- 你可以使用数组索引作为目标来赋值。
	- 假设左侧被赋值的维度为m1×n1，则右侧赋值数据的维度可以为：m1×n1、m1×1、1×n1、1*1（标量）。
	  如果不满足则抛出异常。
	- 左侧的索引包含重复时，赋值被多次完成。保留最后的值。（要小心Python的+=结构）
	
  ![多维数组的多维索引](./imgs/numpy/index_mul_array_assignment.JPG)

#### b. 通过布尔数组索引

通过布尔数组索引能够显式的选择数组中我们想要和不想要的元素。

- 最简单的方式就是使用和原数组一样形状的布尔数组
	> 常用于一次性修改原数组的不规则位置处的值

  ![布尔数组索引](./imgs/numpy/bool_index.JPG)

- 我们也可以采取类似整数切片的方式，对于原数组的每一个维度我们给以一维的布尔数组来选择我们需要的切片
	> 要求一维布尔数组的长度与你要切片的轴的长度一致

  ![布尔数组分片](./imgs/numpy/bool_index_slice.JPG)

### 11. `ix_()`

`ix_()`函数能够排列组一组向量中的值，从而获取所有排列组合的结果。它接受`N`个一维数组，返回一个`N`元素的元组，其中元组的第`k`个元素都是一个这样的`N`维数组`ndarray`：该`ndarray`第`k`维长度等于参数列表第`k`个参数向量长度，第`k`维的值就是参数列表第`k`个参数向量的值，除了第`k`维之外所有其他维度的长度为1。
> 它的作用就是重排每一个向量，使得重排后每一个数组变成`N`维，然后这些数组依次占领第0维，第1维....第`N-1`维，从而使得广播法则能够起作用。

  ![ix_()](./imgs/numpy/ix_function.JPG)

## 二、线性代数

### 1. 转置

`numpy.transpose(a,axes=None)`：获取`a`的转置（一个`array-like`对象）。其中`axes`是一个整数列表，指定交换轴。如果`axes`未指定，则默认翻转维度。

  ![transpose](./imgs/numpy/transpose_function.JPG)

### 2. 逆矩阵

`numpy.linalg.inv(a)`：获取`a`的逆矩阵（一个`array-like`对象）。

- 如果传入的是多个矩阵，则依次计算这些矩阵的逆矩阵。
- 如果`a`不是方阵，或者`a`不可逆则抛出异常

  ![transpose](./imgs/numpy/inv_function.JPG)

### 3. 单位矩阵

`numpy.eye(N[, M, k, dtype])`：返回一个二维单位矩阵行为`N`，列为`M`，对角线元素为1，其余元素为0。`M`默认等于`N`。`k`默认为0表示对角线元素为1（单位矩阵），如为正数则表示对角线上方一格的元素为1（上单位矩阵），如为负数表示对角线下方一格的元素为1（下单位矩阵）

### 4. 点积

`numpy.dot(a, b, out=None)`：计算两个`array-like`对象的点乘。

- 如果是二维矩阵，则点乘就是代数上的矩阵乘法
- 如果是一维向量，则点乘就是向量的内积
- 如果是标量，则点乘就是普通乘法
- 如果是`N`维数组，则是`a`的最后一个轴与`b`的次最后轴之间的乘积和

  ![dot](./imgs/numpy/dot_function.JPG)

### 5. 向量点积

`numpy.vdot(a, b)`：返回一维向量之间的点积。如果`a`和`b`是多维数组，则展平成一维再点积。

  ![vdot](./imgs/numpy/vdot_function.JPG)

### 6. 向量外积

`numpy.outer(a, b, out=None)`：返回一维向量之间的外积。如果`a`和`b`是多维数组，则展平成一维再外积。
 假设`a = [a0, a1, ..., a_M]`，`b = [b0, b1, ..., b_N]`,则外积为:

```
 [
  [a0*b0  a0*b1 ... a0*b_N ]
  [a1*b0    .
  [ ...          .
  [a_M*b0            a_M*b_N ]
 ]
```

  ![outer](./imgs/numpy/outer_function.JPG) 

### 7. 叉乘

`numpy.cross(a, b, axisa=-1, axisb=-1, axisc=-1, axis=None)`:计算两个向量之间的叉乘。叉积用于判断两个三维空间的向量是否垂直。要求`a`和`b`都是二维向量或者三维向量，否则抛出异常。（当然他们也可以是二维向量的数组，或者三维向量的数组，此时一一叉乘）

  ![cross](./imgs/numpy/cross_function.JPG)

### 8. 对角线和

`numpy.trace(a, offset=0, axis1=0, axis2=1, dtype=None, out=None)`：返回对角线的和。如果`a`是二维的，则直接选取对角线的元素之和（`offsert=0`），或者对角线右侧偏移`offset`的元素之和
> 即选取`a[i,i+offset]`之和

- 如果`a`不止二维，则由`axis1`和`axis2`指定的轴选取了取对角线的矩阵。
- 如果`a`少于二维，则抛出异常

  ![trace](./imgs/numpy/trace_function.JPG)

### 9. 计算线性方程的解`Ax=b`

`numpy.linalg.solve(a,b)`：计算线性方程的解`ax=b`，其中`a`为矩阵，要求为秩不为0的方阵，`b`为列向量（大小等于方阵大小）；或者`a`为标量，`b`也为标量。
> 如果`a`不是方阵或者`a`是方阵但是行列式为0，则抛出异常

  ![solve](./imgs/numpy/solve_function.JPG)

### 10. 特征值

`numpy.linalg.eig(a)`：计算矩阵的特征值和右特征向量。如果不是方阵则抛出异常，如果行列式为0则抛出异常。

  ![eig](./imgs/numpy/eig_function.JPG)

### 11. 奇异值分解

`numpy.linalg.svd(a, full_matrices=1, compute_uv=1)`:对矩阵`a`进行奇异值分解，将它分解成`u*np.diag(s)*v`的形式，其中`u`和`v`是酉矩阵，`s`是`a`的奇异值组成的一维数组。
其中：

- `full_matrics`:如果为`True`，则`u`形状为`(M,M)`，`v`形状为`(N,N)`；否则`u`形状为`(M,K)`，`v`形状为`(K,N)`，`K=min(M,N)`
- `compute_uv`：如果为`True`则表示要计算`u`和`v`。默认为`True`。
- 返回`u`、`s`、`v`的元组
- 如果不可分解则抛出异常

  ![svd](./imgs/numpy/svd_function.JPG)


## 三、概率与分布函数


## 四、技巧

### 1. 自动形状推断

当你修改`ndarray`的形状时，你可以省略某个维度的尺寸（设置为`-1`，该维度尺寸会自动推断出来。如：
`a.shape=2,-1,3 # -1 表示能够自动推断出来`

### 2. `histogram` 

`numpy.histogram(a, bins=10, range=None, normed=False, weights=None, density=None)`函数应用到一个数组返回一对变量：

- 直方图数组：区间内统计的样本数量（也可能归一化为样本频率）
- 长条的底边划分序列：区间划分点（该划分点将`range`划分为若干个区间）

其个参数名字为：

- `a`：`array-like`对象，为输入数据。如果是多维数组则展平为一维
- `bins`：
	- 如果为整数，则它定义了在指定`range`内的等宽度长条的数量
	- 如果为整数序列，则它定义了长条的底边
	- 如果为字符串，则比较复杂，具体可参考文档
- `range`：所有等宽长条的下界和上界。如果未提供则结果是元组`(a.min(),a.max())`。如果`a`有元素值在它之外，则忽略该元素。
- `normed`：该关键字是`deprecated`。
- `weights`：权重数组。它是`a`形状相同，它给出了对应的`a`中的元素的权重。如果`desity`为`True`，则该数组被归一化。
- `density`:如果为`False`，则结果是每个长条的样本数量；如果为`True`则结果是每个长条的频率值（估计密度函数）。


> 注意：`matplotlib.pyplot`也有一个建立直方图的函数（`hist(...)`），区别在于`matplotlib.pyplot.hist(...)`函数会自动绘直方图，而`numpy.histogram`仅仅产生数据

  ![hist](./imgs/numpy/matplotlib_hist.JPG)

  ![histogram](./imgs/numpy/histogram_function.JPG)