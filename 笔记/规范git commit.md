## 规范git commit

##### 规范

* feat：新特性；新功能
* fix：修改问题；修复bug
* refactor：代码重构；既不是新增功能，也不是修改bug的代码变动
* perf：性能优化
* docs：文档修改
* style：代码格式修改；不影响代码运行的改动
* test：测试用例修改
* chore：构建过程或辅助工具的变动；比如构建流程，依赖管理
* revert：回滚代码

##### 用例

```markdown
git commit -m "feat(order): 添加order-list里面获取列表的功能"
git push -u origin master
```

