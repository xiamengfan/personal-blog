---
title: Vue3.x中响应性原理的demo实现
date: 2021-08-26 00:08:39
tags: Vue
---

# 响应性原理说明
这里直接引用[官方文档上的注解](https://v3.cn.vuejs.org/guide/reactivity.html)

1. 当一个值被读取时进行追踪：proxy 的 get 处理函数中 track 函数记录了该 property 和当前副作用(effect)
2. 当某个值改变时进行检测：在 proxy 上调用 set 处理函数
3. 重新运行代码来读取原始值：trigger 函数查找哪些副作用依赖于该 property 并执行它们

# demo实现
``` JavaScript
const stack = [];
const map = new WeakMap(); // 对键名对象弱引用，内存自动释放避免泄漏

function createEffect(fn) {
    stack.push(fn);
    fn();
    stack.pop();
}

function track(target, prop) {
    const effect = stack[stack.length - 1];
    if (!effect) {
        return;
    }
    let propMap = map.get(target);
    if (!propMap) {
        map.set(target, propMap = new Map());
    }
    let effects = propMap.get(prop);
    if (!effects) {
        propMap.set(prop, effects = new Set());
    }
    effects.add(effect);
}

function trigger(target, prop) {
    const propMap = map.get(target);
    if (propMap) {
        const effects = propMap.get(prop);
        if (effects) {
            for (let effect of effects) {
                effect();
            }
        }
    }
}

function reactive(data) {
    return new Proxy(data, {
        get(target, prop, receiver) {
            track(receiver, prop);
            return Reflect.get(...arguments);
        },

        set(target, prop, value, receiver) {
            Reflect.set(...arguments);
            trigger(receiver, prop);
        },
    });
}

function ref(value) {
    return reactive({
        value
    });
}

function computed(fn) {
    const res = ref(null);
    createEffect(() => {
        res.value = fn();
    })
    return res;
}


let a = ref(1);
let b = ref(2);

let c, d;

let e = computed(() => {
    return `a is ${a.value}, b is ${b.value}`;
})

var sum = function () {
    c = a.value + b.value;
}

var product = function () {
    d = a.value * b.value;
}

createEffect(sum);
createEffect(product);

console.log(`${e.value}, c: ${c}, d: ${d}`);

a.value = 3;

console.log(`${e.value}, c: ${c}, d: ${d}`);

b.value = 4;

console.log(`${e.value}, c: ${c}, d: ${d}`);
```