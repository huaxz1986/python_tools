<!--
    作者：华校专
    email: huaxz1986@163.com
**  本文档可用于个人学习目的，不得用于商业目的  **
-->
# SQLAlchemy 笔记（基于1.0.12版本）

SQLAlchemy的经典用法是使用它的`ORM (Object Relational Mapper)`。

- 它可以定义一个`Python class`来关联到一个`database table`，该类的实例关联到对应`table`的一行记录。
- 它可以通过修改`Python class`实例的属性从而修改关联的`table`记录

## 一、连接数据库

连接数据库使用`create_engine()`函数：`sqlalchemy.create_engine(*args, **kwargs)`。它返回一个`Engine`实例。

### 1. URL参数

通常`create_engine()`函数的第一个参数是个`url`字符串，表示采用的数据库。格式为：`'dialect[+driver]://user:password@host/dbname[?key=value..]'`。其中：

- `dialect`是数据库的名字，可以为：`'mysql'`、`'oracle'`、`'postgresql'`、`'sqlite'`等等
- `driver`是`DBAPI`（驱动程序）的名字，可以为：`'psycopg2'`、`'pyodbc'`、`'cx_oracle'`等等

典型的`url`：

- postgresql: 
 	- `default`：`create_engine('postgresql://scott:tiger@localhost/mydatabase')`
	- `psycopg2`：`create_engine('postgresql+psycopg2://scott:tiger@localhost/mydatabase')`
	- `pg8000`：`create_engine('postgresql+pg8000://scott:tiger@localhost/mydatabase')`
- mysql:
	- `default`：`create_engine('mysql://scott:tiger@localhost/foo')`
	- `mysql-python`：`create_engine('mysql+mysqldb://scott:tiger@localhost/foo')`
	- `MySQL-connector-python`：`create_engine('mysql+mysqlconnector://scott:tiger@localhost/foo')`
	- `OurSQL`:`create_engine('mysql+oursql://scott:tiger@localhost/foo')`
- oracle:
	- `default`：`create_engine('oracle://scott:tiger@127.0.0.1:1521/sidname')`
	- `cx_oracle`:`create_engine('oracle+cx_oracle://scott:tiger@tnsname')`
- SQL Server:
	- `pyodbc`:`create_engine('mssql+pyodbc://scott:tiger@mydsn')`
	- `pymssql`:`create_engine('mssql+pymssql://scott:tiger@hostname:port/dbname')`	
- SQLite:`create_engine('sqlite:///foo.db')`（注意斜线和反斜线的转义处理）
	- 如果你想使用`:memory:`，则指定一个空URL即可，`create_engine('sqlite://')`
	

### 2. 关键字参数

`create_engine()`的关键字参数有很多。他们有的是用于指定`Engine`的参数，有的是用于指定`Engine`底层的`Dialect`或者`Pool`的参数。常用的关键字参数有： 

- `case_sensitive`：默认为True。如果为`False`则结果的`column names`是
  `case nsensitive`，如`row[’SomeColumn’]`和`row[’somecolumn’]`相同
- `echo`：默认为`False`。如果为`True`则`Engine`会对执行的每一条`statement`进行`log`（默认是输出到`sys.stdout`） 
- `encoding`：字符串编码，默认为`'utf-8'`
- `max_overflow`: the number of connections to allow in connection pool “overflow”,
that is connections that can be opened above and beyond the pool_size setting.默认为10.只针对`QueuePool`有效
- `pool`：已提供的连接池，默认为`None` 
- `pool_recycle`：连接池中的连接每隔多少秒（循环时间）就放回连接池中。默认为`-1`，意思是永不循环。 
- `pool_size`：连接池中打开的连接的数量，默认为5。它可以用于`QueuePool`和`SingletonThreadPool`.在`QueuePool`中，0意味着没有限制。
- `pool_timeout`：从连接池中获取连接的超时时间（秒），默认为30。只用于`QueuePool`

