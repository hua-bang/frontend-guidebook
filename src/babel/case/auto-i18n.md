---
nav:
  title: babel
  order: 7
group:
  title: 案例
  order: 2
title: 自动国际化
order: 3
---

# 自动国际化

“自动国际化”是目前一个比较常见的一个需求了，网上已经有很多现成的方案。

下面，我们就用`babel`来进行实现。

## 思路分析

### 需要转化的变量

要转换的实质上是`字符串`以及`模板`，即`StringLiteral`以及`TemplateLiteral`。

##### 字符串

```ts
const a = '中文';
```

替换为
```ts
import intl from 'intl';

const a = intl.t('a');
```

##### 模板字符

```ts
const name = '中文';
const str = `你好 ${name}`
```

替换为
```ts
import intl from 'intl';

const str = intl.t('a', name);
```

`bundle`中的值为:

```js
// zh_CN.js
module.exports = {
  inl1: '中文',
  inl2: '你好 ${placeholder}'
}
```

```js
// en_US.js
module.exports = {
  inl1: 'English',
  inl2: 'hello ${placeholder}'
}
```

实际思路就是在`bundle`中取值，并传入参数替换其中还占位符号。
```ts
intl.t = (key: string, ...args: []) => {
  let index = 0;
  return bundle[local][key].replace(/\{placeholder\}/, () => args[index++]);
}

const locale = 'zh_CN';
const str = intl.t('intl1', name);
```

### 实现流程

1. 引入模块

2. 转换字符串和模板字符串替换成intl.t的形式

3. 把收集的值收集起来，输出到一个资源文件中 

#### 注意条件

1. `jsx`中的插值表达式
2. 有些文字支持注释配置忽略。

## 代码实现

### 引入模块

在这里,我们需要去判断模块是否引入了, 如果没有,则手动引入.

```ts
visitor: {
  Program: {
    enter(path, state) {
      let imported = false;
        path.traverse({
          ImportDeclaration(curPath) {
            const curImportName = curPath.node.source.value;
            if (curImportName === 'intl') {
              imported = true;
              const specifierPath = curPath.get('specifiers.0');
              state.intlUid = specifierPath.get('local').toString();
            }
          }  
        })
        if (!imported) {  
          const uid = path.scope.generateUid('intl');
          const importAst = api.template.ast(`import ${uid} from 'intl'`);
          path.node.body.unshift(importAst);
          state.intlUid = uid;
        }
      }
    },
  }
}
```

同时,在这里,我们也对所有的`/*i18n-disable*/`注释的字符串和模板字符串打标记,后续跳过处理吗, 同时删除这个注释节点.

```ts
path.traverse({
  'TemplateLiteral|StringLiteral'(path) {
    if (path.node.leadingComments) {
      path.node.leadingComments = path.node.leadingComments.filter((comment, index) => {
        if (comment.value.includes('i18n-disable')) {
          path.node.skipTransform = true;
          return false;
        }
        return true;
      });
    }
    if(path.findParent(p => p.isImportDeclaration())) {
      path.node.skipTransform = true;
    }
  }
});
```

### 字符串 模版处理
之后处理 StringLiteral 和 TemplateLiteral 节点，用 state.intlUid + '.t' 的函数调用语句来替换原节点。

> 注意：替换完以后要用 path.skip 跳过新生成节点的处理，不然就会进入无限循环


```ts
StringLiteral(path, state) {
    if (path.node.skipTransform) {
        return;
    }
    let key = nextIntlKey();
    save(state.file, key, path.node.value);

    const replaceExpression = getReplaceExpression(path, key, state.intlUid);
    path.replaceWith(replaceExpression);
    path.skip();
},
TemplateLiteral(path, state) {
   if (path.node.skipTransform) {
        return;
   }
    const value = path.get('quasis').map(item => item.node.value.raw).join('{placeholder}');
    if(value) {
        let key = nextIntlKey();
        save(state.file, key, value);

        const replaceExpression = getReplaceExpression(path, key, state.intlUid);
        path.replaceWith(replaceExpression);
        path.skip();
    }
},
```

`getReplaceExpression`
```ts
function getReplaceExpression(path, value, intlUid) {
    const expressionParams = path.isTemplateLiteral() ? path.node.expressions.map(item => generate(item).code) : null
    let replaceExpression = api.template.ast(`${intlUid}.t('${value}'${expressionParams ? ',' + expressionParams.join(',') : ''})`).expression;
    if (path.findParent(p => p.isJSXAttribute()) && !path.findParent(p=> p.isJSXExpressionContainer())) {
        replaceExpression = api.types.JSXExpressionContainer(replaceExpression);
    }
    return replaceExpression;
}
```

### 生成文件

`intl` 的 `key` 也需要生成唯一的。

```ts
let intlIndex = 0;
function nextIntlKey() {
    ++intlIndex;
    return `intl${intlIndex}`;
}
```

`save`方法则是收集替换`key`和`value`,保存到`file`.
```ts
function save(file, key, value) {
  const allText = file.get('allText');
  allText.push({
    key, value
  });
  file.set('allText', allText);
}
```

这个是在 pre 初始化的，并且在 post 阶段取出来用于生成 resource 文件，生成位置也是通过插件的 outputDir 参数传入的。

```ts
pre(file) {
    file.set('allText', []);
},
post(file) {
    const allText = file.get('allText');
    const intlData = allText.reduce((obj, item) => {
        obj[item.key] = item.value;
        return obj;
    }, {});

    const content = `const resource = ${JSON.stringify(intlData, null, 4)};\nexport default resource;`;
    fse.ensureDirSync(options.outputDir);
    fse.writeFileSync(path.join(options.outputDir, 'zh_CN.js'), content);
    fse.writeFileSync(path.join(options.outputDir, 'en_US.js'), content);
}
```

## 效果

Input:

```ts
import intl from 'intl';
/**
 * App
 */
function App() {
  const title = 'title';
  const desc = `desc`;
  const desc2 = /*i18n-disable*/`desc`;
  const desc3 = `aaa ${ title + desc} bbb ${ desc2 } ccc`;

  return (
    <div className="app" title={"测试"}>
      <img src={Logo} />
      <h1>${title}</h1>
      <p>${desc}</p>  
      <div>
      {
        /*i18n-disable*/'中文'
      }
      </div>
    </div>
  );
};
```

OutPut:

```ts
import intl from 'intl';
/**
 * App
 */

function App() {
  const title = intl.t('intl1');
  const desc = intl.t('intl2');
  const desc2 = `desc`;
  const desc3 = intl.t('intl3', title + desc, desc2);
  return <div className={intl.t('intl4')} title={intl.t('intl5')}>
        <img src={Logo} />
        <h1>${title}</h1>
        <p>${desc}</p>
        <div>
        {'中文'}
        </div>
      </div>;
}
```

## 总结

其实我们可以更加完善，这个案例只是提供思路.

## TIP
- [Babel 插件通关秘籍](https://juejin.cn/book/6946117847848321055/section/6951617082454704162)
- [source code](https://github.com/QuarkGluonPlasma/babel-plugin-exercize/blob/master/exercize-auto-i18n/src/plugin/auto-i18n-plugin.js)
