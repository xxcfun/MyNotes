## Vue + Vant实现点击发送验证码倒计时的功能

![sendsms](D:\人生苦短，我用python\images\sendsms.png)

完成效果如上图所示，这里使用了Vant里面的输入框组件来完成前端的界面，用Vue来实现点击发送验证码后倒计时的功能。

**需求分析**

* 点击发送验证码按钮，出现（60秒）后重新发送
* 倒计时结束，按钮内容重置为发送验证码
* 手机号未填写，弹出请输入手机号的错误提示
* 在点击发送验证码按钮后，如果手机号改变，按钮重置

**前端界面搭建**

使用vant的输入框组件来完成界面的搭建，新建一个Send.vue文件，绘制前端页面。内容如下：

```vue
<template>
  <div class="div">
    <van-form>
      <van-field
        v-model="form.username"
        label="手机号码"
        placeholder="手机号码"
        type="tel"
        clearable
        maxlength="11"
      />
      <van-field
        v-model="form.sms_code"
        center
        clearable
        label="短信验证码"
        placeholder="请输入短信验证码"
      >
        <template #button>
          <van-button size="small" type="primary">点击发送验证码</van-button>
        </template>
      </van-field>
    </van-form>
  </div>
</template>

<script>
  export default {
    name: 'Send',
    data () {
      return {
        form: {
          username: '',
          sms_code: ''
        }
      }
    }
  }
</script>

<style lang="less">
  .div {
    padding-top: 50px;
  }
</style>
```

现在写完了界面，功能尚未编写，下面开始给发送验证码的按钮添加功能。

**重写组件绑定功能**

给按钮添加@click属性，添加sendSmsCode函数；

绑定disabled属性，点击发送验证码按钮后，让其按钮禁用，等倒数完后再恢复点击的功能；

“点击发送验证码”文本添加双向绑定，在点击后将文本替换为“（60秒）后重新发送”，倒计时结束后重新恢复文本为“点击发送验证码”。

```vue
<!-- van-button组件的标签替换下面内容 -->
<van-button size="small" type="primary" @click="sendSmsCode" :disabled="isSmsSend">{{ sendBtnText }}</van-button>
......
// data() 里面添加下面几个字段
data () {
  return {
    ...
    // 是否已经发送了验证码
    isSmsSend: false,
    // 文本
    sendBtnText: '点击发送验证码',
    // 计时器对象
    timer: null,
    // 倒数60秒
    counter: 60
  }
}
```

下面我们再编写sendSmsCode函数。

**编写sendSmsCode函数**

这里点击发送验证码按钮，先判断下手机号是否已经输入和是否符合规则，不符合判断条件，则通知用户相应的错误。

![error1](D:\人生苦短，我用python\images\error1.png)![error2](D:\人生苦短，我用python\images\error2.png)

```vue
<script>  
  export default {   
    methods: {
      /**
       * 发送验证码
       */
      sendSmsCode () {
        // 判断手机号是否已经输入
        if (!this.form.username) {
          this.$notify('请输入手机号')
          return false
        }
        // 判断手机号是否符合要求
        if (this.form.username.length !== 11) {
          this.$notify('请输入11位手机号')
          return false
        }
        // 调用接口，发送短信验证码
        // 这部分放调用发送短信的接口，这里先不做任何功能，主要先把按钮倒计时的功能实现
        // 将按钮禁用，防止再次点击
        this.isSmsSend = true
        // 开始倒计时，60s之后才能再次点击
        this.countDown()  // 这里实现倒计时的功能，文章下面开始介绍
      }
    }
  }
</script>
```

**编写countDown倒计时函数**

倒计时的功能使用了window里面的setInterval()方法和clearInterval()方法。setInterval() 方法可按照指定的周期（以毫秒计）来调用函数或计算表达式；clearInterval() 方法可取消由 setInterval() 函数设定的定时执行操作。

