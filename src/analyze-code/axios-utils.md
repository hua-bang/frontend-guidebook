---
nav:
  title: 读源码
  order: 8
group:
  title: 源码系列
  order: 1
title: Axios工具函数
order: 4
---

# Axios 工具函数

本文参加了由[公众号@若川视野](https://lxchuan12.gitee.io) 发起的[每周源码共读活动](https://juejin.cn/post/7079706017579139102)，点击了解详情一起参与。

## 前言

[`axios`](https://github.com/axios/axios)作为一个优秀的`HTTP`的请求库，提供各种请求方式，以及拦截器配置去让我们开发者用的更为方便。

这里的话，就简单分析`axios`库中的工具函数，并不涉及`axios`的具体刘晨和业务。

### 项目运行

```shell
git clone https://github.com/axios/axios.git

npm install

npm run examples
```

### 项目结构分析

这里，仅简单分析项目的基本目录

```txt
- .github   github配置
- dist      打包产物
- examples  案例
- lib       源码
- sandbox   sandbox
- test      测试
```

在这里，我们仅分析`lib`中的`utils`。

## 解读

### bind

> 这里实际上就是`bind`使用，但为了兼容旧版游览器，所以做了垫片，使用`apply`进行实现。

```js
var bind = require('./helpers/bind');

// bind.js
'use strict';

module.exports = function bind(fn, thisArg) {
  return function wrap() {
    var args = new Array(arguments.length);
    for (var i = 0; i < args.length; i++) {
      args[i] = arguments[i];
    }
    return fn.apply(thisArg, args);
  };
};
```

### toString

> 实质上就是`Object.prototype.toString`,不过调用方式会有点变化，得使用`call`或者`apply`去绑定`this`的指向。

```js
var toString = Object.prototype.toString;
```

### kindOf

> 这里返回了一个函数，用于判断变量的类型，返回的是字符串。同时，我们看到这里使用了闭包，起到缓存的作用。

```js
var kindOf = (function(cache) {
  // eslint-disable-next-line func-names
  return function(thing) {
    var str = toString.call(thing);
    return cache[str] || (cache[str] = str.slice(8, -1).toLowerCase());
  };
})(Object.create(null));
```

### kindOfTest

> 这里是一个高阶函数，用来返回判断变量是否是某个类型的函数。这里也使用了闭包。

```js
function kindOfTest(type) {
  type = type.toLowerCase();
  return function isKindOf(thing) {
    return kindOf(thing) === kind;
  }
}
```

### supportedProtocols

> axios 支持的协议
 
```js
var supportedProtocols = ['http:', 'https:', 'file'];
```

### protocol

> 得到请求的协议，默认为`http:`

```js
function getProtocol(protocol) {
  return protocol || 'http:';
}
```

### isArray

> 判断变量是否是数组

```js
function isArray(val) {
  return Array.isArray(val);
}
```

### isUndefined
> 判断变量是否定义
```js
function isUndefined(val) {
  return typeof val === 'undefined';
}
```

### isBuffer
> 判断变量是否是Buffer类型
```js
function isBuffer(val) {
  return val !== null && !isUndefined(val) && val.constructor !== null && !isUndefined(val.constructor)
    && typeof val.constructor.isBuffer === 'function' && val.constructor.isBuffer(val);
}
```

### isArrayBuffer
> 判断是否是`ArrayBuffer`

```js
var isArrayBuffer = kindOfTest('ArrayBuffer');
```

### isArrayBufferView

> 判断是否是`ArrayBufferView`

```js
function isArrayBufferView(val) {
  var result;
  if ((typeof ArrayBuffer !== 'undefined') && (ArrayBuffer.isView)) {
    result = ArrayBuffer.isView(val);
  } else {
    result = (val) && (val.buffer) && (isArrayBuffer(val.buffer));
  }
  return result;
}
```

### isString
> 判断是否是字符串

```js
function isString(val) {
  return typeof val === 'string';
}
```

### isNumber
> 判断是否是`Number`
```js
function isNumber(val) {
  return typeof val === 'number';
}
```

### isObject
> 判断变量是否是对象
```js
function isObject(val) {
  return val !== null && typeof val === 'object';
}
```

### isPlainObject
> 判断是否是纯净对象

```js
function isPlainObject(val) {
  if (kindOf(val) !== 'object') {
    return false;
  }

  var prototype = Object.getPrototypeOf(val);
  return prototype === null || prototype === Object.prototype;
}
```

### isDate
> 判断变量是否是`Date`对象
```js
var isDate = kindOfTest('Date');
```

### isFile
> 判断变量是否是`File`对象
```js
var isFile = kindOfTest('File');
```

### isBlob
> 判断变量是否是`blob`对象
```js
var isBlob = kindOfTest('Blob');
```

### isFileList
> 判断变量是否是`FileList`对象

```js
var isFileList = kindOfTest('FileList');
```

### isFunction
> 判断变量是不是函数类型

```js
function isFunction(val) {
  return toString.call(val) === '[object Function]';
}
```

### isStream
> 判断是否是`Stream`类型

```js
function isStream(val) {
  return isObject(val) && isFunction(val.pipe);
}
```

### isFormData

> 判断是否是`FormData`类型,这里还做了垫片，兼容。

```js
function isFormData(thing) {
  var pattern = '[object FormData]';
  return thing && (
    (typeof FormData === 'function' && thing instanceof FormData) ||
    toString.call(thing) === pattern ||
    (isFunction(thing.toString) && thing.toString() === pattern)
  );
}
```

### isURLSearchParams

> 判断是否`isURLSearchParams`

```js
var isURLSearchParams = kindOfTest('URLSearchParams');
```

### trim

> `trim`去除空格，这里也做了垫片。

```js
function trim(str) {
  return str.trim ? str.trim() : str.replace(/^\s+|\s+$/g, '');
}
```

### isStandardBrowserEnv

> 判断是否标准的游览器环境

```js
function isStandardBrowserEnv() {
  if (typeof navigator !== 'undefined' && (navigator.product === 'ReactNative' ||
                                           navigator.product === 'NativeScript' ||
                                           navigator.product === 'NS')) {
     return false;
  }
  return (
    typeof window !== 'undefined' &&
    typeof document !== 'undefined'
  );
}
```

### forEach

> 类似数组的`forEach`,当我们的参数不是对象的时候，会构造一个数组，当我们的参数是数组的时候，遍历每个元素；当参数是对象的时候，遍历每个键值。

```js
function forEach(obj, fn) {
  // Don't bother if no value provided
  if (obj === null || typeof obj === 'undefined') {
    return;
  }

  // Force an array if not already something iterable
  if (typeof obj !== 'object') {
    /*eslint no-param-reassign:0*/
    obj = [obj];
  }

  if (isArray(obj)) {
    // Iterate over array values
    for (var i = 0, l = obj.length; i < l; i++) {
      fn.call(null, obj[i], i, obj);
    }
  } else {
    // Iterate over object keys
    for (var key in obj) {
      if (Object.prototype.hasOwnProperty.call(obj, key)) {
        fn.call(null, obj[key], key, obj);
      }
    }
  }
}
```

### merge

> 合并多个对象

```js
function merge(/* obj1, obj2, obj3, ... */) {
  var result = {};
  function assignValue(val, key) {
    if (isPlainObject(result[key]) && isPlainObject(val)) {
      result[key] = merge(result[key], val);
    } else if (isPlainObject(val)) {
      result[key] = merge({}, val);
    } else if (isArray(val)) {
      result[key] = val.slice();
    } else {
      result[key] = val;
    }
  }

  for (var i = 0, l = arguments.length; i < l; i++) {
    forEach(arguments[i], assignValue);
  }
  return result;
}
```

### extend
> 继承方法的实现，中间也使用了`foreach`

```js
function extend(a, b, thisArg) {
  forEach(b, function assignValue(val, key) {
    if (thisArg && typeof val === 'function') {
      a[key] = bind(val, thisArg);
    } else {
      a[key] = val;
    }
  });
  return a;
}
```

### stripBOM

> BOM（Byte Order Mark）：字节顺序标记，出现在文本文件头部，Unicode编码标准中用于标识文件是采用哪种格式的编码。

BOM字符虽然起到了标记文件编码的作用，其本身却不属于文件内容的一部分，如果读取文本文件时不去掉BOM，在某些使用场景下就会有问题。例如我们把几个JS文件合并成一个文件后，如果文件中间含有BOM字符，就会导致浏览器JS语法错误。因此，使用Node.js读取文本文件时，一般需要去掉BOM。

```js
function stripBOM(content) {
  if (content.charCodeAt(0) === 0xFEFF) {
    content = content.slice(1);
  }
  return content;
}
```

### inherits

> 继承方法

```js
function inherits(constructor, superConstructor, props, descriptors) {
  constructor.prototype = Object.create(superConstructor.prototype, descriptors);
  constructor.prototype.constructor = constructor;
  props && Object.assign(constructor.prototype, props);
}
```

### toFlatObject

> 打平对象

```js
function toFlatObject(sourceObj, destObj, filter) {
  var props;
  var i;
  var prop;
  var merged = {};

  destObj = destObj || {};

  do {
    props = Object.getOwnPropertyNames(sourceObj);
    i = props.length;
    while (i-- > 0) {
      prop = props[i];
      if (!merged[prop]) {
        destObj[prop] = sourceObj[prop];
        merged[prop] = true;
      }
    }
    sourceObj = Object.getPrototypeOf(sourceObj);
  } while (sourceObj && (!filter || filter(sourceObj, destObj)) && sourceObj !== Object.prototype);

  return destObj;
}
```

### endWith

> 判断字符是否以目标字符串结尾

```js
function endsWith(str, searchString, position) {
  str = String(str);
  if (position === undefined || position > str.length) {
    position = str.length;
  }
  position -= searchString.length;
  var lastIndex = str.indexOf(searchString, position);
  return lastIndex !== -1 && lastIndex === position;
}
```

### toArray

> 转换成新数组

```js
function toArray(thing) {
  if (!thing) return null;
  var i = thing.length;
  if (isUndefined(i)) return null;
  var arr = new Array(i);
  while (i-- > 0) {
    arr[i] = thing[i];
  }
  return arr;
}
```

### isTypedArray

> 判断是否是TypedArray.参考[TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)

```js
var isTypedArray = (function(TypedArray) {
  // eslint-disable-next-line func-names
  return function(thing) {
    return TypedArray && thing instanceof TypedArray;
  };
})(typeof Uint8Array !== 'undefined' && Object.getPrototypeOf(Uint8Array));

```

## 总结

我们会发现其实工具函数的源码相比于其他的源码还是比较易懂的，因为工具函数一般是比较`基础，逻辑复用`的函数。

在其中有很多优秀的案例，使用了`高阶函数`，`闭包`等特性来实现。我们要理解他的逻辑，同时我们也要理清这些函数的意义。

## 参考
- [阅读axios源码，发现了这些实用的基础工具函数](https://juejin.cn/post/7042610679815241758#heading-16)
- [source code](https://github.dev/axios/axios/tree/master/lib)