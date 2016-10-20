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
## 浮点数
需要说明的是，因为计算机是二进制来表示浮点数的，所以无法准确表示一个浮点数，只能逼近。
在上述例子中`0.1+0.2=0.30000000000000004`，所以如果变成
```javascript
(function() {
    return (0.1 + 0.2 === 0.30000000000000004);
})();
// true
```
[精确浮点数计算](http://acm.whu.edu.cn/starter/problem/detail?problem_id=1069)，在这里可以看到很多浮点数的计算结果与理想值的都存在一点点细微的差距。
## sort()
这其实是**JavaScript Array**对象的方法，如果在使用该方法时没有使用参数，将按字母顺序对数组中的元素进行排序，说得更精确点，是按照字符编码的顺序进行排序。