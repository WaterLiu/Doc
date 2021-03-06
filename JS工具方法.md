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

### mobx set remoeve path 方法 （目前不支持嵌套数组）
 ```javascript
    import * as mobx from 'mobx';
    set(obj, path, value) {
        const keys = path.split('.');
        for (let i = 0; i < keys.length - 1 ; i++) {
            const key = keys[i];
            if (obj[key] === undefined) {
                mobx.set(obj, key, {});
            }
            obj = obj[key];
        }
        mobx.set(obj, keys[keys.length - 1], value);
    }

    remove(obj, path) {
        const keys = path.split('.');
        for (let i = 0; i < keys.length - 1 ; i++) {
            const key = keys[i];
            if (obj[key] === undefined) {
               return;
            }
            obj = obj[key];
        }
        mobx.remove(obj, keys[keys.length - 1]);
    }
 ```
 
 ### merge 按照 rule 规则merge 支持自定方法回调  不支持数组
  ```javascript
     mergeWith(obj, source, rule, type) {

        let set, remove;
        if (mode === 'mobx') {
            set = mobx.set;
            remove = mobx.remove;
        } else {
            set = _.set;
            remove = _.remove;
        }

        const ruleKeys = _.keys(rule);
        for (let i = 0; i < ruleKeys.length; i++) {
            const ruleKey = ruleKeys[i];
            //如果是方法就回调出去.
            if (_.isFunction(rule[ruleKey])) {
                rule[ruleKey](obj, source, ruleKey, _.get(source, ruleKey));
            } else { // 不是方法，正常合并
                if (_.has(source, ruleKey)) {
                    set(obj, rule[ruleKey], _.get(source, ruleKey));
                } else {
                    if (_.has(obj, rule[ruleKey])) {
                        remove(obj, rule[ruleKey]);
                    }
                }
            }
        }
    }
    
    
     //customizerMergeArray 参数 obj, source, objPath, srcPath, value
    mergeWith(obj, source, rule, customizerMergeArray) {

        let set, remove;
        if (mobx.isObservable(obj)) {
            set = mobx.set;
            remove = mobx.remove;
        } else {
            set = _.set;
            remove = _.remove;
        }

        const ruleKeys = _.keys(rule);
        for (let i = 0; i < ruleKeys.length; i++) {
            const ruleKey = ruleKeys[i];
            //如果是方法就回调出去.
            if (_.isFunction(rule[ruleKey])) {
                rule[ruleKey](obj, source, ruleKey, _.get(source, ruleKey));
            } else { // 不是方法，正常合并
                if (_.has(source, ruleKey)) {
                    const srcObj = _.get(source, ruleKey);
                    if ( Array.isArray(srcObj) || mobx.isObservableArray(srcObj)) {
                        customizerMergeArray(obj, source, rule[ruleKey], ruleKey, _.get(source, ruleKey));
                    } else {
                        set(obj, rule[ruleKey], _.get(source, ruleKey));
                    }
                } else {
                    if (_.has(obj, rule[ruleKey])) {
                        remove(obj, rule[ruleKey]);
                    }
                }
            }
        }
    }
 ```
 
  ### 获取对象上的 所有 paths
  ```javascript
   function paths(obj, parentKey) {
    const result = [];
    if (_.isArray(obj) || isObservableArray(obj)) {
        let index = 0;
        _.forEach(obj, (item) => {
            if (_.isObject(item)) {
                const pathList = paths(item, '[' + index++ + ']');
                _.forEach(pathList, (path) => {
                    const curPath = (parentKey ? parentKey : '') + path;
                    result.push(curPath);
                });
            }
        });

        if (result.length == 0) {
            result.push(parentKey);
        }
    }
    else if (_.isObject(obj)) {
        _.forEach(_.keys(obj), (key) => {
            const pathList = paths(obj[key], key);
            _.forEach(pathList, (path) => {
                const curPath = (parentKey ? parentKey + '.' : '') + path;
                result.push(curPath);
            });
        });
    }
    else {
        result.push(parentKey);
    }
    return result;
}

 ```