> `create_engine()`仅仅创建`Engine`实例，它并没有连接到数据库。只有当执行`Engine.execute()`或者`Engine.connect()`方法时，`Engine`才建立了数据库的连接。

  ![create_engine](imgs/sqlalchemy/create_engine.JPG)

## 二、 Class Mapping

一个`Mapped Class`映射了一张数据库的表。所有的`Mapped Class`都继承自一个基类，称之为`declarative base class`，该基类由`sqlalchemy.ext.declarative.declarative_base()`函数返回。

一个`Mapped Class`继承自`declarative base class`，并且有以下属性：

- 一个`__tablename__`属性，指定了该类所映射的数据库的表名
- 一个或者多个`Column`实例属性，该属性对应着所映射的数据库表的列
	> 它们和`__tablename__`均为Python描述符
- 其他的属性或者方法，这些属性或者方法与数据库表没有映射关系，即普通的Python属性或方法

  ![mapping_class](imgs/sqlalchemy/mapping_class.JPG)

### 1. Column

`sqlalchemy.schema.Column`代表了`database table`中的一列。其初始化方法为：
`__init__(*args, **kwargs)`，重要的关键字参数如下：

- `name`：列名。是个字符串。该参数是第一个位置参数，或者是关键字参数。如果该参数值字符串不包含大写字母，则列名视为`case insensitive`；否则列名视为`case sensitive`
- `type_`:列的类型，是一个`TypeEngine`子类的实例。当然你也可以传入一个`TypeEngine`子类（此时用它的初始化函数生成一个默认的实例）。该参数是第二个位置参数，或者是关键字参数。
	- 如果忽略该参数或者设为`None`，则默认为特殊的类型`NullType`
	- 如果该列使用了`ForeignKey`或者`ForeignKeyConstraint`，则该列的类型为外键引用的列的类型
	- 所有类型都在`sqlalchemy.types`中
- `*args`：其他的位置参数，包括`Constraint`、`ForeignKey`、`ColumnDefault`、`Sequence`实例等。
- `autoincrement`：列是否自增。如果为`True`，则该列（通常是存放整数）会随着`INSERT`操作自增，其值由`DBAPI cursor.lastrowid`属性返回。该属性只能用于满足下面条件的列：列存放的是整数，且列是`primary key`的组成部分，且不是外键
- `default`：一个标量，或者一个`Python callable`或者`ColumnElement`，用于该列的默认值。当插入记录且该列没有提供`value`时，使用这个`default`来插入
- `index`：如果为`True`则表示该列是`indexed`。如果你想对多格列的组合执行`index`，则需要使用`Index`显式构造
- `nullable`：如果为`True`，则表示`database table`中该列允许为`NULL`；否则为`NOT NULL`
- `onupdate`：一个标量，或者一个`Python callable`或者`ClauseElement`，用于该列的默认值。当`update`记录且该列没有提供`value`时，使用这个`default`来更新
- `primary_key`：如果为`True`则标记该列为主键。如果有多个列设置了`primary_key=True`，则这些列一起组合成主键
- `server_default`：一个`FetchedValue`实例，或者一个字符串，或者一个`text()`，表示该列`DDL DEFAULT value`
- `quote`：一个布尔值，表示是否对列名包裹上引号。如果是`None`则为默认行为：名字至少包含一个大写字母时，包裹上引号。
- `unique`：如果为`True`则表示该列有一个`unique constraint`。如果你想对多个列的组合指定`unique`，则需要通过`UniqueConstraint`或者`Index`显式构造
- `system`：如果为`True`则表示该列是`database`自动生成的列，因此该列并不会出现在`CREATE TABLE statement`中

## 三、 Schema

当定义好了`Mapped class`之后，该类存储了`database table`的大量信息。这些信息称之为`table metadata`。我们可以通过该类的`.__table__`属性查看，该属性是一个`Table`实例。

`sqlalchemy.schema.MetaData`是`Table`实例的集合（可以在`MetaData.tables`字典查看），它持有许多`Table`实例并且它可以绑定到`Engine`或者`Connection`。你可以通过它来执行许多数据库操作，如创建数据表等。

