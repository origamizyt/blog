---
title: 二十四点计算器
published: 2024-05-26
description: 如何构建计算24点的网页小工具。
tags: [JavaScript, Web]
category: Tutorials
draft: false
---

# 24 = ?

二十四点是一种棋牌类益智游戏。每轮从除去大小王的 52 张牌中抽取四张，通过加减乘除以及括号凑出 24 这个数字。

由于和室友玩经常有算不出来又不知道有没有解的情况，遂决定写一个小工具。

# 核心算法：穷举

:::note
代码全是用 JS 写的，因为做的是前端。
:::

本人不是学计算机的，因此不会什么高级的算法，能想到的只有尝试每一种情况。

算二十四点即递归，先全排列 pop 两个出来，四种运算全部尝试一遍，再 push 回去重新全排列，直到剩下一个数为止。

先写个全排列：
```js
function* permutations(array) {
    if (array.length <= 1) {
        yield array;
        return;
    }
    for (let i = 0; i < array.length; i++) {
        for (const p of permutations([
            ...array.slice(0, i),
            ...array.slice(i+1)
        ])) {
            yield [array[i], ...p];
        }
    }
}
```

再判断是否有解：
```js
function canSolve(array) {
    if (array.length == 0) throw new Error('empty array');
    if (array.length == 1) {
        return array[0] == 24;
    }
    for (const p of permutations(array)) {
        for (const operator of ['+', '-', '*', '/']) {
            let result = eval(`p[0]${operator}p[1]`);
            if (isNaN(result)) 
                continue;
            if (canSolve([result, ...p.slice(2)])) {
                return true;
            }
            result = eval(`p[1]${operator}p[0]`);
            if (isNaN(result)) 
                continue;
            if (canSolve([result, ...p.slice(2)])) {
                return true;
            }
        }
    }
    return false;
}
```

但这样只能判断是否有解，如何给出解？这就需要一个对象来存储之前的操作了。

# 改进：Expr

用 Expr 类来懒计算。构造函数：
```js
class Expr {
    constructor(operands, operator) {
        this.operands = operands;
        this.operator = operator;
    }
}
```

运算符是一个字符串，可以是加减乘除。操作数可以是 number，也可以是 Expr。

定义两个 `toString` 函数，一个全局，一个方法。注意传入 operator 是为了括号的加入。
```js
function toString(operand, operator) {
    if (operand instanceof Expr) {
        return operand.toString(operator);
    }
    return operand.toString();
}

class Expr {
    // ...
    toString(operator) {
        if ((operator === '*' || operator === '/') && (this.operator === '+' || this.operator === '-')) {
            return `(${toString(this.operands[0], this.operator)}${this.operator}${toString(this.operands[1], this.operator)})`;
        }
        return `${toString(this.operands[0], this.operator)}${this.operator}${toString(this.operands[1], this.operator)}`;
    }
}
```

这样我们可以递归得到数学表达式：
```js
console.log(
    new Expr([
        new Expr([9, 3], '+'),
        new Expr([5, 1], '-')
    ], '/')
    .toString()
)
// (9+3)/(5-1)
```

于是可以计算 Expr 的值：
```js
class Expr {
    // ...
    get value() {
        return eval(this.toString(null));
    }
}

function calc(obj) {
    if (obj instanceof Expr) return obj.value;
    return obj
}
```

改一下 `canSolve`：
```js
function solve(array) {
    if (array.length == 0) throw new Error('empty array');
    if (array.length == 1) {
        if (Math.abs(calc(array[0]) - 24) < 1e-6) { // 小心精度！
            return toString(array[0], null);
        }
        return null;
    }
    for (const p of permutations(array)) {
        for (const operator of ['+', '-', '*', '/']) {
            let result;
            if (result = solve([new Expr([p[0], p[1]], operator), ...p.slice(2)])) {
                return result;
            }
            if (result = solve([new Expr([p[1], p[0]], operator), ...p.slice(2)])) {
                return result;
            }
        }
    }
    return null;
}
```

```js
console.log(solve([3, 3, 8, 8]))
// 8/(3-8/3)
```

至此核心逻辑就写完了。

# UI

正好写这个的时候看到了 Vue 的精简版本 PetiteVue：
::github{repo="vuejs/petite-vue"}

佐以 Bootstrap，做成一个简单的小网页：
```html
<div class="d-flex flex-column justify-content-center" style="height: 100vh">
    <div class='d-flex justify-content-center' style="width: 100vw;">
    <div v-scope>
        <div class="text-center">
            <h1 @click="show" style="cursor: pointer">
                {{ result }}
            </h1>
        </div>
        <div class="d-flex gap-3 mt-3">
            <div v-for="i in 4" class="flex-grow-1">
                <input type="number" v-model="inputs[i-1]" min="1" max="13" class="form-control">
            </div>
        </div>
        <div class="d-flex justify-content-center mt-3 gap-3">
            <button type="button" @click="submit" class="btn btn-primary flex-grow-1">计算</button>
            <button type="button" @click="random" class="btn btn-secondary flex-grow-1">随机</button>
        </div>
        <div class="text-center text-muted mt-2" style="font-size: 12px">
            Powered by PetiteVue &amp; Bootstrap
        </div>
    </div>
    </div>
</div>
<script>
    // ...
    PetiteVue.createApp({
        inputs: [1, 1, 1, 1],
        result: '24点计算器',
        hiddenResult: null,
        submit() {
            this.result = '正在计算...'
            this.hiddenResult = null;
            setTimeout(() => {
                const result = solve(this.inputs);
                if (result === null) {
                    this.result = '无解';
                }
                else {
                    this.result = '有解...';
                    this.hiddenResult = '24 = ' + result.replaceAll('*', '×').replaceAll('/', '÷');
                }
            }, 0);
        },
        show() {
            if (this.hiddenResult)
                this.result = this.hiddenResult;
        },
        random() {
            this.result = '24点计算器';
            for (let i = 0; i < 4; i++) {
                this.inputs[i] = Math.ceil(Math.random() * 13);
            }
        }
    }).mount();
</script>
```

:::note
用 `setTimeout` 把耗时逻辑放到事件循环里，要不然 "正在计算..." 显示不出来。
:::

完整的网页在 [24.origamiz.me](https://24.origamiz.me)，有兴趣可以看看。