---
nav:
  title: Babel
  order: 7
group:
  title: 介绍
  order: 1
title: Tiny Complier
order: 11
---

# The super tiny complier

这一章主要根据开源项目来实现一个`tiny complier`，旨为简单地阐明流程。

## 目标

在本节，我们仅处理`LISP`到`C`语法的转换, 仅处理下方的`case`。

`LISP`代码

```lisp
(add 2 2)
(subtract 4 2)
(add 2 (subtract 4 2))
```

`C`代码
```C
add(2, 2)
subtract(4, 2)
add(2, subtract(4, 2))
```

这里，我们确定了我们要编译目标，下面我们就来实现一下。

## 编译过程

### `Parse`

这个过程会经过`词法解析`分成`token`流，后面根据`语法分析`来生成`ast`抽象语法树。

### `Transform`

这个步骤是对`ast`来进行增、删、改的操作(依赖于你想要编译的结果)，形成新的`ast`。

### `Generate`

这个步骤是通过修改后的`ast`, 来生成最后的代码。

编译流程大体如上，不同编译器的具体编译流程可能不大一样，不过核心流程是相似的。

## 具体分析实现

### `Parse`

`Parse`的过程是将源码转成`ast`，其中包含两个主要步骤，`词法解析`和`语法解析`。

#### `Lexical Analysis`

这里是词法解析，通过分割源码，进行分词，生成`token`，一般我们把分词的工具称为`tokenizer`。

`Token` 是描述语法的最小单位，可以是`numbers`, `labels`,`operators`等。

```ts
type Token = {
  
};

function tokenizer(sourceCode: string): Array<Token> {
  const token: Array<Token> = [];

  // code

  return token;
}

const tokens = tokenizer(sourceCode);
```

下面我们来实现分词

#### `Syntactic Analysis`

语法解析的过程是将`token`进行组装，根据各部分的描述、语法以及关系去进行组装，生成抽象语法树，是对源代码结果的抽下个。

`Abstract Syntax Tree`(`AST`)是一种嵌套结构去描述源代码。

举个例子

```LISP
(add 2 (subtract 4 2))
```

`Tokens`可能如下:
```ts
[
  { type: 'paren', value: '(' },
  { type: 'name',   value: 'add' },
  { type: 'number', value: '2' },
  { type: 'paren', value: '(' },
  { type: 'name', value: 'subtract' },
  { type: 'name', value: '4' },
  { type: 'name', value: '2' },
  { type: 'paren', value: ')' },
  { type: 'paren', value: ')' },
]
```

生成的`AST`可能如下:
```ts
{
  type: 'Program',
  body: [
    {
      type: 'CallExpression',
      name: 'add',
      params: [
        { 
          type: 'NumberLiteral', 
          value: '2' 
        },
        {
          type: 'CallExpression',
          name: 'subtract',
          params: [
            { 
              type: 'NumberLiteral', 
              value: '4' 
            },
            { 
              type: 'NumberLiteral', 
              value: '2' 
            }
          ] 
        }
      ]
    }
  ]
}
```

##### 实现`tokenizer`

```ts
export function tokenizer(input: string):Token[] {
  
  // 距离遍历代码时候所处的下标。 
  let current = 0;

  // 最终得到的TOKEN数组
  const tokens: Token[] = [];

  // 对输入的代码每个字符都要遍历， 提取去对应的TOKEN, 并推入数组中
  while (current < input.length) {

    let char = input[current];

    if (char === '(') {
      tokens.push({
        type: 'paren',
        value: '('
      });
      current++;
      continue;
    }

    if (char === ')') {
      tokens.push({
        type: 'paren',
        value: ')'
      });
      current++;
      continue;
    }

    let WHITESPACE = /\s/;
    if (WHITESPACE.test(char)) {
      current++;
      continue;
    }

    let NUMBERS = /[0-9]/;
    if (NUMBERS.test(char)) {
      let value = '';

      while(NUMBERS.test(char)) {
        value += char;
        char = input[++current];
      }
      
      tokens.push({
        type: 'number',
        value,
      });
      continue;
    }

    if (char === '"') {
      let value = '';
      
      char = input[++current];

      while (char !== '"') {
        value += char;
        char = input[++current];
      }

      char = input[++current];

      tokens.push({ type: 'string', value });
      continue;
    }

    let LETTERS = /[a-z]/i;
    if (LETTERS.test(char)) {
      let value = '';

      // Again we're just going to loop through all the letters pushing them to
      // a value.
      while (LETTERS.test(char)) {
        value += char;
        char = input[++current];
      }

      // And pushing that value as a token with the type `name` and continuing.
      tokens.push({ type: 'name', value });

      continue;
    }

    // 不能分词的类型 直接抛出异常
    throw new TypeError( 'I dont know what this character is: ' +
    char);
  }
  return tokens;
}
```

