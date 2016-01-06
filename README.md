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
大多数的程序猿会认为动态语言(例如:JS)没有类型,但Js确确实实是有类型的.JS中有null,undefined,boolean,string,number,object,symbol(ES6)共7种内建类型,其中除了object以外,都称为原始类型(primitives).

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
其实function是内建类型,但它是object的子类型(subtype):callable object,内部有一个[[Call]]的属性在typeof操作时调用.    
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
    
 >   * undefined是已经在当前可访问的作用域中声明,但没有赋予真正的值;
 
 >   * undeclared是在在当前可访问的作用域中没有声明.
   
    var a;
    a; // undefined
    b; // ReferenceError: b is not defined


值得注意的是:typeof作用在"undeclared"的变量上面时,不会抛出异常,这就确保了typeof是一个安全的操作.存在这种场景,我们在平时开发中需要通过一个变量来标示或者控制一些调试代码的运行.一般会在最上层的debug.js中声明'var DEBUG = true;',在开发环境下引入这个js文件,生产环境下则不.那么我们在产品代码中的判断会如下:
    
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
##第二章：值（Values）
这一章主要关注在JS的内建值类型上：arrays,strings,numbers以及一些特殊值（null,undefined,NAN,-0,Infinity）.
### 1.数组（Array）
>JS不同于其他的类型强制语言，同一个数组的元素可以使任意类型的，从string到number，以及任何object。

    var arr = ['a',10,new Date()];
>注意：不要使用delete操作数组,此操作不会更新length属性。    
>清空数组的几种方式：
    
    //1.控制length属性
    arr.length = 0;
    console.log(arr);// []
    //2.直接赋值
    arr = [];
>尽量减少使用/创建稀疏（sparse）数组
    
    var arr = [];
    arr[4] = 9;
    console.log(arr[0]);//undefined
    console.log(arr.length);//5
>请不要给数组添加非数字类属性,避免影响数组的固有行为，请让数组做一个安静的美男子

    arr['13'] = 13;
    console.log(arr[13]);//13
    console.log(arr.length);//14
### 2.类数组（Array-Likes）：与数组一样（数字键值索引的值集合），但没有数组的一些方法
>JS中常见的类数组有function中的arguments对象以及DOM选择器的返回值。可以通过以下方式实现类数组使用数组的方法。
    
    //1.ES5将类数组转换为数组
    function foo() {
    var arr = Array.prototype.slice.call( arguments );
    arr.push( "z" );
    console.log( arr );
    }
    foo('w','y');//['w','y','z']
    //2.ES6中将类数组转换为数组
     Array.from(document.getElementsByTagName('div'));//div元素数组
     Array.from('aa');//['a','a']
### 3.字符串（Strings）
>我们通常会认为字符串的底层实现是字符数组，但在JS中不是，他们之间的相似只是在表面。

    var a = "foo";
    var b = ["f","o","o"];
    console.log('a.length:',a.length);// 3
    console.log('b.length:',b.length);// 3
    console.log('a.indexOf( "o" ):', a.indexOf( "o" ));// 1
    console.log('b.indexOf( "o" ):', b.indexOf( "o" ));// 1
    console.log('a.concat( "bar" ):', a.concat( "bar" ));// foobar
    console.log('b.concat["b","a","r"]  ):', b.concat( ["b","a","r"] ));// ["f","o","o","b","a","r"] 
    console.log('a[0]:', a[0]);//f
    console.log('b[0]:', b[0]);//f
>字符串是不可变的（immutable）：

    a[0]='d';
    b[0]='d';
    console.log('a[0]:', a[0]);//f
    console.log('b[0]:', b[0]);//d
>字符串可以借用一些自己没有而数组有的方法：

    console.log(Array.prototype.join.call( a, "-" ));//f-o-o
    var mapVar = Array.prototype.map.call( a, function(v){
     return v.toUpperCase() ;
    } ).join( "." );
    console.log('mapVar:',mapVar);
>但很不幸的是有些方法是不能进行借用的,例如数组的reverse,因为字符串是不可变的.
    
    var c = a.split('').reverse().join('');
    console.log(c);
>可以通过以上方式来实现字符串的reverse功能,但这中写法写只能在不能用于多字节的Unicode(𝌆).若要使用可以参考https://github.com/mathiasbynens/esrever。

>总而言之，当使用字符串的时候就不要期望它有字符数组的特性，若要使用，请自己将其转换为字符数组。


###3.数值（Numbers）
>在JS中，number就是所有的数值类型，是基于IEEE 754标准的双精度浮点（64位）格式。其中‘整数’就是没有小数部分数值。    
数值中0是可以省略的，例如：
    
    42.0 === 42. ;//true
    .42 === 0.42 ;//true
>虽然看着很怪异，但确实是合法的，不鼓励使用。   
数值可以使用科学计数法：
    
    var num = 5E3;
    console.log(num);
    console.log(num.toExponential());
>数值会自动装箱成Number对象，调用相关的方法；    
可以使用二进制，八进制，十六进制来标示数值：

    console.log(0xf3);//243
    console.log(0o363);//243
    consile.log(0b11110011);//243
>在浮点数的世界里，有一个痛点就是不能直接判断两个小数是否相等：
    
    0.1 + 0.2 === 0.3; // false
    console.log(0.1+0.2);//0.30000000000000004
>这种问题只存在小数之间，整数是绝对安全的。    
那如何解决这种问题？那就需要使用machine epsilon = 2^-52，在ES6中已经定义了数值常量Number.EPSILON
    
    function numbersCloseEnoughToEqual(n1,n2) {
        return Math.abs( n1 - n2 ) < Number.EPSILON;
    }
    numbersCloseEnoughToEqual(0.1 + 0.2,0.3); // true
    numbersCloseEnoughToEqual(0.0000002,0.00000021);//false
>安全的整数范围：Number.MAX_SAFE_INTEGER（Math.pow(2, 53)-1）,为何成为最大安全整数？
    
    Math.pow(2, 53) === Math.pow(2, 53) + 1 // true
>在IEEE 754中只能安全的表示[ -2^53+1 , 2^53-1 ] 这个范围。超过这个数值范围的运算不能使用数值操作符了。    
整数最大值：Number.MAX_VALUE （1.798e+308）Number.MIN_VALUE（ 5e-324）

    