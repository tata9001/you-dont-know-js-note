# 你不知道的JS(笔记)

## 动机 :
>   在看"你不知道的JS"系列文章的时候,GET到了不少JS不为人知的小秘密.在之前,使用JS只是流于表面,会使用
    JS去实现一些简单的功能(因为是自身是Java栈,用起来还是顺手),可当要去实现一些复杂的功能或者要求修改
    一些第三方JS库的bug时候,就彻底蒙圈了.

>   程序猿界的一个现象是:一年和三年Java的程序猿差别并不会很大,因为Java语言自身限制了很多东西,实现一些
    功能的代码差距并不大;而一个三年的JS程序猿和一年的差距可就大了去了,因为JS限制很少,不同境界的程序猿
    实现一些功能的方式会使千差万别的.

>   借着新工作需要大量使用JS的契机,自己将通过仔细阅读"你不知道的JS"这一系列的文章,深入的探索一下JS的世界.

## 第一章:类型(Types)
### 内建类型(built-in):
大多数的程序猿会认为动态语言(例如:JS)没有类型,但Js确确实实是有类型的.JS中有null,undefined,boolean
    string,number,object,symbol(ES6)共7种内建类型,其中除了object以外,
    都称为原始类型(primitives).

    typeof undefined     === "undefined"; // true
    typeof true          === "boolean";   // true
    typeof 42            === "number";    // true
    typeof "42"          === "string";    // true
    typeof { life: 42 }  === "object";    // true
    typeof null === "object"; // true
    typeof Symbol()      === "symbol";    // true
    
其中:'typeof null === "object"; // true'是一个存在了两个世纪的bug,即使ES6也没有敢修复.如果需要去判断
    一个值是不是null,需要组合判断:
   
    var a = null;
    (!a && typeof a === "object"); // true
    
当然在提到JS的时候,必然不能忘了它的一等公民:function
    
    typeof function a(){} === "function"; // true
其实function是内建类型,但它是object的子类型(subtype):callable object,内部有一个[[Call]]的属性在typeof的适合调用.    
那是不是数组(Array)应该和function一样,有自己的子类型?答案是NO

    typeof [1,2,3] === "object"; // true
    
### 值类型(Values as Types)
JS中,变量(variables)没有类型,但值(values)有类型.变量(variables)可以在任何时间持有任何类型.
当你使用typeof去获取一个变量的类型时,它的意思不是说:"你这个变量的类型是啥?",而是说:"你这个变量的值类型是啥?".
    
    var a = 42;
    typeof a; // "number"
    
    a = true;
    typeof a; // "boolean"
### undefined vs "undeclared"

当一个变量没有值的时候,它就是undefined
    
    var a;
    typeof a; // "undefined"
    typeof b; // "undefined"
大多数JS程序猿都会很容易认为undefined就是undeclared的同义词,然而在js中真不是.
    
* undefined是已经在当前可访问的作用域中声明,但没有赋予真正的值;
* undeclared是在在当前可访问的作用域中没有声明.


    var a;
    a; // undefined
    b; // ReferenceError: b is not defined
值得注意的是:typeof作用在"undeclared"的变量上面时,不会抛出异常,这就确保了typeof是一个安全的操作.      
存在这种场景,我们在平时开发中需要通过一个变量来标示或者控制一些调试代码的运行.一般会在最上层的debug.js
    中声明'var DEBUG = true;',在开发环境下引入这个js文件,生产环境下则不.那么我们在产品代码中的判断会
    如下:
    
    // oops, this would throw an error!
    if (DEBUG) {
        console.log( "Debugging is starting" );
    }
    
安全的检查如下:

    // this is a safe existence check
    if (typeof DEBUG !== "undefined") {
        console.log( "Debugging is starting" );
    }
当然也可以使用
    
    if (window.DEBUG) {
        // ..
    }
但这样的话,代码就不能适应多运行环境.