`parse`中`token` -> `ast`的实现

```ts
export function parser(tokens: Token[]) {
  let current = 0;

  function walk(): NodeType {
    let token = tokens[current];

    if (token.type === 'number') {
      current++;
      return {
        type: 'NumberLiteral',
        value: token.value,
        _context: [],
      };
    }
    if (token.type === 'string') {
      current++;

      return {
        type: 'StringLiteral',
        value: token.value,
        _context: []
      };
    }

    if (
      token.type === 'paren' &&
      token.value === '('
    ) {
      token = tokens[++current];
      let node: NodeType = {
        type: 'CallExpression',
        name: token.value,
        params: [],
        _context: []
      };

      token = tokens[++current];

      while (
        token.type !== 'paren' ||
        (token.type === 'paren' &&
        token.value !== ')')
      ) {
        node.params && node.params.push(walk()!);
        token = tokens[current];
      }
      current++;

      return node;
    }
    throw new TypeError(token.type);
  }

  let ast: NodeType = {
    type: 'Program',
    body: [],
    _context: []
  };

  while (current < tokens.length) {
    ast.body && ast.body.push(walk()!);
  }

  return ast;
}
```

### `Transform`

`Parse`过后的阶段就是`transform`了，这个过程主要是对得到的`AST`及进行修改。其实可以看到，`AST`是由多个不同种类的节点结合而成的，我们可以去新增、修改、删除节点，以得到新的`AST`。

#### `Traversal`

为了能覆盖所有的节点，`traversal` 通过深度优先来让我们遍历相关的节点。

让我们来看看我们上一部分得到的`AST`。
```ts
{
  type: 'Program',
  body: [
    {
      type: 'CallExpression',
      name: 'add',
      params: [
        { 
          type: 'NumberLiteral', 
          value: '2' 
        },
        {
          type: 'CallExpression',
          name: 'subtract',
          params: [
            { 
              type: 'NumberLiteral', 
              value: '4' 
            },
            { 
              type: 'NumberLiteral', 
              value: '2' 
            }
          ] 
        }
      ]
    }
  ]
}
```
所以我们的执行顺序为

1. `Program` - Starting the Program
2. `CallExpression(add)` - Moving to the first element of the Program's body.
3. `NumberLiteral (2)` -  Moving to the first element of CallExpression's params
4. `CallExpression (subtract)` - Moving to the second element of CallExpression's params
5. `NumberLiteral (4)` - Moving to the first element of CallExpression's(subtract) params
6. `NumberLiteral (2)` - Moving to the second element of CallExpression's(subtract) params

这里会涉及`访问者模式`, 我们仅需要访问或修改要操作的节点。

#### `Visitors`

在这里可以使用`visitor`，提供不同的`node type`, 从而执行对应的函数操作对应类型的节点。(有点类似`babel`的`visitor`)。

```ts
const visitor = {
  NumberLiteral() {},
  CallExpression() {}
}
```

同时，在这里，我们给于函数当前通过`node`以及`node`的父节点。

```ts
const visitor = {
  NumberLiteral(node, parent) {},
  CallExpression(node, parent) {}
};
```

目前，我们只考虑了进入结点时的情况，即`enter`,流程如下。

```txt
- Program
  - CallExpression
    - NumberLiteral
    - CallExpression
      - NumberLiteral
      - NumberLiteral
```


但我们也需要考虑`exit`的情况。

