# 工具方法
----
##工具库

[Lodash 是一个一致性、模块化、高性能的 JavaScript 实用工具库: https://www.lodashjs.com/](https://www.lodashjs.com/)


### 判断对象（包括 原型对象）判断是否包含一个属性，支持Getter/Setter
```javascript
function hasProperty(obj, prop, gs) {
    let desc;
    do {
        desc = Object.getOwnPropertyDescriptor(obj, prop);
    } while (!desc && (obj = Object.getPrototypeOf(obj)));

    if (typeof desc === 'undefined') {
        return false;
    } else {
        if (gs !== undefined) {
            return typeof desc[gs] !== 'undefined';
        } else {
            return true;
        }
    }
}
```
#### Example
```javascript
class Example {
    constructor(props) {

    }

    get testGet() {
        return 'testGetValue';
    }
}

const example = new Example();
console.log(hasProperty(example, 'testGet', 'get'));
console.log(hasProperty(example, 'testGet', 'set'));
console.log(hasProperty(example, 'test'));

```
-----
