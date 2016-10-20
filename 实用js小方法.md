# 记录简单实用的JS方法
平时走过路过偶尔见过的**JavaScript**方法
## 判断数据类型
利用`Object.prototype.tostring`方法获得返回对象类型的字符串，从而达到判断类型的目的。
通过`call()`方法使得任意对象可直接调用上述方法从而获得其类型。
```javascript
Object.prototype.toString.call(obj)
```
不同的数据类型返回结果如下
> * 数值：返回[object Number]。
> * 字符串：返回[object String]。
> * 布尔值：返回[object Boolean]。
> * undefined：返回[object Undefined]。
> * null：返回[object Null]。
> * 数组：返回[object Array]。
> * arguments对象：返回[object Arguments]。
> * 函数：返回[object Function]。
> * Error对象：返回[object Error]。
> * Date对象：返回[object Date]。
> * RegExp对象：返回[object RegExp]。
> * 其他对象：返回[object " + 构造函数的名称 + "]。
利用其返回值可以轻松获取到该对象的类型。
由此，可以得到一个比`typeof()`更精确的类型判断函数
```javascript
var type = function (o){
  var s = Object.prototype.toString.call(o);
  return s.match(/\[object (.*?)\]/)[1].toLowerCase();
};

type({}); // "object"
type([]); // "array"
type(5); // "number"
type(null); // "null"
type(); // "undefined"
type(/abcd/); // "regex"
type(new Date()); // "date"
```
在上面这个type函数的基础上，还可以加上专门判断某种类型数据的方法
```javascript
['Null',
 'Undefined',
 'Object',
 'Array',
 'String',
 'Number',
 'Boolean',
 'Function',
 'RegExp',
 'NaN',
 'Infinite'
].forEach(function (t) {
  type['is' + t] = function (o) {
    return type(o) === t.toLowerCase();
  };
});

type.isObject({}) // true
type.isNumber(NaN) // true
type.isRegExp(/abc/) // true
```
## 