通常你可以通过`declarative base class`的`.metadata`属性来获取一个`MetaData`实例。

  ![MetaData](imgs/sqlalchemy/MetaData.JPG)

## 四、 Mapped Class 实例

`Mapped Class`实例提供了一个默认的构造方法，它自动接收关键字参数，这些关键字就是`mapped column`的`identifier`。你可以自定义一个`.__init__()`方法，此时自定义的方法会覆盖默认的行为。

如果某些`mapped column`并未赋值，则`SQLAlchemy`会自动生成`default value`；对于那些已经赋值的`mapped column`，`SQLAlchemy`会自动跟踪这些赋值，这是为了将最终的`value`插入或者更新到数据库中

  ![mapped_class_instance](imgs/sqlalchemy/mapped_class_instance.JPG)
  
## 五、 Session

`ORM`系统是在`Session`中处理数据库操作的。通常在`create_engine()`的时候我们就会通过工厂方法定义一个`Session class`:

- `Session=sqlalchemy.orm.sessionmaker(bind=engine)`：绑定了`engine`
- `Session=sqlalchemy.orm.sessionmaker()`：未绑定`engine`
	- 之后可以通过`Session.configure(bind=engine)`再绑定`engine`

通常创建`Session`的实例，用该实例来操作数据库

  ![session](imgs/sqlalchemy/session.JPG)

### 1. insert 和 update

通过`Session.add()`方法可以添加`mapped class`实例。但是注意此时该实例是`pending`状态，还没有执行数据库插入操作。当下列行为之一发生时，才进行真正的数据库插入操作：

- 显式调用`Session.flush()`操作
- 查询数据库`mapped class`对应的表。此时先`flush`所有`pending`的实例执行数据库插入和更新，再执行查询操作
- 调用`Session.commit()`操作
> 你也可以通过`Session.add_all()`方法来添加一个`mapped class`实例的列表从而添加多个实例

在`ORM`思想中，一旦一个`mapped class`实例被添加到`Session`中，假设它的主键为`pk1`。则所有针对主键`pk1`的数据库查询操作都会返回该实例，而不是创建新的实例来返回。另外，如果你向该`Session`中添加另一个实例，而该实例的主键也是`pk1`则会抛出异常。

`Session`会跟踪那些被添加到它的那些`mapped class`实例。如果某个实例被修改过，则该修改并不是马上执行数据库的`update`操作，而是仅仅记录下来该实例被修改过。可以通过查看`Session.dirty`属性查看那些被修改过的`mapped class`实例。

  ![session_add](imgs/sqlalchemy/session_add.JPG)

### 2. commit

一旦执行了`Session.commit()`方法，那么`Session`中的那些实例就会推送至数据库中：

- 如果数据库中没有该主键的记录，则执行插入操作
- 如果数据库中有该主键的记录，且该`mapped class`实例为`dirty`，则执行更新操作
- 执行`commit transaction`操作

`commit`之后，该`Session`引用的连接资源会放回至连接池中。如果后面继续用该`Session`来操作数据库（如`add()`），则会开启一个新的`transaction`并且重新从连接池中取出连接

`commit`之后，那么之后在一个新的`transaction`中访问数据时，他会刷新数据（从数据库中获取）从而保持`mapped class`实例的最新的状态。

  ![session_commit](imgs/sqlalchemy/session_commit.JPG)

### 3. rolling back

由于`Session`是工作于`transaction`，你可以通过`Session.rollback()`执行回滚操作。

  ![session_rollback](imgs/sqlalchemy/session_rollback.JPG)

### 4. query

通过`Session.query()`方法能够创建一个`Query`对象。这个方法可以使用各种类型的参数，如`mapped class`或者`mapped column descriptor`（如`User`和`User.name`）。通常我们在迭代环境中使用`Query`对象。

