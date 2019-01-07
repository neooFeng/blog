## 点击相关的事件
* touchstart
* touchend
* mousedown
* mouseup
* click

### Why so much events?
1. click仅当mouseDown并且mouseUp后触发，而且如果mouseDown和mouseUp所属element不同，则click事件会在mouseDown和mouseUp所属element的最近共同祖先element触发（这就是为什么当我们用鼠标按下一个按钮然后后悔了的时候，只要继续按住鼠标并移到按钮外松开就好了的原因）
2. mouseDown/mouseUp是及时触发，且鼠标左右中键都可以触发，可以用来激活context menu，此外还可以监听mouseMove的开始与结束。有时需要同时处理某一个元素的鼠标移动开始/结束事件和单击事件，所以不能用mouseDown/mouseUp替代click
3. touch的引入是因为移动设备的普及，其操作需要支持多点触控，且touch不需要over/out、enter/leave等事件


### 触发顺序
* 移动端：
  > touchstart->touchend->mousedown->mouseup->click
* pc端： 
  > mousedown->mouseup->click

### 常见问题：移动浏览器click事件延迟
> http://thx.github.io/mobile/300ms-click-delay/


### 常见问题：click事件穿透
> http://www.cnblogs.com/zldream1106/p/3670988.html


## 移动相关事件
* touchmove
* mouseenter
* mouseover
* mouseout
* mouseleave
* mousemove
* drag event

### 触发顺序
* 移动端，滑动： 
  > touchstart->touchmove->touchend
* pc端，按住移动：
  > mousedown->mousemove->mouseup->click
* pc端，非按住移动：
  > mousemove

### mouseenter/mouseleave vs mouseover/mouseout
mouseenter/mouseleave不冒泡。

### 应用举例：drag and drop


## 浏览器兼容性总结
* pc浏览器不支持touch, 手机浏览器基本支持touch
* IE10+才支持drag event, 手机浏览器基本不支持
* iPhone浏览器中，监听mouseenter/mouseover/mousemove/mouseleave/mouseout事件后不能触发mousedown/mouseup/click


## 事件默认行为

### 应用举例：禁用UC/QQ等国产浏览器左划返回上一页特性


## 事件冒泡

### 应用举例：点击'删除'时，禁止自动选中所在条目