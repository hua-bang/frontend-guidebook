---
nav:
  title: 读源码
  order: 8
group:
  title: 源码系列
  order: 1
title: Vue2工具函数
order: 1
---

# Vue2工具函数

本文参加了由[公众号@若川视野](https://lxchuan12.gitee.io) 发起的[每周源码共读活动](https://juejin.cn/post/7079706017579139102)，点击了解详情一起参与。

## 前言

本次我们主要阅读的是`Vue2`中的`shared`里面编写的工具函数，[链接传送门](https://github.com/vuejs/vue/blob/dev/src/shared/util.js)，由于代码是结合`flow`，可能有一点前置成本，所以我们只看打包后的[dist对应部分](https://github.com/vuejs/vue/blob/dev/dist/vue.js#L14-L379)

## 解读

### EmptyObject

> 这里是构造了一个冻结对象，无法对这个对象进行属性修改。可以参考[`Object.freeze`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)。

```js
var empty = Object.freeze({});
```

### isUndef

> 用来判断变量是否未定义或为空。

```js
function isUndef(v) {
  return v === undefined || v === null;
}
```

### isDef

> 判断变量是否定义或不为空。

```js
function isDef(v) {
  return v !== undefined && v !== null;
}
```

### isTrue 

> 判断是否为True。

```js
function isTrue(v) {
  return v === true;
}
```

### isFalse

> 判断是否为false。

```js
function isFalse(v) {
  return v === false;
}
```

### isPrimitive 
> 判断变量是否是原始值。

```js
function isPrimitive(value) {
  return (
    typeof value === 'string' ||
    typeof value === 'number' ||
    // $flow-disable-line
    typeof value === 'symbol' ||
    typeof value === 'boolean'
  )
}
```

### isObject

> 判断变量是否是对象，这里会检查是否是`null`;

```js
function isObject(obj) {
  return obj !== null && (typeof obj === 'object);
}
```

### toRawType

> 这个函数返回的是变量类型的字符串，借助了[`Object.prototype.toString`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)函数的方法。

```js
var _toString = Object.prototype.toString;

function toRawType (value) {
  return _toString.call(value).slice(8, -1)
}
```

### isPlainObject

> 判断对象是否是纯对象

```js
function isPlainObject (obj) {
  return _toString.call(obj) === '[object Object]'
}
```

### isRegExp

> 判断是否是正则表达式

```js
function isRegExp (v) {
  return _toString.call(v) === '[object RegExp]'
}
```

### isValidArrayIndex

> 判断是否可用的数组索引值

```js
function isValidArrayIndex (val) {
  var n = parseFloat(String(val));
  return n >= 0 && Math.floor(n) === n && isFinite(val)
}
```

### isPromise

> 判断是否是`Promise`

```js
function isPromise(val) {
  return (
    isDef(val) &&
    typeof val.then === 'function' &&
    typeof val.catch === 'function'
  )
}
```

### toString

> 将变量转成字符串，当变量是对象的时候使用`JSON.stringify`。

```js
function toString (val) {
  return val == null
    ? ''
    : Array.isArray(val) || (isPlainObject(val) && val.toString === _toString)
    ? JSON.stringify(val, null, 2)
    : String(val)
}
```

### toNumber

> 将变量转成数字，失败则返回原始字符串

```js
function toNumber(val) {
  var n = parseFloat(val);
  return isNaN(n) ? val : n;
}
```

### makeMap

> 这个函数会返回一个检测函数，判断指定的`key`有没有在生成的`map`中。

首先，该函数会生成根据传入的变量进行分割，并把分割后的值，放入`map`中，最后返回这个`map`的检测函数。

注意：这里没有直接返回`map`, 而是返回一个函数，很巧妙地运用了闭包，防止修改。

```js
function makeMap(
  str,
  expectsLowerCase
) {
  var map = Object.create(null);
  var list = str.split(',');
  for (var i = 0; i < list.length; i++) {
      map[list[i]] = true;
    }
  return expectsLowerCase
    ? function (val) { return map[val.toLowerCase()]; }
    : function (val) { return map[val]; }
}
```

### isBuiltInTag

> 判断是否是内置的`tag`

```js
var isBuiltInTag = makeMap('slot,component', true);
```

### isReservedAttribute

> 判断是否是保留地属性

```js
var isReservedAttribute = makeMap('key,ref,slot,slot-scope,is');
```

### Remove

> 主要是借助了[splice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice), 去除数组中指定元素

```js
function remove (arr, item) {
  if (arr.length) {
    var index = arr.indexOf(item);
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```

### hasOwn
> 借助了`call`以及`hasOwnProperty`来进行实现，检查属性是否属于自身。

```js
var hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn (obj, key) {
  return hasOwnProperty.call(obj, key)
}
```

### cached

> 缓存函数，这里实质上也是一个高阶函数了，同时利用了闭包地特性。

```js
function cached(fn) {
  const cache = Object.create(null);
  return (function cachedFn(key) {
    const hit = cache[key];
    return hit || (cache[key] = fn(key));
  })
}
```

### camelize

> 连字符转小驼峰
```js
var camelizeRE = /-(\w)/g;
var camelize = cached(function (str) {
  return str.replace(camelizeRE, function (_, c) { return c ? c.toUpperCase() : ''; })
});
```

### capitalize 
> 首字母转大写

```js
var capitalize = cached(function (str) {
  return str.charAt(0).toUpperCase() + str.slice(1)
});
```

### hyphenate 
> 小驼峰转连字符

```js
var hyphenateRE = /\B([A-Z])/g;
var hyphenate = cached(function (str) {
  return str.replace(hyphenateRE,'-$1').toLowerCase()
});
```

### polyfillBind

> 主要是为`bind`做垫片，兼容旧版游览器

```js
function polyfillBind (fn, ctx) {
  function boundFn (a) {
    var l = arguments.length;
    return l
      ? l > 1
      ? fn.apply(ctx, arguments)
      : fn.call(ctx, a)
      : fn.call(ctx)
  }

  boundFn._length = fn.length;
  return boundFn
}

function nativeBind (fn, ctx) {
  return fn.bind(ctx)
}

var bind = Function.prototype.bind
  ? nativeBind
  : polyfillBind;
```

### toArray

> 将类数组转成数组

```js
function toArray (list, start) {
  start = start || 0;
  var i = list.length - start;
  var ret = new Array(i);
  while (i--) {
    ret[i] = list[i + start];
  }
  return ret
}
```

### extend

> 这里并非继承，而是合并，同时会将两个对象地属性，合并到对一个对象上。

```js
function extend (to, _from) {
  for (var key in _from) {
    to[key] = _from[key];
  }
  return to
}
```

### toObject

> 这里是将数组转成对象

```js
function toObject (arr) {
  var res = {};
  for (var i = 0; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i]);
    }
  }
  return res
}
```

### noop

> 空函数

```js
function noop(a, b, c) {}
```

### no

> 一直返回`false`

```js
var no = function (a, b, c) { return false; };
```

### identity

> 返回参数本身

```js
var identity = function (_) { return _; };
```

### genStaticKeys
> 生成静态的key， 应该是在编译部分所用到
```js
function genStaticKeys (modules) {
  return modules.reduce(function (keys, m) {
    return keys.concat(m.staticKeys || [])
  }, []).join(',')
}
```

### looseEqual

> 当两个引用值的引用不同，即使内容相同，但严格来讲是不相同的。
> 这里的函数是宽松相等的作用

```js
function looseEqual (a, b) {
  if (a === b) { return true }
  var isObjectA = isObject(a);
  var isObjectB = isObject(b);
  if (isObjectA && isObjectB) {
    try {
      var isArrayA = Array.isArray(a);
      var isArrayB = Array.isArray(b);
      if (isArrayA && isArrayB) {
        return a.length === b.length && a.every(function (e, i) {
          return looseEqual(e, b[i])
          })
      } else if (a instanceof Date && b instanceof Date) {
        return a.getTime() === b.getTime()
      } else if (!isArrayA && !isArrayB) {
        var keysA = Object.keys(a);
        var keysB = Object.keys(b);
        return keysA.length === keysB.length && keysA.every(function (key) {
          return looseEqual(a[key], b[key])
        })
      } else {
        /* istanbul ignore next */
        return false
      }
    } catch (e) {
      /* istanbul ignore next */
      return false
    }
  } else if (!isObjectA && !isObjectB) {
    return String(a) === String(b)
  } else {
    return false
  }
}
```

### looseIndexOf

> 宽松的`indexOf`, 这里主要借助的是`looseEqual`来进行判断。

```js
function looseIndexOf (arr, val) {
  for (var i = 0; i < arr.length; i++) {
    if (looseEqual(arr[i], val)) { return i }
  }
  return -1
}
```

### once

> 这里也是闭包和高阶函数的一个体现， 让函数只执行一次

```js
function once (fn) {
  var called = false;
  return function () {
    if (!called) {
      called = true;
      fn.apply(this, arguments);
    }
  }
}
```

### 常量以及生命周期

```js
var SSR_ATTR = 'data-server-rendered';

var ASSET_TYPES = [
  'component',
  'directive',
  'filter'
];

var LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured',
  'serverPrefetch'
];
```

## 参考

- [初学者也能看懂的 Vue2 源码中那些实用的基础工具函数](https://juejin.cn/post/7024276020731592741)
- [源码部分](https://github.com/vuejs/vue/blob/dev/dist/vue.js#L14-L379)