## 聊一聊Django RestFramework框架的视图组件

#### 前述

RestFramework是基于Django实现RESTful API的框架，也是在前后端分离的领域中诞生的。之前写过RESTful API风格接口规范，感兴趣的可以去读一读我前面的文章，这次重点来聊一聊RestFramework。

#### 1.正常视图类

先用我们正常思路写出增删改查查5个处理方法。

**url**

```python
path('orders/', views.OrderList.as_view()),
path('orders/<int:order_id>/', views.OrderDetail.as_view())
```

**view**

```python
""" order表的ModelSerializer类 """
class OrderListSerializer(serializers.ModelSerializer):
	
    class Meta:
		model = model.Order
		fields = '__all__'
        
        
""" order表的查找所有信息和添加信息操作 """
class OrderList(APIView):
    
    def get(self, request):
        order_obj = models.Order.objects.all()
        os = OrderListSerializer(order_obj, many=True)
        data = os.data
        return Response(queryset)
    
    def post(self, request):
        os = OrderListSerializer(data=request.data, many=False)
        if os.is_valid():
            os.save()
            return Response(os.data)
        else:
            return Response(os.errors)
        

""" order表的删除信息、修改信息和查找单条信息操作 """
class OrderDetail(APIView):
    
    def delete(self, request, id):
        models.Order.objects.get(pk=id).delete()
        return Response("删除成功")
    
    def put(self, request, id):
        order_obj = models.Order.objects.get(pk=id)
        os = OrderListSerializer(data=request.data, instance=order_obj)
        if os.is_valid():
            os.save()
            return Response(os.data)
        else:
            return Response(os.errors)
        
	def get(self, request, id):
        order_obj = models.Order.objects.get(pk=id)
        os = OrderListSerializer(order_obj, many=False)
        return Response(os.data)
        
```

这样就写完了，先让我们看看有什么问题，好像除了代码多点也没啥问题。这是一张表，如果要处理10张表，每张表都要写相同的功能，那这个工作量就非常的巨大了，本着python减少代码量的初心，开始对这个代码来做优化。

#### 2.mixins的使用

在RestFramework中，mixins已经为我们封装好了这5种方法。

```python
""" order表的ModelSerializer类 """
class OrderListSerializer(serializers.ModelSerializer):
	
    class Meta:
		model = model.Order
		fields = '__all__'
        

""" order表的查找所有信息和添加信息操作 """
class OrderList(ListModelMixin, CreateModelMixin, GenericAPIView):
	queryset = models.Order.objects.all()
    serializer_class = OrderListSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
    
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
    
    
""" order表的删除信息、修改信息和查找单条信息操作 """
class OrderDetail(UpdateModelMixin, DestroyModelMixin, RetrieveModelMixin, GenericAPIView):
    queryset = models.Order.objects.all()
    serializer_class = OrderListSerializer
    
    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)
    
    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
    
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)
        
```

贴一张表格来说明下这个被封装好的处理类，直接继承然后调用，这样比前面方便了许多。

| mixins             | 作用                                 | 对应http的请求方法 |
| ------------------ | ------------------------------------ | ------------------ |
| ListModelMixin     | 定义list方法，返回一个queryset的列表 | get                |
| CreateModelMixin   | 定义create方法，创建一个实例         | post               |
| RetrieveModelMixin | 定义retrieve方法，返回一个具体的实例 | get                |
| UpdateModelMixin   | 定义update方法，对某个实例进行更新   | put/patch          |
| DestoryModelMixin  | 定义delete方法，删除某个实例         | delete             |

GenericAPIView是RestFramework重装的APIView类，GenericAPIView在继承了APIView的同时又封装了一些新功能。

用mixins确实方便了不少，但里面还是有重复的代码，下面用generics继续封装继续给代码瘦身。

#### 3.generics的使用

generics中的ListCreateAPIView和RetrieveUpdateDestroyAPIView类分别封装了增查、改删查的方法。

```python
""" order表的ModelSerializer类 """
class OrderListSerializer(serializers.ModelSerializer):
	
    class Meta:
		model = model.Order
		fields = '__all__'

        
""" order表的查找所有信息和添加信息操作 """
class OrderList(generics.ListCreateAPIView):
    queryset = models.Order.objects.all()
    serializer_class = OrderListSerializer


""" order表的删除信息、修改信息和查找单条信息操作 """
class OrderDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = models.Order.objects.all()
    serializer_class = OrderListSerializer
    
```

这次又简化了不少，视图中仅仅包含了queryset和serializer_class两个各自类持有的属性，这样做就将类的共用代码通过类的继承来实现想要的功能，提高了代码的复用性。

细看一下，这两个类还是有共同的代码，还可以继续简化。

#### 4.ModelViewSet的使用

**url修改**

```python
path('orders/', views.OrderList.as_view())
```

**view**

```python
""" order表的ModelSerializer类 """
class OrderListSerializer(serializers.ModelSerializer):
	
    class Meta:
		model = model.Order
		fields = '__all__'


""" order表的查找所有信息、添加信息、删除信息、修改信息和查找单条信息操作 """
class OrderList(ModelViewSet):
    queryset = models.Order.objects.all()
    serializer_class = OrderListSerializer

```

这就是最牛逼的封装。

但高封装性也有自身的缺点，就是代码的灵活性随着一层层的封装变的越来越差，显然，最简单的不一定是最适合的，还是得让开发者根据自己的需求来选择最合适的封装方式。

#### 最后贴一张Django REST Framework View的图谱

![图谱](https://qnmlgb.top/media/editor/tupu_20220117173450653258.png)

