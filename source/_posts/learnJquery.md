---
title: learnJquery
date: 2017-06-04 15:58:36
tags:
---
# 1. jQuery整体架构
![img](http://www.ceerqingting.com/2017/06/04/learnJquery/jquery.jpg)
```javascript
    //函数自执行
    (function(global, factory) {
       factory(global);
          }(typeof window !== "undefined" ? window : this, function(window, noGlobal) {
              var jQuery = function( selector, context ) {
              return new jQuery.fn.init( selector, context );
            };
            jQuery.fn = jQuery.prototype = {};
            // 核心方法
            // 回调系统
            // 异步队列
            // 数据缓存
            // 队列操作
            // 选择器引
            // 属性操作
            // 节点遍历
            // 文档处理
            // 样式操作
            // 属性操作
            // 事件体系
            // AJAX交互
            // 动画引擎
            return jQuery;
     }));
```

## 1. 五大块
- 选择器
- DOM操作
- 事件
- AJAX
- 动画

## 2. 示例
- 接口：
```javascript
      .ajaxComplete()
      .ajaxError()
      .ajaxSend()
      .ajaxStart()
      .ajaxStop()
      .ajaxSuccess()
```
- 底层接口：
```javascript
    jQuery.ajax()
    jQuery.ajaxSetup()
```
- 快捷方法：<br>
```javascript
    jQuery.get()
    jQuery.getJSON()
    jQuery.getScript()
    jQuery.post()
```
## 3. jQuery接口设计原理
```javascript
    jQuery.each(["get", "post"]), function(i, method){
      jQuery[ method ] = function( url, data, callback, type) {
        //Shift arguments if data argument was omitted
        if( jQuery.isFunction( data )){
          type = type || callback;
          callback = dta;
          data = undefined;
        }
        return jQuery.ajax({
          url: url,
          type: method,
          data: data,
          success: callback
        })
      }
    }
```

# 2. 立即调用表达式
> 解决命名空间与变量污染的问题  
参考：[立即调用的函数表达式](http://www.cnblogs.com/TomXu/archive/2011/12/31/2289423.html)  

## 1. 写法
```javascript
//写法一：
  (function(window, factory){
    factory(window)
  }(this, function(){
    return function(){
      //jQuery的调用
    }
  }))
//写法二：
  var factory = function(){
    return function(){
      //执行方法
    }
  }
  var jQuery = factory();
//写法三：
  (function(window, undefined){
    var jQuery = function(){}
    //....
    window.jQuery = window.$ = jQuery;
  })(window);
```
## 2. 写法优势
  - window和undefined都是为了减少变量查找所经过的scope作用域。当window通过传递给闭包内部之后，在闭包内部使用它的时候，可以把它当成一个局部变量，显然比原先在window scope下查找的时候要快一些。

  - undefined也是同样道理，其实这个undefined并不是JavaScript数据类型的undefined，而是一个普普通通的变量名。只是因为没给它传递值，他的值就是undefined, undefined并不是JavaScript的保留字。  
   
## 3. 为什么要传递undefined  
```javascript
//Applications Javscript中的undefined并不是作为关键字 因此可以允许用户对其赋值  
 var undefined = "a";
 (function(window){
   alert(undefined); //IE8 "a"
 })(window)
```
## 4. 为什么要创建一个外层包裹
```javascript
  1、匿名函数与自执行
    function(){
      //代码逻辑
    }
   //上面这种写法是错了，声明了它但是又不给名字又没有使用，所以在语法上错误的，那么怎么去执行一个匿名的函数呢？
   //要调用一个函数，我们必须要有方法定位它、引用它。所以，我们要取一个名字：
   var jQuery = function(){
     //代码逻辑
   }
   //jQuery使用()将匿名函数括起来，然后后面再加一对小括号（包含参数列表），那么这小括号能把我们的表达式组合分块，并且每一块（也就是每一对小括号），都有一个返回值。这个返回值实际上也就是小括号中表达式的返回值。所以，当我们用一对小括号把匿名函数括起来的时候，实际上小括号返回的，就是一个匿名函数的Function对象。因此，小括号对加上匿名函数就如同有名字的函数般被我们取得它的引用位置了。所以如果在这个引用变量后面再加上参数列表，就会实现普通函数的调用形式。

2、jQuery在不同平台的下的加载逻辑
 // AMD 和 CommonJS 的支持代码
  if(typeof module === "object" && typeof module.export === "object"){
    module.exports = global.document ? factory(global, true): function(w){
      if(!w.document){
        throw new Error("jQuery requires a window with a document");
      }
      return factory(w);
    }
  }else{
    factory(global);
  }
```
## 5. 总结
> 全局变量是魔鬼, 匿名函数可以有效的保证在页面上写入JavaScript，而不会造成全局变量的污染，通过小括号，让其加载的时候立即初始化，这样就形成了一个单例模式的效果从而只会执行一次。

# 3. jQuery的类数组对象结构
## 1. 为什么是类数组对象呢？  
```javascript
//9种方法重载
  1.jQuery([selector, [context]])
  2.jQuery(element)
  3.jQuery(elementArray)
  4.jQUery(object)
  5.jQuery(jQuery object)
  6.jQuery(html, [ownerDocument])
  7.jQuery(html, [attributes])
  8.jQuery()
  9.jQuery(callback)
```
>9种方法整体可以分三大块： 选择器、dom的处理、dom加载
换句话说jQuery就是为了获取DOM、操作DOM而存在的，所以为了更方便这些操作，让节点与实例对象通过一个桥梁给关联起来，jQuery内部就采用了一种叫“类数组对象”的方法作为存储接口，所以我们既可以像对象一样处理jQuery操作，也能像数组一样可以使用push、pop、shift、unshift、sort、each、map等类数组的方法操作jQuery对象了  

## 2. jQuery对象可用数组下标索引是什么原理?  
通过$(".Class")构建的对象结构如下：  
![img](/images/selector.jpg)  
```javascript
  var aQuery = function(selector){
      if(!(this instanceof aQuery)){
        return new aQuery(selector);
      }
      var elem = document.getElementById(/[^#].*/.exec(selector)[0]);
      this.length = 1;
      this[0] = elem;
      this.context = document;
      this.selector = selector;
      this.get = function(num){
        return this[num];
      }
      return this;
    }
```
>通过对象键值对的关系保存属性， 原型保存方法, 以上为模拟jQuery的对象结构  

## 3. jQuery的无new构建原理
>函数aQuery()内部首先保证了必须是通过new操作符构建。这样就能保证当前构建的是一个带有this的实例对象，既然是对象我们可以把所有的属性与方法作为对象的key与value的方式给映射到this上，所以如上结构就可以模拟出jQuery这样的操作了，即可通过索引取值，页也可以链式方法取值，但是缺陷是每次调用ajQuery方法等于是创建了一个新的实例，那么类似get方法就要在每一个实例上重新创建一遍，性能大打折扣，所以jQuery在结构上的优化不仅仅只是我们看到的，除了实现类数组结构、方法的原型共享，而且还实现方法的静态与实例的共存

# 4. jQuery中的ready与load事件
## 1. jQuery3种针对文档加载的方法
```javascript
$(document).ready(function(){
  //...代码...
})
//document ready简写
$(function(){
   //...代码...
})
$(document).load(function(){
   //...代码...
})
```
## 2. ready和load区别
>DOM文档加载步骤
1. 解析HTML结构
2. 加载外部脚本和样式表文件
3. 解析并执行脚本代码
4. 构造HTML DOM模型。//ready
5. 加载图片等外部文件
6. 页面加载完毕 //load
```javascript
  //jQuery处理文档加载时机
  jQuery.ready.promise = function( obj ){
    if(!readyList){
      readyList = jQuery.Deferred();
      if(document.readyState === 'complete'){
        //Handle it asynchronously to allow scripts the opportunity to delay ready
        setTimeot(jQuery.ready);
      }else{
        document.addEventListener("DOMContentLoaded",completed, false);
        window.addEventListner("load", completed, false)
      }
    }
    return readyList.promise(obj);
  }
```
# 5. jQuery多库共存处理
## 1. 使用DEMO
```javascript
  jQuery.noConflict();
  //使用jQuery
  jQuery("aaron").show();
  //使用其他库的$()
  $("aa").style.display = "block";
  //这个函数必须在导入jQuery文件之后，并且在导入另一个导致冲突的库之前使用
```
## 2. 原理
```javascript
  var _jQuery = window.jQuery, _$ = window.$;
  jQuery.noConflict = function(deep){
    if(window.$ === jQuery){
      window.$ = _$;
    }
    if(deep && window.jQuery === jQuery){
      window.jQuery = _jQuery;
    }
    return jQuery;
  }
```
# 6. 对象的构建
```javascript
//类一
  function ajQuery(){
    this.name = 'jQuery';
    this.sayName = function(){
      return this.name;
    }
  }
  var a = new ajQuery();
  var b = new ajQuery();
  var c = new ajQuery();

//类二
 function ajQuery(){
   this.name = 'jQuery';
 }  
 ajQuery.prototype = {
   sayName: function(){
     return this.name
   }
 }
 var a  = new ajQuery();
 var b  = new ajQuery();
 var c  = new ajQuery();
```
类一与类二产生的结构几乎是一样的，本质区别是：  
类二new产生的a、b、c三个实例对象共享了原型的sayName方法，节省了内存空间，类一则是要为每一个实例复制sayName方法，每个方法属性都占用一定空间，类二则是要通过scope连接到原型链查找，无形之中要多一层作用域的查找
```javascript
  //jQUery对象的构建
  jQuery = function(selector, context){
    return new jQuery.fn.init(selector, context);
  }
  jQuery.fn = jQuery.prototype = {
    init: function(){
      return this
    },
    jquery: version,
    constructor: jQuery,
    ....
  }
  var a = $();
```
ajQuery与jQuery结构不同点：
1. 没有采用new操作符
2. return返回的是一个通过new出来的对象

# 7. 分离构造器
## 1. 通过new操作符构建一个对象：
   - 创建一个新对象
   - 将构造函数的作用域赋给新对象（所以this就指向了这个新对象）
   - 执行构造函数中的代码
   - 返回这个新对象  
```javascript
  //常见类式写法
  var $$ = ajQuery = function(selector){
    this.selector = selector;
    return this;
  }
  ajQuery.fn = ajQuery.prototype = {
    selectorName: function(){
      return this.selector;
    },
    construction: ajQuery
  }
  var a = new $$('aaa');
  a.selectorName();//aaa
```
改造jQuery无new格式
```javascript
  var $$ = ajQuery = function(selector){
    if(!(this instanceof ajQuery)){
      return new ajQuery(selector);
    }
    this.selector = selector;
    return this
  }
```
错误写法：
```javascript
   var $$ = ajQuery = function(selecotr){
     this.selector = selector;
     return new ajQuery(selector);
   }
   //Uncaught RangeError: Maximum call stact size exceeded
```
jQuery为了避免出现这种死循环，采取的方法为把原型上的一个init方法作为构造器
```javascript
  var $$ = ajQuery = function(selector){
    //把原型上的init作为构造器
    return new ajQuery.fn.init(selector);
  }
  ajQuery.fn = ajQuery.prototype = {
    name: 'aaron',
    init: function(){
      console.log(this);
    },
    constructor: ajQuery
  }
```
问题：init是ajQuery原型上作为构造器的一个方法，那么this就不是ajQuery了，所以this就完全引用不到ajQuery的原型了，这里通过new把init方法与ajQuery给分离成2个独立的构造器

# 8. 静态与实例方法共享设计
## 1. 遍历方法
```javascript
  $('.aaron').each() //作为实例方法存在
  $.each() //作为静态方法存在
```
第一条语句是给有指定的上下文调用的，就是$('.aaron')获取的DOM合集，第二条语句$.each()函数可用于迭代任何集合，无论是“名/值”对象或者数组。在迭代数组的情况下，回调函数每次都会传递一个数组索引和相应的数组值作为参数，那是不是要写两个方法
```javascript
  //jQuery源码
  jQuery.prototype = {
    each: function(callback, args){
      return jQuery.each(this, callback, args);
    }
  }
```
## 2. 共享方法
> jQuery通过new原型prototype上的init方法当做构造器，那么init的原型链方法就是实例的方法，所以jQuery通过2个构造器划分2种不同的调用方式，一种是静态，一种是原型，方法是共享的
```javascript
//jQuery方案
  ajQuery.fn = ajQuery.prototype = {
    name: 'aaron',
    init: function(selector){
      this.selector = selector;
      return this;
    },
    constructor: ajQuery
  }
  ajQuery.fn.init.prototype = ajQuery.fn;
```
![img](/images/init.jpg)  
通过原型传递解决问题，把jQuery的原型传递给jQuery.prototype.init.prototype,因为是引用传递所以不需要担心循环引用性能的问题
# 9. 方法链式调用的实现 
## 1. DSL
> Domain Specific Language: 用于描述和解决特定领域问题的语言
```javascript
 $('input[type="button"]').eq(0).click(function(){
    alert('点击我！')
}).end().eq(1).click(function(){
    $('input[type="button"]:eq(0)').trigger('click');
}).end().eq(2).toggle(function(){
    console.log('toggle')
})
```
这种管道风格的DSL链式代码：
- 节约JS代码
- 所返回的都是同一个对象，提高代码效率
## 2. return this
```javascript
   var $$ = ajQuery = function(selector){
      return new ajQuery.fn.init(selector);
    }
    ajQuery.fn = ajQuery.prototype = {
      name: 'aaron',
      init: function(selector){
        this.selector = selector;
        return this;
      },
      constructor: ajQuery
    }
    ajQuery.fn.init.prototype = ajQuery.fn;
    ajQuery.fn.setName = function(myName){
      this.myName = myName;
      return this;
    }
    ajQuery.fn.getName = function(){
      $('#aaron').html(this.myName);
      return this;
    }
    $$().setName('jiangfen').getName();
```
如果需要链式处理，只需要在方法内部返回当前这个实例对象的this就可以了，因为返回当前实例的this,从而可以访问自己的原型。这样处理只是同步链式，除了同步链式还有异步链式。

# 10. 插件接口的设计
## 1. jQuery插件开发分为两种
- 挂在jQuery命名空间下的全局函数，也可称为静态方法
- 挂在jQuery原型下的方法，这样通过选择器的jQuery对象实例也能共享该方法。

```javascript
  //提供的接口
  $.extend(target, [object1], [objectN])
  //接口的使用： 
  jQuery.extend({
    data: function(){},
    removeData: function(){}
  })

  jQuery.fn.extend({
    data: function(){},
    removeData: function(){}
  })
```
## 2. jQuery实现方法  
- jQuery支持自己扩展属性，jQuery.extend和jQuery.fn.extend其实是同指向同一方法的不同引用
- jQuery.extend调用的时候上下文指向的是jQuery构造器
- jQuery.fn.extend调用的时候上下文指向的是jQuery构造器的实例对象
```javascript
   var  $$ = ajQuery = function(selector){
      return new ajQuery.fn.init(selector);
    }
    ajQuery.fn = ajQuery.prototype = {
      name: "aaron",
      init: function(selector){
        this.selector = selector;
        return this;
        },
      constructor: ajQuery
    }
     ajQuery.fn.init.prototype = ajQuery.fn;
     ajQuery.extend = ajQuery.fn.extend = function(){
       var options, src, copy, target = arguments[0]||{},i = 1, length = arguments.length;
       //只有一个参数，就是对jQuery自身的扩展处理
       //extend,fn.extend
       if(i === length){
         target = this; //调用的上下文对象jQuery/或者实例
         i--;
       }
       for(;i < length; i++){
         //从i开始取参数，不为空开始遍历
         if((options = arguments[i])!=null){
           for(name in options){
             copy = options[name];
             //覆盖拷贝
             target[name] = copy;
           }
         }
       }
       return target;
     }
     ajQuery.fn.extend({
       setName: function(myName){
         this.myName = myName;
         return this;
       },
       getName: function(){
         document.querySelector("#aaron").innerHTML = this.myName;
         return this;
       }
     })
   $$().setName("张辉是个大傻逼").getName();
```
# 11. 回溯处理的设计
## 1. 关于jQuery对象的包装
```javascript
  var $test = $('test');
```
返回jQuery对象  
![img](/images/prevObject.jpg)  
> jQuery内部维护着一个jQuery对象栈， 每个遍历方法都会找到一组新元素（一个jQuery对象），然后jQuery会把这组元素推入到栈中。  
> 而每个jQuery对象都有三个属性： context、selector和prevObject, 其中的prevObject属性就指向这个对象栈中的前一个对象，通过这个属性可以回溯到最初的DOM元素集中。  

```javascript
  <ul id="test">
    parent
    <li>child</li>
  </ul>
 //分开
 var aa = $('#test);
 test.find('li').click(function(){
   alert(1); //1
 })
 test.click(function(){
   alert(2)
 })
//链式
test.find('li').click(function(){
  alert(1);
}).end().click(function(){
  alert(2);
}) 
```
jQuery为我们操作内部对象栈提供了非常有用的2个方法
- .end()
  回到前一个jQuery对象
- .addBack()
  调用它会在栈中回溯一个位置，然后把两个位置上的元素组合起来，并把这个新的组合之后的元素集推入栈的上方  

利用这个DOM元素栈可以减少重复的查询和遍历的操作


# 11. end与addBack
## 源码实现
```javascript
  end: function(){
    return this.prevObject || this.constructor(null);
  }

  jQuery.fn.extend({
    find: function(selector){
          jQuery.find(selector, self[i], ret);
          //Needed because $(selector,context)becomes $(context).find(selector)
          ret = this.pushStack(len > 1? jQuery.unique(ret):ret);
          ret.selector = this.selector?this.selector + " " + selector: selector;
          return ret;
    }
  })
```
>流程解析   
1. 首先构建一个新的jQuery对象，因为constructor是指向构造器的，所以这里就等同于调用jQuery()方法了，返回了一个新的jQuery对象   
2. 然后用jQuery.merge语句把elems节点合并到新的jQuery对象   
3. 最后给返回的新jQuery对象添加prevObject属性


# 12. 仿栈与队列的操作
## 1. get方法
```javascript
  get: function(num){
    return num != null?//Return just the one element from the set
    (num < 0 ? this[num + this.length])://Return all the elements in a clean array
    slice.call(this)
  }

```