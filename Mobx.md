# Doc
[Mobx 官网文档：https://cn.mobx.js.org/](https://cn.mobx.js.org/)


###
Mobx对象： mobx 转换 JS （or 有原型） 对象时会把信息添加到该对象的 __proto__ 里面。
以下Case，是有问题得：
 
 ```javascript
 const mobxObject = observable({frame: {x: 10, y: 20}});
 const jsObject = toJS(mobxObject);
 const proto = Object.getPrototypeOf(this.currentElement);  //这里得到的proto 是含有mobxObject 的mobx 信息的原型
 Object.setPrototypeOf(jsObject,proto);                     // 设置到jsObject 元信息，携带了mobxObject 的 mobx 对象信息。
```