- `for instance in session.query(User).order_by(User.id)`：迭代产生的是`User`类的实例
- `for instance in session.query(User.name, User.fullname).order_by(User.id)`：迭代产生的是一个元组
	> 只要`query()`参数为多个`mapped class`或者`mapped column descriptor`，或者它们的组合时，迭代就产生一个`named tuple`。这个元组可以当作普通的Python对象，其各个属性就是`mapped column descriptor`以及`mapped class`（属性名就是`mapped column descriptor`名字以及`mapped class`的类名）

  ![session_query](imgs/sqlalchemy/session_query.JPG)

#### a. 重命名

- 迭代`Query`对象产生命名元组时，你可以自定义这个命名元组的名字，通过在`query(User.name.label('my_name'))`，利用`label()`，则该命名元组有个`my_name`属性，对应的是`User`的`name`属性。
- 你可以通过`sqlalchemy.orm.aliased`来重命名一个`mapped class`。如`user_alias = aliased(User, name='user_alias')`，此后`user_alias`就可以代表`User`（对应于SQL语句中的`AS`表达式）

  ![session_query_rename](imgs/sqlalchemy/session_query_rename.JPG)

#### b. limit、 offset、 order_by

你可以对`Query`对象进行`Python array slice`来实现`SQL`的`LIMIT`和`OFFSET`，分片操作返回的是查询的结果数组。你也可以通过`Query.order_by()`方法来实现`ORDER BY`，其中`order_by()`方法的参数为`mapped column descriptor`（如`User.id`），该方法返回一个新的`Query`对象。
> `Query`对象的大多数方法都会返回一个新的`Query`对象

  ![session_query_limit_offset_order_by](imgs/sqlalchemy/session_query_limit_offset_order_by.JPG)


#### c. filter

如果你想对查询结果进行更复杂的筛选，则可以使用`Query.filter()`方法。该方法的参数可以对`mapped column descriptor`（如`User.id`）进行复杂的运算（比如相等比较，不等比较等）。
常用的`filter operator`有：

- 相等比较`equals`，如`query.filter(User.name == 'ed')`
- 不等比较`equals`，如`query.filter(User.name != 'ed')`
- `LIKE`，如`query.filter(User.name.like('%ed%'))`
- `IN`，如`query.filter(User.name.in_(['ed','wendy','jack']))`
	> 或者用`query.filter(User.name.in_(session.query(User.name).filter(User.name.like('%ed%'))))`
- `Not IN`，如`query.filter(~User.name.in_(['ed','wendy','jack']))`
- `IS NULL`，如`query.filter(User.name == None)`
	> 或者用`query.filter(User.name.is_(None))`
- `IS NOT NULL`,如`query.filter(User.name != None)`
	> 或者用`query.filter(User.name.isnot(None))`
- `AND`，有三种形式：
	- 利用`and_`函数：`query.filter(sqlalchemy.and_(User.name == 'ed', User.fullname == 'Ed Jones'))`（注意不是Python的`and`操作符）
	- 利用`filter`的多表达式：`query.filter(User.name == 'ed', User.fullname == 'Ed Jones')`
	- 利用`filter`链：`query.filter(User.name == 'ed').filter(User.fullname == 'Ed Jones')`
- `OR`：利用`or_`函数：`query.filter(sqlalchemy.or_(User.name == 'ed', User.fullname == 'Ed Jones'))`（注意不是Python的`or`操作符）
- `MATCH`：`query.filter(User.name.match('wendy'))`。该过滤器的行为会因数据库的不同而呈现不同的行为，其中`SQLite`不支持这种过滤。对于大多数数据库，该过滤器相当于`MATCH`或者`CONTAINS`


  ![session_query_filter](imgs/sqlalchemy/session_query_filter.JPG)

#### d. 返回标量和列表

有一些`Query`方法会立即执行`SQL`查询并且返回查询结果。

- `Query.all()`方法返回查询记录，结果是一个列表
- `Query.first()`，它执行的是`limit one`，返回一个标量值，该标量是第一条查询记录
- `Query.one()`，它首先获取所有的记录：
	- 如果有超过一条记录，则抛出`MultipleResultsFound`异常
	- 如果一条记录也没有，则抛出`NoResultFound`异常
	- 如果只有一条记录，则返回一个标量值，该标量就是查询到的记录
