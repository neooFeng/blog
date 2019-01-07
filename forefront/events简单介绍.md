## 事件默认行为

### 应用举例：禁用UC/QQ等国产浏览器左划返回上一页特性


## 事件冒泡

### 应用举例：点击'删除'时，禁止自动选中所在条目


## 点击相关的事件
* touchstart
* touchend
* touchcancel
* mousedown
* mouseup
* click

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