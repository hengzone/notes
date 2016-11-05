# 开篇
扩展程序和插件，这两个东西其实还是有些差别的，我一直以为他们是同一个东西，其实在**chrome**下还是有所差别。
在文档中一直提及**extension**的意思就是指扩展程序，我还奇怪不应该是**plugin**之类的，因为在其他软件中经常能看到这种那种的插件。
相比之下，扩展程序指的是通过调用**chrome**提供的**chrome API**来扩展浏览器功能的一种组件，工作在浏览器层面，用**html+css+javaScript**来开发制作。而插件，指的是通过调用**Webkit**内核**NPAPI**来扩展内核功能的一种组件，工作在内核层面，理论上可以用任何一种生成本地二进制程序的语言开发，比如 C/C++、Delphi 等。比如Flash player 插件，就属于这种类型。一般在网页中用`<object>`或者`<embed>`标签声明的部分，就要靠插件来渲染。
**PS：为了少打两个字，后文中出现的所有“插件”都代指“扩展程序”**。
## 概述
首先了解下扩展程序的工作机制
![架构](http://pan.plyz.net/d.asp?u=1611312563&p=567748-20160320224953115-380984028.jpg)
**chrome**插件中的文件大体上可以分成2部分：**chrome**插件中确确实实存在的文件，并且是应用程序级别的,如上图的**chrome Extension Scripts**以及注入到每个网页Dom当中的文件（如**Content Scripts**或者**Injected Scripts**）.这些文件都被放在**manifest.json**当中，Chrome内部会自动识别不同个文件的作用。
在任何时候，**popup**和**background**都只有一份，相比较起来，如果你有多个Tab(就是**chrome**当中的选项卡),那么**Content Scripts**和 **Injected Scripts**会运行在每一个Tab当中，也就是可以跨选项卡。当然，你可以指定往哪个Tab当中去注入Scripts,也就是说，注入操作是可选择的。
### Manifest.json
清单文件，设置扩展程序的一些基本信息，把**backgrounds**，**popups**，**content scripts**和**injected scripts**放在一起，并以结构化的方式书写。
### background
每个扩展程序只能有一个**background page**，后台网页是一个**HTML**页面，也可以是一个脚本，在扩展程序的进程中运行，整个生命周期都存在。
其主要用于管理浏览器本身的事物或状态，比如监听点击扩展程序图标事件，监听右键点击相应菜单，创建菜单更换图标等。
### popup page
点击扩张程序图标时会弹出的那个页面，这是一个**HTML**页面，是**default_popup**设置的，如果参数为空则不会弹出页面。
可以通过这个页面来设置、调整扩展程序的状态或查看扩张程序的工作状态。
### content scripts
内容脚本，是在网页的上下文中运行的**JavaScript**文件，就一般网页一样可以使用标准文档对象模型来获取、操作网页内容。
但是它与被打开页面自带的脚本处于不同的环境之下，所以不能对用户打开网页的变量或方法进行访问。
## 关系
！[相互关系](http://pan.plyz.net/d.asp?u=1611312563&p=567748-20160320232835709-1156243379.jpg)
在应用程序级别的部分是可以有互相访问的权限的。比如**Popup**文件能用`chrome.extension.getBackgroundPage()`访问**background**里面的东西，这就好像**Backbone**视图可以访问他们的**Model**和**Collection**。
**Content Scripts**是存在于每个独立的Dom页面，和**background**和**popup**用**message**的方式进行通信交流。特别的，它可以使用chrome.tabs.sendMessage和chrome.runtime.onMessage.addListener去向对方发送消息。这和**Backbone**的事件系统很像.
**Injected Scripts**和**Content Scripts**的不同在于它不能监听或者发送消息给其他的**chrome**插件部分。
## 初探 1.0
先制作一个能够增加鼠标右键选项的扩展程序。主要用到`chrome.contextMenus` API 向 Google Chrome 浏览器的右键菜单添加项目。
首先创建一个**manifest.json**
```json
{
  "manifest_version": 2,
  "name": "有道翻译插件",
  "version": "1.0",
  "description": "闲着蛋疼自己写的chrome扩展程序 -- by Felix",
  "icons": {
    "16": "icons/16.png",
    "32": "icons/32.png",
    "64": "icons/64.png"
  },
  "author": "Felix",
  "permissions": [
    "*://fanyi.youdao.com/*",
    "contextMenus",
    "tabs"
  ],
  "browser_action": {
    "default_icon": "icons/32.png",
    "default_title" : "划词翻译"
  },
  "background": {
    "scripts": ["myScript.js"],
    "persistent": false
  }
}
```
这里我选择[有道API](http://fanyi.youdao.com/openapi)，进去后申请接口得到**API key**和**keyfrom**就可以使用对应的接口获取查询结果了。
这里需要注意的是在`permissions`选项中需要加入请求的链接地址，这里使用的是有道的翻译接口**"*://fanyi.youdao.com/*"**，最后的星号不能漏了，本来以为只要域名全了就能访问了，结果在这个星号上折腾了好久。
接着创建需要引用到的`myScript.js`
```javascript
var API_key = 'xxx';
var keyfrom = 'xxx';

function selection(info, tab) {
    if (isChina(info.selectionText)) {
        var xhr = new XMLHttpRequest();
        var url = 'http://fanyi.youdao.com/openapi.do?keyfrom='+ keyfrom +'&key='+ API_key +'&type=data&doctype=json&version=1.1&q=';
        xhr.onreadystatechange = function(){
            if (xhr.readyState==4 && xhr.status==200){
                var res = xhr.responseText;
                alert(res);
            }
        };
        xhr.open("GET",url + info.selectionText,true);
        xhr.send(null); 
    }
}

function isChina(str) {
    if (/.*[\u4e00-\u9fa5]+.*$/.test(str)) { 
        return false; 
    } 
    return true;
}

var select = chrome.contextMenus.create({"title":"划词翻译","contexts":["selection"],"onclick":selection});
```
其中`API_key`和`keyfrom'是之前申请接口后有道提供的接口使用凭证。
这样在浏览器页面选中一些字符后点击右键就能看到一个**有道翻译**的选项，点击后就会执行上述程序段中的方法，先判断是否有中文，如果没有则向查询接口发出请求，如果请求成功就将请求内容弹出来。
到这里为止暂时已经达到目标。之后就是相关的优化了。
## 优化 2.0
先来做一个弹出框，就是点击插件按钮后能够显示的那个页面，在里面加上一个开关，用来控制一个功能打开或关闭。其实这个弹出框说白了也是一个网页，所以做起来还是很简单。
### 思路
利用**chrome**提供的[chrome.runtime](https://crxdoc-zh.appspot.com/extensions/runtime)接口来完成不同环境中**js**文件的通信，从而实现一个开关效果来控制插件的启用和停用。
这里完成的是`popup.js`（弹出框页面的**js**，负责实现按钮点击时向`myScript.js`传递“启用插件”这样的消息）、`tabs.js`（用户正在浏览的标签页的**js**，负责发起查询请求以及将查询结果显示在页面中，相当于用户每打开一个浏览器标签页时都会多引入这个**js**文件）和`myScript.js`（运行在浏览器后台的**js**，设置标志位，并提供通信接口，用于返回和设改变标志位的状态）三个文件之间的通信。
在我看来，不论在哪里也不论写多少个**chrome.runtime**都只会有一个通信通道在被使用。
利用`chrome.runtime.onMessage.addListener(function(message, sender, sendResponse) { do something })`就可以监听消息传递了。
就像这样
1. myScript.js
```javascript
if (!window.extension_translation_state) {
    window.extension_translation_state = false; // 默认关闭插件效果
}

chrome.runtime.onMessage.addListener(function(message, sender, sendResponse) {
  if (message.method === 'setTranState') {
    window.extension_translation_state = !window.extension_translation_state;
    sendResponse({
      method: 'returnTranState',
      message: window.extension_translation_state
    });
  }
  if (message.method === 'getTranState') {
    sendResponse({
      method: 'returnTranState',
      message: window.extension_translation_state
    });
  }
});
```
这里设置了一个标志位并监听两个消息事件，一个用于变更状态，一个用于返回当前状态。
标志位设置为一个**window**的全局变量，不然设置成**var**之类的隔一段时间就会被初始化还是被收回了，导致一小段时间后标志位就被重置了？暂时还不知道具体原因。。。
2. popup.js
```javascript
var checkbox = document.getElementById('translate');

chrome.runtime.sendMessage({
  method: 'getTranState'
}, function(response) {
  checkbox.checked = response.message;
});

checkbox.addEventListener("click", function(){
  chrome.runtime.sendMessage({
    method: 'setTranState'
  }, function(response) {
    checkbox.checked = response.message;
  });
});
```
这个文件在每次点击插件按钮弹出弹窗时都会加载一遍，所以先利用**chrome.runtime**请求得到插件的状态，然后为其绑定点击事件，当每次点击的时候都发送一个改变状态的消息。
3.popup.js
```javascript
document.onmouseup = function(e) {
  if (!e.button === 2) {
    return;
  }
  chrome.runtime.sendMessage({
    method: 'getTranState'
  }, function(response) {
    if (response.message) {

      // 插件处于开启转状态，则执行相关页面信息获取、发起查询请求和结果展示的操作

    }
  });
};
```
每次新打开一个页面都会将这个**js**文件加载，它会为这个页面多添加一个鼠标点击事件，并且通过消息通道获得当前插件的启用状态，如果处于启用状态则执行先关的操作。
### 其它改进
首先在原本的`manifest.json`文件中更改如下选项
```json
{
  ...

  "browser_action": {
    "default_icon": "icons/32.png",
    "default_title" : "Felix's extension",
    "default_popup": "popup/popup.html"
  },

  ...
}
```
这样就指定了那个弹出框所要用到的页面`popup.html`，接着就是制作弹出框页面，页面的修饰可以单独写一个样式表`popup.css`，以及这个页面中可能涉及到的一些操作事件可以写进`popup.js`
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>felix's extension</title>
  <link rel="stylesheet" href="../css/popup.css">
</head>
<body>
  <div class="contain">
    <span class="title fl">划词翻译</span>
    <input id="translate" class="checkbox fr" type="checkbox">
  </div>
  <script src="../js/popup.js"></script>
</body>
</html>
```
这样在点击插件按钮是就能够显示出这里制作的弹窗页面了。其实和正常的网页没啥两样。
结合之前的文件，这样就完成了这个划词翻译的全部制作。
## 使用
因为是第三方开发没有上传谷歌插件市场，用了一会之后就不能正常使用了。这里需要将这个插件的**ID**信息加入到**扩展程序白名单**中才行。
先下载[chrome.adm](http://pan.baidu.com/s/1hrZQCks)，然后`win + r`输入`gpedit.msc`之后右键**管理模块**中选择**添加/删除模块**，把刚才下载的东西添加进去。
添加完成后在**经典管理模板**下面依次找到**Google Chrome**→**扩展程序**→**配置扩展程序白名单**中把这个插件的**ID**添加进去后就可以长期使用了。
## 结语
说白了就是锻炼下阅读文档以及动手实践的能力，没有遇到什么难题，除此之外就是了解到了几个对象的操作。
也不是所谓的重复造轮子，确实是疲于各种划词插件的不靠谱，也许是我下载方式有问题吧。自己做一个也没啥坏处，反正闲着也是闲着。