- `Query.one_or_none()`，它首先获取所有的记录：
	- 如果有超过一条记录，则抛出`MultipleResultsFound`异常
	- 如果一条记录也没有，则返回`None`
	- 如果只有一条记录，则返回一个标量值，该标量就是查询到的记录
- `Query.scalar()`，它返回记录的第一条数据作为结果返回
	- 如果有超过一条记录，则抛出`MultipleResultsFound`异常
	- 如果一条记录也没有，则返回`None`
	- 如果只有一条记录，则返回一个标量值，该标量就是查询到的记录

  ![session_query_result](imgs/sqlalchemy/session_query_result.JPG)

#### e. 通过字符串设定SQL

前面介绍的`Query.filter()`方法和`Query.order_by()`方法都用的是`mapped column descriptor`（如`User.id`），你也可以在这些地方使用字符串，这看起来更像是SQL。

通过`sqlalchemy.text()`函数中传入字符串字面量可以实现该做法。  
`session.query(User).filter(text("id<224")).order_by(text("id"))`等价于  `session.query(User).filter(User.id<224).order_by(User.id)`

当然`224`这个数字你也可以通过绑定参数来动态传入。这是通过`Query.params()`方法绑定的。  
`session.query(User).filter(text("id<:value and name=:name")).params(value=224, name='fred').order_by(User.id)`。参数是以`:key`字符串的形式实现的。在`params(key=value)`绑定的。

如果你想通过字符串指定完整的SQL查询语句，可以使用`Query.from_statement()`方法。  
`session.query(User).from_statement(text("SELECT * FROM users where name=:name")).params(name='ed')`

  ![session_query_with_text](imgs/sqlalchemy/session_query_with_text.JPG)

#### f. count

`Query.count()`方法执行的是SQL的`COUNT`，它用于确定能返回多少条查询记录。在`SQLAlchemy`内部，它首先执行一个子查询，然后在这个子查询的结果集之上在执行`COUNT`查询。这种实现方法比较低效，更高效的方法是`SELECT count(*) FROM table`。我们可以通过`sqlalchemy.func.count()`来实现这种高效的做法：

- `session.query(func.count(User.name)).all()`:返回的是一个列表，列表元素为元组，元组内容对应于`func.count()`的参数。
- `session.query(func.count(User.name)).scalar()`：返回的是标量，指定有多少个`User.name`
- `session.query(func.count('*')).select_from(User).scalar()`:返回的是标量，指定有多少条记录
	> 假设有100条记录，可能只有10个`User.name`，因为`User.name`可以重复
	>
	> 必须指定`.select_from(User)`，否则`Query`压根不知道要查询哪个表
- `session.query(func.count(User.id)).scalar()`:因为`id`是主键，因此主键有多少条，记录就有多少条

  ![session_query_count](imgs/sqlalchemy/session_query_count.JPG)

#### g. 外键
在`mapped class`的定义过程中，如果有外键，我们可以将该列指定为`ForeignKey`，如：

```
class Address(Base):
	__tablename__='addresses'
	...
	user_id=Column(Integer,ForeignKey('users.id'))
	...
```
其中`ForeignKey`的参数为字符串，指定了外键所在的表（`User.__tablename__`指定的）以及列名（`id`）

#### h. 关系
我们可以通过`sqlalchemy.orm.relationship()`定义两个`mapped class`之间的关系：一对多、多对一、多对多、一对一。该函数返回一个`RelationshipProperty`。你可以在`mapped class`类内定义关系，也可以在类外定义关系。如：

```
class Address(Base):
	__tablename__='addresses'
	...
	user_id=Column(Integer,ForeignKey('users.id'))
	user=relationship("User",back_populates='addresses')
	...
User.addresses=relationship("Address",back_populates='user',order_by=Address.id)
```