相关具体讲解可见菜鸟教程：[setInterval()方法](https://m.runoob.com/jsref/met-win-setinterval.html)、[clearInterval()方法](https://www.runoob.com/jsref/met-win-clearinterval.html)

```vue
<script>  
  export default {   
    methods: {
      /**
       * 倒计时
       */
      countDown () {
        // 将setInterval()方法赋值给前面定义的timer计时器对象，等用clearInterval()方法时方便清空这个计时器对象
        this.timer = setInterval(() => {
          // 替换文本，用es6里面的``这个来创建字符串模板，让秒实时改变
          this.sendBtnText = `(${this.counter}秒)后重新发送`
          this.counter--
          if (this.counter < 0) {
            // 当计时小于零时，取消该计时器
            clearInterval(this.timer)
          }
        }, 1000)
      },
      /**
       * 发送验证码
       */
      sendSmsCode () {
        ...此处省略相关代码
      }
    }
  }
</script>
```

这样倒计时的功能基本完成，但存在一个问题，倒计时完成后，按钮没有重置，所以还得写一个重置倒计时的函数。

**编写重置函数**

在倒计时结束时，重置计时器对象，重置按钮变可用，重置文本。

```vue
<script>  
  export default {   
    methods: {
      /**
       * 重置倒计时
       */
      reset () {
        // 重置按钮可用
        this.isSmsSend = false
        // 重置文本内容
        this.sendBtnText = '点击发送验证码'
        if (this.timer) {
          // 存在计时器对象，则清除
          clearInterval(this.timer)
          // 重置秒数，防止下次混乱
          this.counter = 60
          // 计时器对象重置为空
          this.timer = null
        }
      },
      /**
       * 倒计时
       */
      countDown () {
        ...
          if (this.counter < 0) {
            // 当计时小于零时，取消该计时器
            // 在这里调用重置函数
            this.reset()
          }
        ...
      },
      /**
       * 发送验证码
       */
      sendSmsCode () {
        ...此处省略相关代码
      }
    }
  }
</script>
```

**监听手机号输入框事件**

最后，在第一次点击按钮倒计时开始后，如果改变手机号的话，得重置该按钮，这样在输入框中绑定onPhoneChange函数，让其监听后调用充值方法，这里vant组件中，有个input属性就是当输入框内容改变时触发事件，所以，我们再次修改如下：

```vue
<template>
  <div class="div">
    <van-form>
      <van-field
        v-model="form.username"
        label="手机号码"
        placeholder="手机号码"
        type="tel"
        clearable
        maxlength="11"
        <!--添加这一行-->
        @input="onPhoneChange"
      />
    </van-form>
  </div>
</template>
```

然后在methods中完善该方法：

```vue
<script>  
  export default {   
    methods: {
      ...
      // 当手机号变化时，重置发送按钮
      onPhoneChange () {
        this.reset()
      }
      ...省略其它代码
    }
  }
</script>
```

这样，功能就完成了，下面贴上所有代码以供参考

**全部代码**

```vue
<template>
  <div class="page-account-register">
    <van-form>
      <van-field
        v-model="form.username"
        label="手机号码"
        placeholder="手机号码"
        type="tel"
        clearable
        maxlength="11"
        @input="onPhoneChange"
      />
      <van-field
        v-model="form.sms_code"
        center
        clearable
        label="短信验证码"
        placeholder="请输入短信验证码"
      >
        <template #button>
          <van-button size="small" type="primary" @click.prevent="sendSmsCode" :disabled="isSmsSend">{{ sendBtnText }}</van-button>
        </template>
      </van-field>
    </van-form>
  </div>
</template>

<script>
  export default {
    name: 'Send',
    data () {
      return {
        form: {
          username: '',
          sms_code: ''
        },
        // 是否已经发送了验证码
        isSmsSend: false,
        sendBtnText: '点击发送验证码',
        timer: null,
        counter: 60
      }
    },
    methods: {
      /**
       * 重置倒计时
       */
      reset () {
        this.isSmsSend = false
        this.sendBtnText = '点击发送验证码'
        if (this.timer) {
          clearInterval(this.timer)
          this.counter = 60
          this.timer = null
        }
      },
      /**
       * 倒计时
       */
      countDown () {
        this.timer = setInterval(() => {
          this.sendBtnText = `(${this.counter}秒)后重新发送`
          this.counter--
          if (this.counter < 0) {
            this.reset()
          }
        }, 1000)
      },
      /**
       * 发送验证码
       */
      sendSmsCode () {
        // 判断手机号是否已经输入
        if (!this.form.username) {
          this.$notify('请输入手机号')
          return false
        }
        // 判断手机号是否符合要求
        if (this.form.username.length !== 11) {
          this.$notify('请输入11位手机号')
          return false
        }
        // 调用接口，发送短信验证码
        this.isSmsSend = true
        // 开始倒计时，60s之后才能再次点击
        this.countDown()
      },
      // 当手机号变化时，重置发送按钮
      onPhoneChange () {
        this.reset()
      }
    }
  }
</script>

<style lang="less">
  .page-account-register {
    padding-top: 50px;
  }
</style>

```


***

*如需转载，请加上本文的链接并标明出处*

*[一条爱吃屎的狗：https://www.qnmlgb.top](https://www.qnmlgb.top/)*