# 开篇
不是八阿哥的八阿哥，看似正确然而却让人打脑壳。
## 举例
例1
```javascript
(function() {
    return NaN === NaN;
})();
// false
```
例2
```javascript
(function() {
    return (0.1 + 0.2 === 0.3);
})();
// false
```
例3
```javascript
[5, 12, 9, 2, 18, 1, 25].sort();
// [1, 12, 18, 2, 25, 5, 9]
```
例4
```javascript
var a = "1"
var b = 2
var c = a + b
// c = "12" 

var a = "1"
var b = 2
var c = +a + b
// c = 3
```
例5
```javascript
(function() {
    return ['10','10','10','10'].map(parseInt);
})();
// [10, NaN, 2, 3]
```
例6
```javascript
(function() {
    return 9999999999999999; // 16个9
})();
// 10000000000000000
```
例7
```javascript
var a = [];
for (var i = 0; i < 10; i++) {
    a[i] = function () {
        console.log(i);
    };
}
a[1](); 
a[2]();
a[3]();
// 10,10,10
```
下面对这些错误做个说明记录
## NaN
**NaN**，不是一个数字，是一种特殊的值来代表不可表示的值。因为有很多方法来表示一个非数字，所以一个非数字不会等于另一个为NaN的非数字。
> 这是对你的提醒，NaN的意思是“不为NaN".
  — Ariya Hidayat

