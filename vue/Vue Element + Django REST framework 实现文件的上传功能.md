##  Vue Element + Django REST framework 实现文件的上传功能

文件上传作为开发中经常使用的功能，本篇文章意在将此功能封装成一个组件，在有需要的时候直接拿来使用。

##### 逻辑思路

* 用户上传文件，文件自动提交
* 文件提交后返回文件的url路径
* 将返回来的url路径随其它字段一块提交给后台。
* 后台拿到请求的数据，保存到数据库中。

##### 后台接口设计

* model设计

这里model我们只需要上传文件，所以直接编写一个字段信息即可。

```python
class FilesModel(models.Model):
    """ 文件上传 """
    file = models.FileField(upload_to='uploads/')

    class Meta:
        db_table = 'files'
        ordering = ['-id']
```

* settings

这里面settings得添加文件根路径的设置。

```python
# 媒体文件设置
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
```

这样以后所有上传的文件都保存在根目录的“/media/uploads/”路径下。

* serializer

后端用的是 Django REST framework 来实现后端的REST API，需要进行序列化。

```python
class FilesSerializer(serializers.ModelSerializer):
    """ 文件上传专用 """
    class Meta:
        model = FilesModel
        fields = '__all__'
```

* views

views试图代码如下：

```python
class FileViewset(viewsets.ModelViewSet):
    """ 文件上传 """
    authentication_classes = (JSONWebTokenAuthentication, SessionAuthentication)
    queryset = FilesModel.objects.all()
    serializer_class = FilesSerializer
```

* urls

填写路由配置

```python
router = DefaultRouter()
router.register(r'files', views.FileViewset, basename='files')

urlpatterns = [
    re_path('^', include(router.urls))
]
```

* test

运行django项目，访问 http://127.0.0.1:8000/files/ 进行测试<img src="D:\MyNotes\images\file.png" alt="file" style="zoom:60%;" />

##### 前端界面

这里使用 Element-UI 里面的 Upload 上传组件

* 文件上传组件

新建 `FileUpload.vue` 文件来封装文件上传的组件。

```javascript
<template>
  <el-upload
    class="upload"
    :action="postUrl()"
    :on-success="getFilePath"
    :on-remove="removeFile"
    :limit="limit"
    :on-exceed="exceedFile"
    :file-list="fileList">
    <el-button size="small" type="success">点击上传</el-button>
    <div slot="tip" class="el-upload__tip">只支持上传.pdf格式的文件；只允许上传1个文件，如上传错误请先移除再进行上传操作。</div>
  </el-upload>
</template>

<script>
  export default {
    name: 'FileUpload',
    data () {
      return {
        fileList: [],
        limit: 1,
        filePath: ''
      }
    },
    methods: {
      // 文件上传相关
      // 上传文件的地址
      postUrl () {
        return 'http://127.0.0.1:8000/files/'
      },
      // 上传成功的狗子  获取文件上传的路径
      getFilePath (response) {
        this.$message({
          message: '上传成功',
          type: 'success'
        })
        // 获取地址
        this.filePath = response.file
        // 子组件传值给父组件
        this.$emit('filePath', this.filePath)
      },
      // 移除文件的钩子
      removeFile (file) {
        this.$message({
          message: `成功移除${file.name}`,
          type: 'success'
        })
        console.log(file)
      },
      // 文件超出数量钩子
      exceedFile (file) {
        this.$message({
          message: '上传失败，只能上传一个文件',
          type: 'warning'
        })
        console.log(file)
      }
    }
  }
</script>
```

参数详解

`action`：上传的地址，也就是后端上传文件的接口地址，这里通过 `postUrl` 函数来获取。

`on-success`：文件上传成功的钩子，当文件上传成功时，返回该文件的地址，并通过子组件传父组件的方式返回给父组件。

`on-remove`：文件移除时的钩子，当文件移除时，页面提示用户文件已经移除。

`limit`：文件最大允许上传的个数，我们这个场景里限制只传1个。

`on-exceed`：文件上传个数限制的钩子，上面限制一个文件，再次上传提示上传失败。

`file-list`：上传的文件列表。

这样，一个文件上传的组件封装完成。

* 组件使用

在需要使用文件上传功能的地方直接引用组件，并给组件绑定一个函数来获取组件里面返回的文件url路径。

```javascript
<template>
  <div>
    ...
    <file-upload @filePath="getFilePath"/>
    ...
  </div>
</template>

<script>
  import FileUpload from '../../components/file/FileUpload'

  export default {
    components: { FileUpload },
    data () {
      return {
        ...
        file: '',
        ...
      }
    },
    methods: {
      // 父组件接收文件路径
      getFilePath (data) {
        this.file = data
        console.log(this.file)
      },
      ...
    }
  }
</script>
```

这样file即为文件上传的地址，后面使用的时候直接调用file的值即可。

