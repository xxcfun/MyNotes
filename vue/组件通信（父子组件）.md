##  通过tabbar案例来学习组件通信（父传子/子传父）

父组件传递给子组件：通过props属性； 

子组件传递给父组件：通过$emit触发事件；

![tabbar](https://qnmlgb.top/media/editor/tabbar_20220617152819042002.png)

##### 父传子

通过props属性

```javascript
""" 父组件 """

<template>
  <div>
    <tab-bar :tabs="tabs"/>  // 绑定tabs传给子组件
  </div>
</template>

<script>
  import TabBar from "./tab-bar";
  export default {
    name: "App",
    components: {TabBar},
    data() {
      return {
        tabs: ["推荐", "歌单", "我的"]
      }
    }
  }
</script>
```

```javascript
""" 子组件 """

<template>
  <div class="tab-bar">
    <div
      class="tab-item"
      :class="{active: currentIndex === index}"
      v-for="(tab, index) in tabs"
      :key="tab.index"
    >
      <span>{{tab}}</span>
    </div>
  </div>
</template>

<script>
  export default {
    name: "tab-bar",
    props: {  // 使用props属性接收父组件传来的tabs
      tabs: Array,
      default() {
        return []
      }
    },
    data() {
      return {
        currentIndex: 0
      }
    }
  }
</script>

<style scoped>
  .tab-bar {
    margin-bottom: 30px;
    display: flex;
  }
  .tab-item {
    flex: 1;
    text-align: center;
  }
  .tab-item.active {
    color: red;
  }
  .tab-item.active span{
    border-bottom: 3px solid red;
    padding: 5px 10px;
  }
</style>
```

父组件里面的数据，需要子组件来进行展示，这个时候我们通过props来完成组件之间的通信。

在子组件中用props来定义好要接收的数据，props有两种常见的用法：一个是使用字符串数组；另一个是使用对象类型，我这里使用了对象类型。如上子组件需要接收tabs数据。

这时父组件在引用这个子组件的同时，将自身组件中的tabs使用v-bind绑定的方法传给子组件，这样props属性里面就能接收到tabs。

##### 子传父

通过$emit触发事件

```javascript
""" 子组件 """
<template>
  <div class="tab-bar">
    <div
      class="tab-item"
      :class="{active: currentIndex === index}"
      v-for="(tab, index) in tabs"
      :key="tab.index"
      @click="selectItem(index)"  // 添加一个点击事件
    >
      <span>{{tab}}</span>
    </div>
  </div>
</template>

<script>
  export default {
    name: "tab-bar",
    emits: ["clickItem"],  // 这里触发emits里的clickItem事件
    props: {
      tabs: Array,
      default() {
        return []
      }
    },
    data() {
      return {
        currentIndex: 0
      }
    },
    methods: {  // 通过子组件发生点击事件来触发对应的事件
      selectItem(index) {
        this.currentIndex = index
        this.$emit("clickItem", index)  // 这里通过emit给父组件触发事件并携带index参数
      }
    }
  }
</script>

```

```javascript
""" 父组件 """

<template>
  <div>
    <tab-bar :tabs="tabs" @clickItem="clickItem"/>  // 这里监听事件名称，并且绑定到对应的方法中
    <div>{{content[currentIndex]}}</div>  // 显示内容
  </div>
</template>

<script>
  import TabBar from "./tab-bar";
  export default {
    name: "App",
    components: {TabBar},
    data() {
      return {
        tabs: ["推荐", "歌单", "我的"],
        content: ["推荐列表", "歌单列表", "关于我的界面"],
        currentIndex: 0  // 根据currentIndex的值切换界面内容
      }
    },
    methods: {  // 方法实现
      clickItem(index) {
        this.currentIndex = index
      }
    }
  }
</script>
```

这里通过子组件触发一个$emit("clickItem")事件，父组件使用@clickItem来监听子组件触发的事件，在子组件发生selectItem点击事件时触发对应的clickItem事件来完成子传父的通信。

在父组件监听到子组件发来的事件后，绑定到自身的clickItem方法中，通过子组件触发而来的index参数来改变父组件里面的界面内容。



~END

