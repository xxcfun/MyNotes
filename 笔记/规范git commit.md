## 规范git commit

* type：commit的类型
* feat：新特性
* fix：修改问题
* refactor：代码重构
* perf：性能优化
* docs：文档修改
* style：代码格式修改
* test：测试用例修改
* ci：持续集成方面的改动
* chore：其它修改，比如构建流程，依赖管理
* revert：回滚代码
* scope：commit影响的范围，比如：router，component，utils，build...
* subject：commit的概述
* body：commit具体修改内容，可以分为多行
* footer：一些备注

git commit -m "feat(order):添加order-list里面获取列表的功能"

git push -u origin master