- 第一个位置参数为关联的`mapped class`类名
- `back_populates`参数：该参数指出了对方的关联名。如`Address`的关联名为`user`。那么你通过`User.addresses`获取的`Address`的`user`属性就指向本`User`。
- 关联结合外键可以自动确定是一对多关系/多对一关系。如例子中多个`Address`对应一个`User`。因此`User.addresses`是一个Python列表，而`Address.user`是一个`User`实例

  ![forienkey_relation](imgs/sqlalchemy/forienkey_relation.JPG)

#### i. 使用关系

默认情况下，`User.addresses`返回的是一个Python列表（你可以配置它为一个字典，或者一个`Python set`）。你可以修改`User.addresses`，在二元关系中，你针对某一端的实例修改之后，另一端的实例也会自动被修改从而匹配二元关系。

当我们查询`User`表的时候，并不会同时查询`Address`表；只有当访问`User.addresses`属性时，才会有`SQL`查询发生在`Address`表

  ![relation_usage](imgs/sqlalchemy/relation_usage.JPG)

#### j. join

通常我们可以使用`Query.filter()`来执行联合查询。它会隐式的执行 `JOIN`。如

```
session.query(User, Address).filter(User.id==Address.user_id).\
filter(Address.email_address=='jack@google.com').all()
```

你也可以显式的执行 `JOIN` 操作：如：`
session.query(User).join(Address).filter(Address.email_address=='jack@google.com').all()
`，这里隐含着一个条件，即`User`和 `Address`之间只有一个 `foreign key`。如果没有外键，或者有多个外键，则你必须采用下列方法：

- `query(User).join(Address,User.id==Address.user_id)`：显式指定条件
- `query(User).join(User.addresses)`：specify relationship from left to right
- `query(User).join(Address,User.addresses)`：同上，但是显式指定关系
- `query(User).join('addresses')`：同上，但是用字符串代替

当然你也可以用 `query(User).outerjoin(User.addresses)`来指定外连接

#### k. 别名
当使用 join 时，你必须指定多个表。但是有时候是表自己跟自己 join ，此时你必须使用别名。如：

```
adalias1 = aliased(Address)
adalias2 = aliased(Address)
for username, email1, email2 in \
	session.query(User.name, adalias1.email_address, adalias2.email_address).\
	join(adalias1, User.addresses).\
	join(adalias2, User.addresses).\
	filter(adalias1.email_address=='jack@google.com').\
	filter(adalias2.email_address=='j25@yahoo.com'):
 		print(username, email1, email2)
```

#### l. 子查询

如果我们想知道每个	`User`有几个`Address`，则我们可以使用子查询。方法为：

```
from sqlalchemy.sql import func
stmt = session.query(Address.user_id, func.count('*').\
 	label('address_count')). group_by(Address.user_id).subquery()
session.query(User, stmt.c.address_count).\
  outerjoin(stmt, User.id==stmt.c.user_id).order_by(User.id).all()
```

- `func`生成`SQL function`
- `Query.subquery()`方法生成一个 `SELECT statement`
- `stmt.c`是子查询的列属性。通过它可以引用子查询的各列
- 这里之所以用外连接，是因为可能有某些`User`没有 `Address`

如果子查询的结果是一条`mapped class`记录，则你要用别名机制将它放入父查询中：

```
stmt = session.query(Address).\
	filter(Address.email_address != 'j25@yahoo.com').subquery()
adalias = aliased(Address, stmt)
session.query(User, adalias).join(adalias, User.addresses).all()
```

#### m. exists

通常你可以在`Query.filter()`中使用 `exists`	 条件：

```
from sqlalchemy.sql import exists
session.query(User.name).filter(exists().where(Address.user_id==User.id)).all()
```	

也有一些操作会自动使用 `EXISTS` 语句。上述的代码可以替代为：

```
session.query(User.name).filter(User.addresses.any()).all():
```

这里 `any()`可以提供筛选条件，如`session.query(User.name).filter(User.addresses.any(Address.email_address.like('%google%')))`
如果是多对一关系，你可以使用`has()`，如：`session.query(Address).filter(~Address.user.has(User.name=='jack')).all()`

下面是常用的过滤：

