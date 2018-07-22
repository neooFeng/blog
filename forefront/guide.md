> Tip：阅读本文需要掌握html、css、javascript语法，并能独立完成带有网络交互功能的网页。   
  
### 目录
* 前端基础
* 前端防御式编程
* 响应式网站开发
* 浏览器原理与性能优化
* 搜索引擎与SEO
* 推荐学习

### 前端基础
* **HTML/CSS 布局**  
个人觉得CSS布局是检验一个程序员是否有合理编码习惯的好方式，很多老人写的CSS依旧是凌乱不堪毫无章法，很难想象他们能写好后端代码。推荐学习：
  1. [布局技巧](http://zh.learnlayout.com/)
  2. [flex布局](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
  3. [bootstrap](http://v3.bootcss.com/)
  4. [编码规范（@mdo）](http://codeguide.co/)、[编码规范（Google）](https://google.github.io/styleguide/htmlcssguide.html)  
     （要求掌握知识点：盒模型、各种position的意义、flex布局、float、fix等布局的优缺点）

* **JavaScript 精粹**  
JavaScript是一门非常棒的语言，语法简单，使用灵活，它是一位[大神](http://www.ruanyifeng.com/blog/2011/06/birth_of_javascript.html)仅用了10天发明出来的语言，所以难免也有一些不尽人意的地方。[JavaScript The Right Way](http://jstherightway.org/zh-cn/)应该是全世界最好的学习JS的地方了，里面有太多的干货，千万不要错过里面的任何一句话。  

* **CSS 编程思想**  
一个人写代码是不是真的厉害，我觉得看他使用不同语言的时候，尤其是使用一门新语言的时候，是否还能写出符合语言编程范式、充满设计感的代码就可以了。以前我以为CSS这种语言，应该不存在什么编程思想，直到我看到下面这个教程，才发现真是“编程思想无国界”~~    
[Scalable and Modular Architecture for CSS](https://smacss.com/book/)这篇文章展示了如何将抽象、继承等编程思想应用到CSS这门不算编程语言的语言中，令人拍案叫绝。

### 前端防御式编程
防御式编程是[Code Complete（程序员必读书籍）](https://book.douban.com/subject/1432042/)中提到过的一种编程思想，它假设所有外部输入都是不可信的，在我们编码时，要对所有可能的意外情况做处理，保证程序的健壮性。在实践过程中我一直遵循这个原则，确实减少了很多bug的发生。  
目前我在前端只见到以下几种场景，希望继续补充：  

    // 不推荐
    <div>{{model.user.name}}</div>
    // 推荐
    <div>{{model.user.name || 'default name'}}</div>  

    // 不推荐
    $. ajax({ 
        url: 'path/ url', 
        type: 'get', 
        success( data){ 
            // do something 
        } 
    });
    // 推荐
    $. ajax({ 
        url: 'path/ url', 
        type: 'get', 
        success( data ){ 
            // do something 
        },
        error( exception ){ 
            // do something 
        }  
    });

    // 不推荐
    this.body = yield render('path/template');
    // 推荐
    try {
        this.body = yield render('path/template');
    }catch(e){
        log.error(e);
        this.body = yield render('path/error');
    }

### 响应式网站开发
**目前比较主流的实现响应式网站开发的方法有两种：**
1. 通过前端或后端_判断userAgent来跳转不同的页面_完成不同设备浏览器的适配，也就是维护两个不同的站点来根据用户设备进行对应的跳转，常见网站b如[m.jd.com](https://m.jd.com/)，[www.jd.com](https://www.jd.com/)；
2. 使用media query等手段，让页面根据不同设备浏览器自动改变页面的布局和显示，但不做跳转，bootstrap就是Twitter开放的一个支持meida query的CSS框架，常见网站如[z.cn](https://www.amazon.cn)； 

**两种方式的优缺点：**  
* userAgent跳转
1. 需要开发并维护至少两个站点跳转来适配不同用户的设备浏览器；
2. 选择使用哪个站点内容由设备的userAgent信息来判断，无法根据屏幕尺寸或分辨率来决定；
3. 多了一次跳转。   
* media query自适应
1. 移动端浏览器加载了与桌面端浏览器相同的资源，例如图片、脚本资源等，导致移动端加载到冗余或体积较大的资源。对于网络和计算资源相对较少的移动端来说，这显然不是一个很好的处理方案。虽然可以经过一些优化来减少一部分移动端加载的内容，但是能做的非常有限，不能完全解决这个问题；
2. 桌面端浏览器和移动端浏览器访问站点需要展示的内容可能不完全相同，这种响应式的方式只实现了内容布局显示的适应，但是要做更多差异性的功能比较难。   

**总结：**  
* userAgent跳转的方式比较适用于功能复杂并对性能要求较高的站点应用；  
* media query的方式适用于访问量较小、性能要求不高的应用场景。  

### 浏览器原理与性能优化
讲到性能优化，躲不过要先了解一下[浏览器的工作原理](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/)，然后推荐以下快餐资源学习，更系统的知识可以参考文末> 推荐学习
1. [桌面浏览器前端优化策略](https://my.oschina.net/zhangstephen/blog/1601382)
2. [移动端浏览器前端优化策略](https://my.oschina.net/zhangstephen/blog/1601383)
3. [CSS Selector Performance](https://smacss.com/book/selectors)

### 搜索引擎与SEO
关于SEO的资料有很多，程序员应该更关注搜索引擎的原理，下面知识一篇基本资料  
1. [百度搜索引擎优化指南2.0](https://ziyuan.baidu.com/college/courseinfo?id=193&page=3)
2. [How Google Works (Youtube)](https://www.youtube.com/watch?v=BNHR6IQJGZs)
3. [数学之美](https://book.douban.com/subject/26163454/)

### 推荐学习
1. [阮一峰技术博客](http://www.ruanyifeng.com/blog/javascript/)  
    除了前端知识，这个博客里阮一峰还写了很多web开发过程中早晚会遇到的知识，比如SSH、HTTPS、数字签名、字符编码等，解释的比较通俗易懂，适合新手阅读。
2. [前端技术解析](https://book.douban.com/subject/27021790/)  
    作者14年毕业于华中科技大学，毕业后在腾讯工作，一直做前端方面的工作。  

    这本书对我这个主要做后端的人来说最大的帮助就是它完整的介绍了一遍前端技术发展历史，尤其是各种javascript框架的演变，从JS直接操作DOM，到MVC/MVP/MVVM/Virtual DOM等框架，作者没有介绍某一流行框架的使用方法，而是自己写伪代码来介绍各种类型框架的工作原理，这一点很棒。我们公司使用的是最原始的jquery操作DOM，所以每次听到React、AngularJS、Vue这些高级名词，都是不明觉厉，现在感觉好多了。  

    当然书中也介绍了一些移动端网页开发需要注意的事项，包括实现自适应的几种方式、页面加载效率等，对我的帮助都很大。
3. [高性能网站建设指南](https://book.douban.com/subject/3132277/)  
    这本书我只看了目录，因为感觉里面提到的东西基本在各种博客上已经看过就没有在读，但我还是要推荐一下，顺便推荐一篇文章[How Browsers Work](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)，了解了浏览器工作原理，就知道为什么这些tips可以提高网站响应速度了。
4. [HTTP权威指南](https://book.douban.com/subject/10746113/)  
    这本书可以说是程序员的必读书之一了，HTTP协议应该是最广泛使用的应用层协议。