然而，就算是本身的全局函数也仍然有缺陷
```javascript
console.log(isNaN('hello'));  // true
console.log(isNaN(['x']));    // true
console.log(isNaN({}));       // true
```
不过好在新的**ES6**中有一个方法可以用来检测NaN值--**Number.isNaN()**。这时候，就只有真正的**NaN**值才能返回`true`了
```javascript
console.log(Number.isNaN(NaN));            // true
console.log(Number.isNaN(Math.sqrt(-2)));  // true
console.log(Number.isNaN('hello'));        // false
console.log(Number.isNaN(['x']));          // false
console.log(Number.isNaN({}));             // false
```
## 数值精度
需要说明的是，因为计算机是二进制来表示浮点数的，所以无法准确表示一个浮点数，只能逼近。
在上述例子中`0.1+0.2=0.30000000000000004`，所以如果变成
```javascript
(function() {
    return (0.1 + 0.2 === 0.30000000000000004);
})();
// true
```
[精确浮点数计算](http://acm.whu.edu.cn/starter/problem/detail?problem_id=1069)，在这里可以看到很多浮点数的计算结果与理想值的都存在一点点细微的差距。想要从根本上排除这种小差距可以使用逼近的比较
```javascript
if(fabs(a-b) < 1E-10) // do something
```
不过当这个数字逐渐变大的时候上述例子中又会返回一个`true`，不知道是不是数字变大之后能够自动四舍五入额
```javascript
(function() {
    return (0.2 + 0.2 === 0.4);
})();
// true
这处理的，甚是诡异啊……
## sort()
这其实是**JavaScript Array**对象的方法，如果在使用该方法时没有使用参数，将按字母顺序对数组中的元素进行排序，说得更精确点，是按照字符编码的顺序进行排序。
说白了在对数字进行排序的时候无法按照这一方式来排序所以无法得到正确的序列。这就需要我们自己选择排序方法来实现数字数组的排序，可以是正序也可以是倒序的
```javascript
function positive (a, b) {
    return a > b
}

function reverse (a, b) {
    return a < b
}
[5, 12, 9, 2, 18, 1, 25].sort(positive);
// [1, 2, 5, 9, 12, 18, 25]
[5, 12, 9, 2, 18, 1, 25].sort(reverse);
// [25, 18, 12, 9, 5, 2, 1]
```
较函数具有两个参数**a**和**b**，其返回值如下：
* 若 a 小于 b，在排序后的数组中 a 应该出现在 b 之前，则返回一个小于 0 的值。
* 若 a 等于 b，则返回 0。
* 若 a 大于 b，则返回一个大于 0 的值。
## 变量类型
```javascript
var a = "1"
var b = 2
var c = a + b
// c = "12" 
typeof(c)
// string
```
虽然写着`var`，但是每一个变量实际还是有一个类型的，使用`typeof`不难看出当一个`string`类型的变量**a**和一个`number`**b**类型的变量相加时得到了一个`string`类型的变量**c**，之就相当于完成了一次字符串的拼接操作。
不知道是不是`string`类型的等级要高一些，导致了类型转换的发生，就像其他语言中`int`和`float`相加时总会造成类型转换。
```javascript
var a = "1"
var b = 2
var c = +a + b
// c = 3
typeof(+a)
// number
```
这一个其实是**JavasCript**中一个符号的特殊效果，就像是`{}`、`[]`、`.`这些符号在**JavaScript**中都可以直接使用一样，这里的`+`其实可以看成是一个强制的类型转换，好比是`parseInt()`方法一样，将字符串中的数字**1**提取了出来，从`typeof(+a)`也可以看到此时已经变成两个`number`类型的变量在做了一个加法操作了。
## map与parseInt
先说说`map()`
> `map()` 方法返回一个由原数组中的每个元素调用一个指定方法后的返回值组成的新数组。
  语法：array.map(callback[, thisArg])，其中`callback`含有三个参数：
  1.currentValue：`callback`的第一个参数，数组中当前被传递的元素；
  2.index：`callback`的第二个参数，数组中当前被传递的元素的索引；
  3.array：`callback`的第三个参数，调用`map()`方法的数组。
  thisArg：执行 callback 函数时 this 指向的对象。

接着说说`parseInt()`
> `parseInt()`函数将给定的字符串以指定基数（radix/base）解析成为整数。
  语法：parseInt(string, radix)，其中的两个参数：
  1.string：要被解析的值。如果参数不是一个字符串，则将其转换为字符串。字符串开头的空白符将会被忽略；
  2.radix：一个2到36之间的整数值，用于指定转换中采用的基数。比如参数"10"表示使用我们通常使用的十进制数值系统。
  PS：总是指定**radix**参数可以消除阅读该代码时的困惑并且保证转换结果可预测。当忽略该参数时，不同的实现环境可能产生不同的结果。

其实到这里就可以看出来，`parseInt()`方法接收的第二个参数是具有重要意义的，它指定了转换的进制，然而`map()`回调函数`callback`中的第二个参数在这就是捣蛋的，它的第二个参数传进来的是数组索引，然后`parseInt()`却误以为它是进制基数，所以显而易见的错误就发生了。
要想完成想要的功能，可以这样做
```javascript
['10','10','10','10'].map(function(val) {
    return parseInt(val, 10);
});
```
这样就能保证每次将数组中每个字符串都转换为10进制的数字。
在[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt)中，`parseInt()`方法的第二个参数在缺省状态下也会有其对应的处理。
> 在没有指定基数，或者基数为 0 的情况下，JavaScript 作如下处理：
  1.如果字符串 string 以"0x"或者"0X"开头, 则基数是16 (16进制).
  2.如果字符串 string 以"0"开头, 基数是8（八进制）或者10（十进制），那么具体是哪个基数由实现环境决定。ECMAScript 5 规定使用10，但是并不是所有的浏览器都遵循这个规定。因此，永远都要明确给出radix参数的值。
  3.如果字符串 string 以其它任何值开头，则基数是10 (十进制)。
  4.如果第一个字符不能被转换成数字，parseInt返回NaN。

## 返回值
例子中的当返回16位包括16位9组成的数字的时候，都会出现的自动加1的情况，感觉和数值的精度有些关系吧，但是具体情况还没能查阅到相关资料，以后补上。

## 循环/闭包
其实正常单独使用**for**循环赋值的时候并不会出现这样的情况，关键在于这里赋值时使用了`function(){}`，产生了一个闭包。
曾经循环遍历元素为其绑定点击事件，想要获取每个元素各自的属性值的时候也这样写过
```javascript
var arr = document.getElementsByTagName("li");
for(var i = 0; i < arr.length;i++){
    arr[i].onclick = function(){
        alert(i);
    }
}
```
最后结果显而易见，不论单击哪一个元素其实都只会弹出一个相同的数字，那个数字就是`arr.length`。
循环体中等号'**=**'作为函数实例，这个函数实例产生了一个闭包，它引用外部闭包的变量，当外部闭包变量发生变化时内部闭包得到的值也会发生变化。说白了就是这样的传值方式不能够准确的将每个值都传入到内部闭包，而只会传入循环结束后的那个值。
```javascript
var a = [];
for (var i = 0; i < 10; i++) {
    (function(arg){
        a[arg] = function () {
            console.log(arg);
        };
    })(i)
}
a[1](); 
a[2]();
a[3]();
// 1,2,3
```
同理，循环遍历多个同类元素为其绑定事件的时候也要准确传入每个元素。