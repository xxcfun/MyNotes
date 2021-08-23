## python操作SQL server

用python来操作SQL server需要安装pymssql，在你的虚拟环境中安装

```python
pip install pymssql
```

安装完成后需要在python文件中导入这个库

```python
import pymssql
```

创建数据库连接，如果数据里的编码为utf8，这里的charset也得是相应的utf8，不然会出现中文乱码的情况，如果是GBK编码，下面就改为GBK即可。

```python
conn = pymssql.connect(
	host = '主机名',
	user = '用户名',
	password = '密码',
	database = '数据库名称',
	charset = 'utf8'
    # chartset = 'GBK'
)
```

创建一个游标，注意，这里cursor后面不要忘记()，不然程序会报错

```python
cursor = conn.cursor()
```

编写sql

```python
sql = '需要操作的sql语句'
```

使用游标来进行操作、

```python
cursor.execute(sql)
results = ''	# 定义结果集为空
results = cursor.fetchall()		# 使用fetchall()函数返回查询的所有结果
print(results)
```

最后关闭数据链接

```python
conn.close()
```

所有示例代码如下

```python
import pymssql

conn = pymssql.connect(
    host = '127.0.0.1',
    user = 'sa',
    password = 'admin123456',
    database = 'database',
    charset = 'utf8'
)

cursor = conn.cursor()

sql = "select * from list where DATEDIFF(day,Date,GETDATE())=0"

cursor.execute(sql)

results = ''
results = cursor.fetchall()
print(results)

conn.close()

```

到此Python链接sqlserver完成，如有不当之处，欢迎指正，谢谢！！！



***

> *如需转载，请加上本文的链接并标明出处*
>
> *[一条爱吃屎的狗：https://www.qnmlgb.top](https://www.qnmlgb.top/)*
