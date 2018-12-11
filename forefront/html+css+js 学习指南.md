 
  ### 目录
* 开源知识
  * HTML/CSS 布局
  * JavaScript 学习
* 个人JS学习札记
  * [技巧] 防御式编程
  * [趣事] 浮点数异常
  * [趣事] 自执行代码
  * [最佳实践] 公用方法封装
  * [趣事] 异步规则
  * [趣事] 对象的深浅拷贝
  * [趣事] JS如何被解析执行  
  
  
  
### 开源知识
* **HTML/CSS 布局**  
    > 要求掌握知识点：盒模型、各种position的意义、flex布局、float、fix等布局的优缺点   

    [布局技巧](http://zh.learnlayout.com/)   
    简洁明了，适合初学CSS布局方法   
       
    [flex布局](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)   
    新一代布局利器，但需要注意浏览器兼容性  
      
    [bootstrap](http://v3.bootcss.com/)   
    Twitter开源的CSS工具集，可以从中学习下css class的命名组合  
      
    [编码规范（@mdo）](http://codeguide.co/)、[编码规范（Google）](https://google.github.io/styleguide/htmlcssguide.html)    
  
  
* **JavaScript 学习**  
    > JavaScript是前端最重要的语言，必须学的66的

    [JavaScript The Right Way](http://jstherightway.org/zh-cn/)  
        这个网站搜集了很多JS学习资源，涵盖了JS的方方面面，每个方面搜集的都是非常有质量的文章，值得仔细学习，一个链接也不要错过。   
          
    [JavaScript-Garden](http://bonsaiden.github.io/JavaScript-Garden/zh/)  
        这篇文章的作者是两位 Stack Overflow 用户合作而成，非常精炼的讲解了使用JS的一些最佳实践，以及最佳实践背后的JS原理。  
          


### 个人总结
**[技巧] 防御式编程**  
防御式编程是[Code Complete（程序员必读书籍）](https://book.douban.com/subject/1432042/)中提到过的一种编程思想，它假设所有外部输入都是不可信的，在我们编码时，要对所有可能的意外情况做处理，保证程序的健壮性。  
在实践过程中遵循这个原则，确实能减少很多bug的发生。目前我在前端只见到以下几种场景，希望继续补充：  

    // 不推荐
    <div>{{user.name}}</div>
    // 推荐
    <div>{{user.name || 'default name'}}</div>  

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

**[技巧] 防御式编程**  
**[趣事] 浮点数异常**  
**[趣事] 自执行代码**  
**[最佳实践] 公用方法封装**  
**[趣事] 异步规则**  
**[趣事] 对象的深浅拷贝**  
**[趣事] JS如何被解析执行**  