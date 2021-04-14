`JavaScript`=`ECMAScript`+`JavaScript`特有的东西(`BOM`+`DOM`)

#### ECMAScript：客户端脚本语言的标准

#####  1.JS与HTML结合方式

* 内部JS

用`<script>`标签直接引入，在标签内部编写JS代码。并且可以放在任何位置。同时这个位置对执行顺序有影响。

* 外部JS

使用`<script src="">`标签引入外部JS

* 注意

  `<script>`标签可以定义在HTML的任意位置，但是这个位置对执行顺序有影响。

##### 2.注解

* 单行注释：//注释内容
* 多行注释：/* 注释内容 */

##### 3.数据类型

* 原始数据类型

  * __number :__

    数字，数字/小数/NAN（not a number 一个不是数字的数字类型）

  * __string__

    字符串，` “abc”` 或者` ‘abc’`

  * __boolean__

    `true` 或者` false`

  * __null__

    一个对象为空的占位符,

    * 注意

      使用typeof运算符，该数据类型为 object。

  * __undefined__

    未定义，如果一个变量没有给初始值，默认赋值`undefined`。

* 引用数据类型
  * 对象

##### 4.变量

* 变量

  一块存储数据的内存空间

* Java强类型，JavaScript是弱类型

  * 强类型：在开辟存储空间时，定义了空间的存储数据类型，只能存储固定类型的数据
  * 弱类型：开辟的存储空间，可以存放任意类型的数据

* 语法

   `var 变量名=变量；`

##### 5.运算符

* 一元运算符：只有一个运算数的运算符
  * ++
  * --
  * +（正号）
* 算数运算符
  * +
  * -
  * *
  * /
  * %
* 赋值运算符
  * =
  * +=
  * -+
* 比较运算符
  * \>
  * 
* 逻辑运算符

* 特色语法 

  使用var的变量，定义的是局部变量

  不适用是全局变量

##### 6.流程控制语句

* if else
* switch
  * 在JS中，可以接受任何类型变量
* while
* do...while 
* for

#### JavaScript对象

##### 1.Function

```javascript
var 方法名=Function(形式参数列表){
    方法体
}
```

```javascript
//返回该方法的参数个数
方法名.length  
```

* 在方法声明中有一个隐藏的内置对象（数组），arguments，封装所有的实际参数。

```javascript
var  fun1=function (){
        var sum=0;
        for (var i=0;i<arguments.length;i++){
            sum+=arguments[i];
        }
        return sum;
    }
    let fun = fun1(1,6,55,8);
    alert(fun)
```

##### 编码

 ## BOM



## DOM

> 控制HTML文档内容
>
> 主要用来获取页面标签。

#### 1.事件







