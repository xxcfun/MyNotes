## 学了三天的OpenCV能干点什么

老板：最近维护的项目差不多可以了，你去搞搞那个什么计算机视觉，正好你会点python，去看看能搞点什么，今年咱们能和百度一起合作一块干点项目。

我：这东西没接触过，可能对我有点难。

老板：给你研发资金。

我：那我搞搞看？

##### 场景

自己看了三天博客，了解了一些常用的api，看看把这些api综合起来，先入个门，做个车流量检测的小程序，等后面会机器学习了再建模训练模型。

缺少摄像头，老板：公司仓库门口有个，找个梯子给它薅下来。

缺少电源线，老板：买新的；

缺少网线，老板：买新的；

缺少个好用的电脑，老板：下面有个11代i7+3060，你拿上来用吧。

##### 三天后

首先上个成果图片，没个框就是一辆运动的车辆，由宽高求出每辆车的中心点，如果这个中心点过了中间的蓝线，就将车流量+1，由此来实现对车流量的检测。

当然公司在13楼，摄像头拍出来太模糊了、、、

想看源码的往下拉，文章的最底部我也放了我学习opencv的时候练习的代码仓库。

![opencv](https://qnmlgb.top/media/editor/opencv_20220315165841346902.png)

##### 程序分析

现在开始干货了。

这个小程序首先得有个思路，如下：

* 通过摄像头读取视频
  * 通过 `VideoCapture` 这个api加载摄像头视频源，摄像头一般用的都是rtsp码流
* 将视频的静态背景全部去除
  * 首先将视频用 `cvtColor` 转为灰度视频
  * 用 `GaussianBlur` 高斯滤波去噪，解决高斯噪点
  * 用 `getStructuringElement` 去除背景
  * 这样得到个黑白图像，黑为静，白为动
* 用形态学处理车辆
  * 为了追求更好的效果，使用 `erode` 对图像进行腐蚀，去除整个画面中的小斑点
  * 由于腐蚀完的图像像素变小，所以再用 `dilate` 进行多次放大
  * 剩下车辆物体内部的小斑点，用 `morphologyEx` 进行闭操作，去除移动物体内部的小块
  * 这样经过多次处理后，车辆就变成了一个个的小白块，有利于后面画车辆的轮廓
* 描绘车辆轮廓
  * 描绘前先进行查找轮廓，使用 `findContours` 进行查找，按树形存储轮廓，并且只保存角点
  * 然后再进行一次过滤，以此来验证有效的车辆信息，先前面定义一个宽高常量用来过滤
  * 过滤后有效的车辆就用 `rectangle` 进行绘制，在原来frame图像上绘制，当然位置和宽高前面得用 `boundingRect` 拿到
  * 有了车辆的轮廓框，顺便获取到车辆的中心点，源代码中使用了 `center()` 这个函数
  * 最后将每辆车的中心点存入 `cars[]` 数组，并用 `circle` 画圆心的api来绘制车辆的中心点
* 对车辆进行逻辑处理并显示信息
  * 进行逻辑处理前，先定义图像中的那条蓝线，每有一个中心点通过此蓝线，就将车辆总数+1，并将此点从上面的 `cars[]` 数组中移除
  * 画线使用的是 `line` api，车辆统计显示的文字使用 `putText` api
  * 由此程序完成，在退出程序时不要忘了释放资源

##### 代码实现

```python
# 首先装好 python-opencv 依赖包
# 注释不少，我相信聪明的你肯定会理解的
import cv2
import numpy as np

# 最小高度和宽度
min_w = 60
min_h = 40
# 车辆数组
cars = []
# 检测线高
line_width = 300
# 线的偏移量
offset = 7
# 初始化车辆总数
carnum = 0


# 求中心点
def center(x, y, w, h):
    x1 = int(w/2)
    y1 = int(h/2)
    cx = x + x1
    cy = y + y1

    return cx, cy


# 读视频 960*1280 当然这里也可以引用本地视频文件，参数就是本地视频的路径
cap = cv2.VideoCapture('rtsp://admin:Aa123456@192.168.1.84/H264?ch=1&subtype=0')

bgsubmog = cv2.createBackgroundSubtractorMOG2()

# 形态学kernel
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))

while True:
    ret, frame = cap.read()

    if ret:
        # 灰度
        cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        # 去噪
        blur = cv2.GaussianBlur(frame, (3, 3), 5)
        # 去背景
        mask = bgsubmog.apply(blur)

        # 腐蚀 去掉图像中的小斑块
        erode = cv2.erode(mask, kernel)

        # 膨胀 还原放大
        dilate = cv2.dilate(erode, kernel, iterations=3)

        # 闭操作 去掉物体内部的小块
        close = cv2.morphologyEx(dilate, cv2.MORPH_CLOSE, kernel)
        close = cv2.morphologyEx(close, cv2.MORPH_CLOSE, kernel)
        close = cv2.morphologyEx(close, cv2.MORPH_CLOSE, kernel)

        # 查找轮廓
        contours, h = cv2.findContours(close, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        # 画线 从左侧画一条检测线
        cv2.line(frame, (line_width, 600), (line_width, 880), (255, 255, 0), 3)

        # for循环画轮廓
        for (i, c) in enumerate(contours):
            (x, y, w, h) = cv2.boundingRect(c)

            # 对车辆进行过滤，以验证是否是有效的车辆
            isValid = (w >= min_w) and (h >= min_h)
            if not isValid:
                continue

            # 到这里都是有效的车辆
            # 将车被标记出来
            cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 0, 255), 2)
            # 拿到车的中心点cpoint
            cpoint = center(x, y, w, h)
            # 将点存入车辆数组
            cars.append(cpoint)
            # 画出车的中心点
            cv2.circle(frame, (cpoint), 5, (0, 0, 255), -1)

            for (x, y) in cars:
                # 要有一条线，有范围，车辆经过后减去
                if (x > line_width - offset) and (x < line_width + offset):
                    carnum += 1
                    cars.remove((x, y))

        cv2.putText(frame, "Cars Count:" + str(carnum), (800, 100), cv2.FONT_HERSHEY_SIMPLEX, 2, (255, 255, 255), 3)
        # cv2.imshow('video', close)
        cv2.imshow('origin', frame)

    key = cv2.waitKey(1)
    # 27就是键盘左上角的ESC键
    if key == 27:
        break

# 释放资源
cap.release()
cv2.destroyAllWindows()

```

##### 总结

里面很多api对于初学者来看不算友好，可以看我学习代码仓库，里面有讲解这些api的代码，学习opencv也不算难，难点是怎么用好里面的api。

学了三天，总算有点收获，里面最多的还是api的学习运用，目前还没有接触机器学习、特征点匹配检测、图像分割修复的内容。等后面还有实战的小程序，我也会继续分享思路和代码。目前我也很菜，就一起共勉一起学习了。

##### 仓库地址

学习代码仓库：[https://github.com/xxcfun/learnOpenCV](https://github.com/xxcfun/learnOpenCV)

车量检测代码：[https://github.com/xxcfun/learnOpenCV/blob/master/10目标识别/cars.py](https://github.com/xxcfun/learnOpenCV/blob/master/10%E7%9B%AE%E6%A0%87%E8%AF%86%E5%88%AB/cars.py)