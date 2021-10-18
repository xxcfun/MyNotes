##  长期更新，js工具

##### 格式化时间

场景使用：视频、音频的时间计算；

举例：formatTime(4500)；输出：01:15:00；

```javascript
export function formatTime (interval) {
    // 向下取整
    interval = interval | 0
    // 拿到小时
    const hour = ((interval / ( 60 * 60 ) | 0) + '').padStart(2, '0')
    if (hour) {
      interval = interval - ( hour * 60 * 60 )
    }
    // 拿到分钟
    const minute = ((interval / 60 | 0) + '').padStart(2, '0')
    // 拿到秒
    const second = (interval % 60 + '').padStart(2, '0')
    return `${hour}:${minute}:${second}`
  }
```