```txt
-> Program (enter)
  -> CallExpression (enter)
    -> Number Literal (enter)
    <- Number Literal (exit)
    -> Call Expression (enter)
      -> Number Literal (enter)
      <- Number Literal (exit)
      -> Number Literal (enter)
      <- Number Literal (exit)
    <- CallExpression (exit)
  <- CallExpression (exit)
<- Program (exit)
```

为了支持这个，我们最终的一个`visitor`的形式为

```ts
const visitor = {
  NumberLiteral: {
    enter(node, parent) {},
    exit(node, parent) {},
  }
};

```

`Transform`的实现
```ts
export function traverser(ast: NodeType, visitor: Visitor) {
 
  function traverseArray(array: NodeType[], parent: NodeType | null) {
    array.forEach(child => {
      traverseNode(child, parent);
    });
  }

  function traverseNode(node: NodeType, parent: NodeType | null) {
    let methods = visitor[node.type];

    if(methods && methods.enter) {
      methods.enter(node, parent);
    }

    switch (node.type) {
      case 'Program':
        node.body && traverseArray(node.body, node);
        break;
      case 'CallExpression':
        node.params && traverseArray(node.params, node);
        break;
      case 'NumberLiteral':
      case 'StringLiteral':
        break;
      default:
        throw new TypeError(node.type);
    }

    if(methods && methods.exit) {
      methods.exit(node, parent);
    }
  }

  traverseNode(ast, null);
}

export function transformer(ast: NodeType) {
  const newAst: NodeType = {
    type: 'Program',
    body: [],
    _context: [],
  };

  ast._context = newAst.body || [];

  traverser(ast, {
    NumberLiteral: {
      enter(node, parent) {
        parent && parent._context.push({
          type: 'NumberLiteral',
          value: node.value,
          _context: [],
        });
      }
    },
    StringLiteral: {
      enter(node, parent) {
        parent && parent._context.push({
          type: 'StringLiteral',
          value: node.value,
          _context: [],
        });
      }
    },
    CallExpression: {
      enter(node, parent) {
        let expression: NodeType = {
          type: 'CallExpression',
          callee: {
            type: 'Identifier',
            value: node.name
          },
          arguments: [],
          _context: [],
        };

        node._context = expression.arguments;
        if (parent) {  
          if (parent.type !== 'CallExpression') {
            expression = {
              type: 'ExpressionStatement',
              expression: expression,
              _context: [],
            };
          }
          parent._context.push(expression);
        }
      }
    }
  });

  return newAst;
}
```


### `Generate`

最后一步就是我们的代码生成了。其实实质上的代码生成，也就是根据我们的`AST`或者`字符串`进行组合生成。

有的编译器会直接根据`token`生成代码，而有的会根据`AST`进行生成，我们这里采用的是`AST`的生成。

实际上，我们的代码生成器将知道如何“打印”所有不同的，`AST` 的节点类型，它将递归地调用自身以打印嵌套节点，直到所有内容都打印成一长串代码(`generate code`)。

```ts
function codeGenerator(node: NodeType): string {
  switch (node.type) {
    case 'Program': 
      return node.body!.map(codeGenerator).join('\n');
    case 'ExpressionStatement':
      return codeGenerator(node.expression as NodeType) + ';';
    case 'CallExpression':
      return (
        codeGenerator(node.callee as NodeType) +
          '(' + 
        (node.arguments as NodeType[]).map(codeGenerator).join(', ') +
         ')'
      );
    case 'Identifier':
      return node.value || '';
    case 'NumberLiteral':
      return node.value || '';
    case 'StringLiteral':
      return `"${node.value}"`;
    default:
      throw new TypeError(node.type);
  }
}
```

## 效果

以下是编译流程代码
```ts
export function compiler(input: string) {
  const tokens = tokenizer(input);
  const ast = parser(tokens);
  const newAst = transformer(ast);
  const output = codeGenerator(newAst);

  return output;
}
```

#### Input
```ts
const output = compiler('(add 2 (subtract 4 2))');
```

#### output
```ts
add(2, subtract(4, 2))
```

## TIP

- [the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler)
- [source code]()