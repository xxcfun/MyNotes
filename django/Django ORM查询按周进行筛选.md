## Django ORM查询按周进行筛选

在Django ORM操作中，我们可以对时间字段按年月日的类型快速筛选，但是django里面没有内置按周进行筛选，所以，我们在这里使用isoweekday函数来进行操作。

先说一下isoweekday的作用，函数返回的值是1-7，分别代表周一到周日 。拿到这个数值后就很方便计算出一周开始的日期。废话不多说，贴上代码如下：

```python
...
# 导入时间处理函数
import datetime

# 用这个函数来举例
def week(request):
    # 现在的时间
	now_time = datetime.datetime.now()
    # 距离周日相隔的天数，这里得到int型数值
    day_num = now_time.isoweekday()
    # 查周日的日期，现在时间减去相隔天数得出周日的日期
    week_day = (now_time - datetime.timedelta(days=day_num))
    # 改格式，将datetime类型转换为date类型
    monday = week_day.date()
    # orm查一周内的数据
    all = Customer.objects.filter(created_at__range=(monday, now_time))
    return HttpResponse(all)
```

这个timedelta函数的作用是计算两个时间的差值，在上面代码中是为了把day_num这个int型数值转变为datetime类型。这个timedelta也可以用来计算明天或者昨天的日期。



***

> *如需转载，请加上本文的链接并标明出处*
>
> *[一条爱吃屎的狗：https://www.qnmlgb.top](https://www.qnmlgb.top/)*