- `query.filter(Address.user == someuser)`：多对一关系的相等比较
- `query.filter(Address.user != someuser)`：多对一关系的不相等比较
- `query.filter(Address.user == None)`：`IS NULL`, 多对一关系的比较
- `query.filter(User.addresses.contains(someaddress))`：一对多关系的包含过滤
- `query.filter(User.addresses.any(Address.email_address == 'bar'))`：一对多关系的包含过滤
- `query.filter(Address.user.has(name='ed'))`：多对一关系的包含过滤
- `session.query(Address).with_parent(someuser, 'addresses')`：用于任意关系的比较

#### n. 预加载

sqlalchemy 默认行为是懒加载，即：只有在必要的时候才执行 SQL 操作。当然如果你也可以预加载，即还没有用到的时候就提前执行SQL操作，这样的好处是可以在一条查询语句中返回多个查询值，从而减少查询的次数。

有三种预加载：两种是隐式的自动的行为、一种是显式指定。所有的这些预加载都是利用`Query.options()`方法指定。

##### 1> Subquery Load

通过在 `.options()`中指定 `orm.subqueryload()`，可以实现预加载。它利用一个子查询实现。
```
from sqlalchemy.orm import joinedload
jack = session.query(User).options(joinedload(User.addresses)).filter_by(name='jack').one()
```

##### 2>  Joined Load

通过`.options()`中指定 `orm.joinedLoad()`，可以实现预加载 。它利用一个 LEFT OUTER JOIN 来实现。
```
from sqlalchemy.orm import subqueryload
jack = session.query(User).options(subqueryload(User.addresses)).filter_by(name='jack').one()
```

##### 3> 显式 Join + Eagerload

显式利用 join 和 `orm.contains_eager()`实现预加载。

```
from sqlalchemy.orm import contains_eager
jacks_addresses = session.query(Address).join(Address.user).filter(User.name=='jack').\
	options(contains_eager(Address.user)).all()
```

### 5. 删除

通过 `session.delete(mapped_class_obj)` 可以删除一条记录。但是，如果该记录有外键关联，则它并不删除其他表中关联的记录。

你可以配置 `cascade` 来实现关联记录的删除行为：

```
class User(Base):
  __tablename__ = 'users'
	
  id = Column(Integer, primary_key=True)
  name = Column(String)
  fullname = Column(String)
  password = Column(String)	
  addresses = relationship("Address", back_populates='user',
  cascade="all, delete, delete-orphan")
...
class Address(Base):
  __tablename__ = 'addresses'
  id = Column(Integer, primary_key=True)
  email_address = Column(String, nullable=False)
  user_id = Column(Integer, ForeignKey('users.id'))
  user = relationship("User", back_populates="addresses")
```

这样当你 `session.delete(user1)`时，该`user1`对应的`address`记录会相应同步地被删除。

### 6. 多对多关系

定义一个多对多关系需要使用一个未绑定的`Table`对象作为中介。如：

- 定义一个未绑定 `Table`，通过 `Table()`显式构造：

```
from sqlalchemy import Table, Text
post_keywords = Table('post_keywords', Base.metadata,
Column('post_id', ForeignKey('posts.id'), primary_key=True),
Column('keyword_id', ForeignKey('keywords.id'), primary_key=True))
```

- 定义一端的 `mapped class`：

```
class BlogPost(Base):
  __tablename__ = 'posts' 
  id = Column(Integer, primary_key=True)
  user_id = Column(Integer, ForeignKey('users.id'))
  headline = Column(String(255), nullable=False)
  body = Column(Text) 
  # many to many BlogPost<->Keyword
  keywords = relationship('Keyword',secondary=post_keywords, back_populates='posts')
	...
```

- 定义另一端的 `mapped class`：

```
class Keyword(Base):
  __tablename__ = 'keywords'
 
  id = Column(Integer, primary_key=True)
  keyword = Column(String(50), nullable=False, unique=True)
  posts = relationship('BlogPost', secondary=post_keywords, back_populates='keywords')
	...
```

这样通过在 `relationship`中指定 `secondary`关键字参数（一个未绑定的 `Table`） 就能实现多对多关系。