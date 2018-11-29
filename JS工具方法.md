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

### mobx merge，功能和lodash中的merge相同。 可以将source中新增的key-value键值对加入到object的observable属性中
 
 ```javascript
export function mobxMerge(object, source) {
    mergeWith(object, source, (objValue, srcValue, key, object, source) => {
        if (typeof objValue === 'undefined') {
            let obj = {};
            obj[key] = srcValue;

            extendObservable(object, obj);
            return srcValue;
        }

    });
}
 ```

### mobx merge，功能和lodash中的mergeWith相同。 可以将source中新增的key-value键值对加入到object的observable属性中
 
 ```javascript
export function mobxMergeWith(object, source, customizer) {
    mergeWith(object, source, (objValue, srcValue, key, object, source, stack) => {
        if (typeof objValue === 'undefined') {
            let obj = {};
            obj[key] = srcValue;

            extendObservable(object, obj);
            return customizer ? customizer(objValue, srcValue, key, object, source, stack) : srcValue;
        }

    });
}
 ```

### lodash set 方法 （目前不支持嵌套数组）
 ```javascript
export function set(obj, path, value) {
    const p = path.split('.');
    const length = p.length;

    if (length < 1) throw new TypeError('advHas error');

    let newObj = {};
    for (let i = length - 1; i >= 0; i--) {
        const keyObject = p[i];
        if (i === length - 1) {
            newObj[keyObject] = value;
        }
        else {
            if (i === 0) {
                obj[keyObject] = newObj;
            } else {
                let temp = {};
                temp[keyObject] = newObj;
                newObj = temp;
            }
        }
    }
}
 ```
