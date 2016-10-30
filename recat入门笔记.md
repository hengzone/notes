# react基础学习
基本语法的认识与学习。
参照[阮一峰的react入门实例教程](http://www.ruanyifeng.com/blog/2015/03/react.html)。其中给出很多基础demo，参照其中的源码对**react**做一个基础的了解。
[react原文档](https://facebook.github.io/react/docs/hello-world.html)参考。
## react介绍
* React不是一个完整的MVC框架，最多可以认为是MVC中的V（View），甚至React并不非常认可MVC开发模式；
* React的服务器端Render能力只能算是一个锦上添花的功能，并不是其核心出发点，事实上React官方站点几乎没有提及其在服务器端的应用；
* React不是一个新的模板语言，JSX只是一个表象，没有JSX的React也能工作。
我觉得web开发站在前端角度就是一个与用户交互的过程，那么多的按钮和不同的事件，完成事件后的页面元素的变动，等等这些，随着需求的逐渐演变，这其中的复杂度越变越高。
> 复杂或频繁的DOM操作通常是性能瓶颈产生的原因（如何进行高性能的复杂DOM操作通常是衡量一个前端开发人员技能的重要指标）。

好比是如今的大多数电商商城的主页，那么多的产品，还包括不同的分类、详情、相互间的关系，种种这些元素都要包含在一个页面中而且还要保证这些板块间的相互统一，虽然难以想象是如何做到的，但是绝对不可能像我原来用**jQuery**写出来的那意大利面条般的代码。
### ReactJS的背景和原理
React为此引入了**虚拟DOM（Virtual DOM）**的机制：在浏览器端用Javascript实现了一套DOM API。基于React进行开发时所有的DOM构造都是通过虚拟DOM进行，每当数据变化时，React都会重新构建整个DOM树，然后React将当前整个DOM树和上一次的DOM树进行对比，得到DOM结构的区别，然后仅仅将需要变化的部分进行实际的浏览器DOM更新。而且React能够批处理虚拟DOM的刷新，在一个事件循环（Event Loop）内的两次数据变化会被合并，例如你连续的先将节点内容从A变成B，然后又从B变成A，React会认为UI不发生任何变化，而如果通过手动控制，这种逻辑通常是极其复杂的。尽管每一次都需要构造完整的虚拟DOM树，但是因为虚拟DOM是内存数据，性能是极高的，而对实际DOM进行操作的仅仅是Diff部分，因而能达到提高性能的目的。这样，在保证性能的同时，开发者将不再需要关注某个数据的变化如何更新到一个或多个具体的DOM元素，而只需要关心在任意一个数据状态下，整个界面是如何Render的。
### 组件化
**虚拟DOM(virtual-dom)**不仅带来了简单的UI开发逻辑，同时也带来了组件化开发的思想，所谓组件，即封装起来的具有独立功能的UI部件。React推荐以组件的方式去重新思考UI构成，将UI上每一个功能相对独立的模块定义成组件，然后将小的组件通过组合或者嵌套的方式构成大的组件，最终完成整体UI的构建。
React认为一个组件应该具有如下特征：

* 可组合（Composeable）：一个组件易于和其它组件一起使用，或者嵌套在另一个组件内部。如果一个组件内部创建了另一个组件，那么说父组件拥有（own）它创建的子组件，通过这个特性，一个复杂的UI可以拆分成多个简单的UI组件；
* 可重用（Reusable）：每个组件都是具有独立功能的，它可以被使用在多个UI场景；
* 可维护（Maintainable）：每个小的组件仅仅包含自身的逻辑，更容易被理解和维护。
## 基础模板
后续只记录**react**相关代码，遂使用同统一固定的**html**模板。
```html
<!DOCTYPE html>
<html>
<head>
    <meta content="text/html; charset=utf-8" />
    <title>demo-7</title>
    <script src="../build/react.js"></script>
    <script src="../build/react-dom.js"></script>
    <script src="../build/browser.min.js"></script>
</head>
<body>
<div id="demo"></div>
<script type="text/babel">
    <!-- 在这里写入代码 -->
</script>
</body>
</html>
```
首先是上面用到的三个库：**react.j**、**react-dom.js**、**browser.min.js**，**react.js**是**React**的核心库，**react-dom.js**是提供与 **DOM**相关的功能，**Browser.js**的作用是将**JSX**语法转为**JavaScript**语法，这一步很消耗时间，实际上线的时候，应该将它放到服务器完成。
乘着前天看**git**的余热，这里用**git**下载**react**
```linux
git clone https://github.com/facebook/react
```
当然也可以直接去(react官网)[https://facebook.github.io/react/]上去下载压缩文件，里面包含了需要用到的库文件。按照路径引用就可以了。
其中需要注意的是`<script>`标签里的**type**属性是`text/babel`，这是**react**独有的**jsx**语法，所有用到**jsx**的地方都要加上`type="text/babel"`，因为它与**JavaScript**语法不兼容。
## ReactDOM.render()
这是**react**的最基本方法，用于将内容插入到页面的指定区域中去。
```jsx
const element = <h1>Hello, world</h1>;
ReactDOM.render(
    element,
    document.getElementById('demo')
);
```
上面代码将一个标题插入到ID名为'demo'的一个元素中去。
## 更新页面元素
页面元素就像是电影中的一帧画面一样，它代表着界面UI在这个时间点上所呈现的内容。
在**react**中，元素一经创建就不在可修改，想要修改需要再次调用`ReactDOM.render()`，下面这个时间更新就能说明问题了
```jsx
function tick() {
    const element = (
        <h1>It is {new Date().toLocaleTimeString()}.</h1>
    );
    ReactDOM.render(
        element,
        document.getElementById('demo')
    );
}

setInterval(tick, 1000);
```
上面将元素和插入动作都写成了一个`tick()`方法，利用`setInterval()`每隔一秒就去调用一次`tick()`，这样就能在页面上时刻刷新时间。
在**react**中，这样的刷新只会改变需要改变部分的页面元素和不会整个的刷新其父元素或整个子元素，这样的刷新能够提高页面的整体性能。
## 组件化
可以将页面中的内容划分为多个组件，然后再通过组合组件的方式来形成最终的页面。
由于暂时还没细致了解**ES6**的那些东西，所以这里使用`React.createClass()`来生成一个组件
```jsx
var Html = React.createClass({
    render: function(){
        return <h1>hello,{this.props.name}</h1>;
    }
});

ReactDOM.render(
    <Html name="Felix"/>,
    document.getElementById('demo')
);
```
在上面的代码中，变量`Html`就是一个组件类，所有组件都必须有自己的`render`方法，用于输出组件。
这里需要注意的是
**
1.组件类的第一个字母必须是大写的，否则会报错；
2.组件中不能同时包含两个顶层标签，否则也会报错；
3.`class`属性需要写成`className`，`for`属性需要写成`htmlFor`，这是因为`class`和`for`是 JavaScript 的保留字。
**
PS：他们很喜欢将一些在我看起来已经算是很颗粒化的模块再分成更细小的模块，直到不能再细分下去为止。
## this.props.children
`this.props.children`属性表示组件的所有子元素。
`this.props.children`的值有三种可能：如果当前组件没有子节点，它就是`undefined`；如果有一个子节点，数据类型是`object`；如果有多个子节点，数据类型就是`array`。
我们可以使用**react**提供的[React.Children](https://facebook.github.io/react/docs/react-api.html#react.children)方法来处理这样模糊不定的数据结构。使用`React.Children.map()`或者`React.Children.forEach()`来遍历进行操作。这里使用前者，因为后者不会返回任何结果。这里需要返回插入页面的元素
```jsx
var Notes = React.createClass({
    render: function() {
        return (
            <ol>
                {
                    React.Children.map(this.props.children, function (child) {
                        return <li>{child}</li>;
                    })
                }
            </ol>
        );
    }
});

ReactDOM.render(
    <Notes>
        <span>hello</span>
        <span>world</span>
    </Notes>,
    document.body
);
```
## 验证
其实看到这里我已经发觉**react**和原来遇到的**yii2**、**ThinkPHP**这两个**PHP**框架中的验证保护简直出奇的相似。细思极恐啊，前端都已经做到这种份上了么？难道前端的模块化工程已经演变成后写后台差不多么？在**ES6**中已经是各种类、各种继承了。可怕啊……
组件可以接收任意形式的参数，但有时候可能需要验证数据的类型是否符合要求。利用组件的`propTypes`属性来验证实例化的组件属性是否符合要求，还可以通过`getDefaultProps()`方法来设置默认的属性值
```jsx
var Title = React.createClass({
    // 对输入参数个类型做限制。
    // 如果类型不符会报警告。
    propTypes: {
        title: React.PropTypes.string.isRequired,
    },

    // 设置默认属性
    getDefaultProps : function () {
        return {
            title : 'Hello World'
        };
    },

    render: function() {
        return <h1> {this.props.title} </h1>;
    }
});

var data = '123';

ReactDOM.render(
    <Title title={data} />,
    document.getElementById('demo')
);
```
如果传入的值为非字符串类型的数据时就会在控制台输出一句警告。此外，就算没有设置`data`属性也可以调用到默认设置好的属性值。
## 获取真实的DOM节点
由于**react**提供的虚拟DOM，这只是存在于内存中的一种数据结构，并不是真实的DOM节点，所有的DOM变动都会先反应在虚拟DOM上，然后会自行比较发生改变的部分，将改变部分的内容反映到真实DOM节点上。之前的[更新页面元素](#_3)就能很好的说明这一点。这是一种称为[DOM diff](http://calendar.perfplanet.com/2013/diff/)的算法，它可以极大的提高页面元素的表现能力。
从组件中获取真实的DOM需要借助`ref`属性的帮助
```jsx
var Component = React.createClass({
    handleClick: function() {
        this.refs.myTextInput.focus();
    },

    handleDbClick: function() {
        alert('Double Click');
    },

    handleFocus: function() {
        console.log(this.refs.myTextInput.value);
    },

    render: function() {
        return (
            <div>
                <input type="text" ref="myTextInput" onFocus={this.handleFocus} onCut={this.handleFocus}/>
                <input type="button" value="Focus the text input" onClick={this.handleClick} onDoubleClick={this.handleDbClick} />
            </div>
        );
    }
});

ReactDOM.render(
    <Component />,
    document.getElementById('demo')
);
```
上述`Component`组件中有一个输入框和一个按钮，在输入框中有一个`ref`属性，然后可以通过`this.ref.refName`来获取真实DOM节点。
> 需要注意的是，由于`this.refs.refName`属性获取的是真实 DOM ，所以必须等到虚拟 DOM 插入文档以后，才能使用这个属性，否则会报错。

从代码中可以看到，可以很方便的在元素内就指定其在某些触发情况的处理函数。在输入框有聚焦、剪切触发的时候会有相关的处理，同样的按钮在单击和双击时也会触发相关的事件，有个小问题是这里的双击同样会触发单击事件的处理方法。
[文档](https://facebook.github.io/react/docs/events.html#supported-events)中有更多触发类型的说明。
## 生命周期
