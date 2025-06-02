# JS编码专题

## 1.call apply bind

```js
// 定义一个用于演示的函数
function greet(age, city) {
  console.log(`你好，我是 ${this.name}，今年 ${age} 岁，来自 ${city}。`);
}

// 定义一个用于演示的对象
const person = {
  name: '小明'
};

console.log('--- call 的原生用法 ---');
greet.call(person, 20, '北京'); // 输出：你好，我是 小明，今年 20 岁，来自 北京。

console.log('\n--- apply 的原生用法 ---');
greet.apply(person, [25, '广州']); // 输出：你好，我是 小明，今年 25 岁，来自 广州。

console.log('\n--- bind 的原生用法 ---');
const boundGreet = greet.bind(person, 30);
boundGreet('杭州'); // 输出：你好，我是 小明，今年 30 岁，来自 杭州。

const anotherBoundGreet = greet.bind(person);
anotherBoundGreet(32, '南京'); // 输出：你好，我是 小明，今年 32 岁，来自 南京。


// --- 实现自己的 call ---
// 给 Function 的原型上添加一个 myCall 方法
Function.prototype.myCall = function(context, ...args) {
  // 1. 如果没有传入 context，或者 context 是 null/undefined，就默认指向全局对象（浏览器是 window，Node.js 是 global）
  // 这样做是为了模拟原生 call 在传入 null/undefined 时 this 指向全局的行为
  context = context || window;

  // 2. 将当前函数（this，也就是调用 myCall 的函数，例如这里的 greet）作为 context 对象的一个临时属性
  // 这样，当 context.fn() 执行时，函数内部的 this 就会指向 context
  context.fn = this;

  // 3. 执行这个临时函数，并传入剩余的参数
  const result = context.fn(...args);

  // 4. 删除 context 上新增的临时属性，避免污染原对象
  delete context.fn;

  // 5. 返回函数执行的结果
  return result;
};


console.log('\n--- 实现自己的 myCall ---');
greet.myCall(person, 22, '上海'); // 输出：你好，我是 小明，今年 22 岁，来自 上海。
greet.myCall(null, 18, '成都'); // 输出：你好，我是 undefined，今年 18 岁，来自 成都。（如果全局没有定义 name）


// --- 实现自己的 apply ---
// 给 Function 的原型上添加一个 myApply 方法
Function.prototype.myApply = function(context, args = []) {
  // 1. 处理 context，和 myCall 类似
  context = context || window;

  // 2. 将当前函数作为 context 对象的一个临时属性
  context.fn = this;

  let result;
  // 3. 检查 args 是否是数组，并执行函数
  // 如果 args 存在且是数组，则使用展开运算符传入参数
  if (args && Array.isArray(args)) {
    result = context.fn(...args);
  } else {
    // 如果 args 不存在或不是数组，则直接执行函数（不传参数）
    result = context.fn();
  }

  // 4. 删除临时属性
  delete context.fn;

  // 5. 返回结果
  return result;
};

console.log('\n--- 实现自己的 myApply ---');
greet.myApply(person, [28, '深圳']); // 输出：你好，我是 小明，今年 28 岁，来自 深圳。
greet.myApply(null, [19, '重庆']); // 输出：你好，我是 undefined，今年 19 岁，来自 重庆。


// --- 实现自己的 bind ---
// 给 Function 的原型上添加一个 myBind 方法
Function.prototype.myBind = function(context, ...args) {
  // 1. 保存当前函数（也就是调用 myBind 的函数，例如这里的 greet）
  const self = this;

  // 2. 返回一个新的函数
  return function(...newArgs) {
    // 3. 在新函数被调用时，使用 apply 来执行原始函数
    // 确保 this 指向 context，并且将 myBind 接收的参数和新函数接收的参数合并后传入
    return self.apply(context, [...args, ...newArgs]);
  };
};

console.log('\n--- 实现自己的 myBind ---');
const myBoundGreet = greet.myBind(person, 35);
myBoundGreet('成都'); // 输出：你好，我是 小明，今年 35 岁，来自 成都。

const myAnotherBoundGreet = greet.myBind(person, 38, '武汉');
myAnotherBoundGreet(); // 输出：你好，我是 小明，今年 38 岁，来自 武汉。
```

## 2.防抖节流

```js
// --- 防抖（Debounce）的实现 ---
function debounce(func, delay) {
  let timeoutId; // 用于存储计时器的 ID

  // 返回一个新的函数，这个新函数才是真正会被调用的
  return function(...args) {
    // 每次这个新函数被调用时，都先清除上一次的计时器
    clearTimeout(timeoutId);

    // 重新设置一个计时器
    // 计时器到期后，才执行传入的 func 函数
    // func.apply(this, args) 确保 func 在正确的 this 上下文和参数下执行
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

console.log('\n--- 防抖（Debounce）示例 ---');
function handleSearch(keyword) {
  console.log(`正在搜索：${keyword}`);
}

const debouncedSearch = debounce(handleSearch, 500); // 设置 500 毫秒的防抖延迟

// 模拟用户快速输入
debouncedSearch('a');
debouncedSearch('ab');
debouncedSearch('abc');
// 500ms 后，只会打印一次 "正在搜索：abc"

// 模拟用户停止输入一段时间后再次输入
setTimeout(() => {
  debouncedSearch('apple');
  debouncedSearch('apples');
}, 1000); // 1 秒后，又会触发一次防抖，最终打印 "正在搜索：apples"


// --- 节流（Throttle）的实现 ---
function throttle(func, delay) {
  let lastTime = 0; // 记录上次函数执行的时间

  // 返回一个新的函数
  return function(...args) {
    const now = Date.now(); // 获取当前时间

    // 如果当前时间距离上次执行时间超过了延迟（delay）
    if (now - lastTime >= delay) {
      // 立即执行函数
      func.apply(this, args);
      // 更新上次执行时间为当前时间
      lastTime = now;
    }
    // 如果没有超过延迟，则不执行，等待下一次机会
  };
}

console.log('\n--- 节流（Throttle）示例 ---');
function handleScroll() {
  console.log('页面滚动了！');
}

const throttledScroll = throttle(handleScroll, 1000); // 设置 1000 毫秒的节流间隔

// 模拟滚动事件，每 200 毫秒触发一次
let scrollCount = 0;
const scrollInterval = setInterval(() => {
  if (scrollCount < 10) { // 模拟触发 10 次
    throttledScroll();
    scrollCount++;
  } else {
    clearInterval(scrollInterval);
    console.log('模拟滚动结束。');
  }
}, 200);
// 预期：每 1000 毫秒（1 秒）打印一次 "页面滚动了！"
```

## 3.promise

```js
// --- Promise 的简易实现 ---
class MyPromise {
  constructor(executor) {
    // Promise 的初始状态是 pending（等待中）
    this.state = 'pending';
    // 成功时存储的值
    this.value = undefined;
    // 失败时存储的原因
    this.reason = undefined;
    // 存储成功回调函数，因为可能在 Promise 状态变为 fulfilled 之前就调用了 .then()
    this.onFulfilledCallbacks = [];
    // 存储失败回调函数，因为可能在 Promise 状态变为 rejected 之前就调用了 .catch()
    this.onRejectedCallbacks = [];

    // resolve 函数：用于将 Promise 状态从 pending 变为 fulfilled
    const resolve = (value) => {
      // 只有当状态是 pending 时才能改变
      if (this.state === 'pending') {
        this.state = 'fulfilled'; // 改变状态
        this.value = value; // 存储成功的值
        // 遍历并执行所有已注册的成功回调
        this.onFulfilledCallbacks.forEach(fn => fn(this.value));
      }
    };

    // reject 函数：用于将 Promise 状态从 pending 变为 rejected
    const reject = (reason) => {
      // 只有当状态是 pending 时才能改变
      if (this.state === 'pending') {
        this.state = 'rejected'; // 改变状态
        this.reason = reason; // 存储失败的原因
        // 遍历并执行所有已注册的失败回调
        this.onRejectedCallbacks.forEach(fn => fn(this.reason));
      }
    };

    // 立即执行传入的 executor 函数
    // executor 函数会接收 resolve 和 reject 作为参数
    try {
      executor(resolve, reject);
    } catch (error) {
      // 如果 executor 执行过程中抛出错误，则直接 reject Promise
      reject(error);
    }
  }

  // .then() 方法：用于注册成功回调
  then(onFulfilled) {
    // 如果当前 Promise 已经是 fulfilled 状态，立即执行回调
    if (this.state === 'fulfilled') {
      onFulfilled(this.value);
    } else if (this.state === 'pending') {
      // 如果是 pending 状态，将回调添加到成功回调队列中
      this.onFulfilledCallbacks.push(onFulfilled);
    }
    // 为了支持链式调用，返回 this
    return this;
  }

  // .catch() 方法：用于注册失败回调（实际上是 .then(null, onRejected) 的语法糖）
  catch(onRejected) {
    // 如果当前 Promise 已经是 rejected 状态，立即执行回调
    if (this.state === 'rejected') {
      onRejected(this.reason);
    } else if (this.state === 'pending') {
      // 如果是 pending 状态，将回调添加到失败回调队列中
      this.onRejectedCallbacks.push(onRejected);
    }
    // 为了支持链式调用，返回 this
    return this;
  }
}

console.log('\n--- Promise 原生用法示例 ---');
// 模拟一个异步操作，1 秒后成功或失败
function simulateAsyncOperation(success) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (success) {
        resolve('原生 Promise 操作成功！');
      } else {
        reject('原生 Promise 操作失败！');
      }
    }, 1000);
  });
}

simulateAsyncOperation(true)
  .then(result => {
    console.log('原生 Promise 成功:', result); // 1 秒后输出
  })
  .catch(error => {
    console.log('原生 Promise 失败:', error);
  });

simulateAsyncOperation(false)
  .then(result => {
    console.log('原生 Promise 成功:', result);
  })
  .catch(error => {
    console.log('原生 Promise 失败:', error); // 1 秒后输出
  });


console.log('\n--- MyPromise 简易实现示例 ---');
// 使用我们自己实现的 MyPromise 模拟异步操作
function mySimulateAsync(success) {
  return new MyPromise((resolve, reject) => {
    setTimeout(() => {
      if (success) {
        resolve('MyPromise 成功！');
      } else {
        reject('MyPromise 失败！');
      }
    }, 1500); // 设置 1.5 秒延迟
  });
}

mySimulateAsync(true)
  .then(res => console.log('MyPromise 成功:', res))
  .catch(err => console.log('MyPromise 失败:', err));

mySimulateAsync(false)
  .then(res => console.log('MyPromise 成功:', res))
  .catch(err => console.log('MyPromise 失败:', err));

// 模拟 Promise 立即 resolve 的情况
new MyPromise((resolve, reject) => {
  resolve('MyPromise 立即成功！');
}).then(res => console.log('MyPromise 立即成功:', res));

// 模拟 Promise 立即 reject 的情况
new MyPromise((resolve, reject) => {
  reject('MyPromise 立即失败！');
}).catch(err => console.log('MyPromise 立即失败:', err));


```

## 4.基础算法题

```
1.两数之和-给定⼀个数组nums和⼀个⽬标值target，在该数组中找出和为⽬标值的两个数
2.三数之和-给定⼀个数组nums，判断nums中是否存在三个元素a，b，c，使得a+b+c=target，找出所有满⾜条件且不重复的三元组合
3.输⼊⼀个字符串，找到第⼀个不重复字符的下标
4.输⼊⼀个字符串，打印出该字符串中，所有字符的排列组合
5.冒泡排序
6.选择排序
7.快速排序
8.插⼊排序
9.列表转成树
10.深度优先遍历-对树进⾏遍历，从第⼀个节点开始，遍历其⼦节点，直到它的所有⼦节点都被遍历完毕，然后再遍历它的兄弟节点
11. ⼴度优先遍历-以横向的维度对树进⾏遍历，从第⼀个节点开始，依次遍历其所有的兄弟节点，再遍历第⼀个节点的⼦节点，⼀层层向下遍历
12. 查找树形结构中符合要求的节点
13. ⼆叉查找树-判断⼀个数组，是否为某⼆叉查找树的前序遍历结果，⼆叉查找树特点是所有的左节点⽐⽗节点的值⼩，所有的右节点⽐⽗节点的值⼤
14.买卖股票问题-给定⼀个整数数组，其中第i个元素代表了第i天的股票价格
15.斐波那契数列-从第3项开始，当前项等于前两项之和：1123581321
……，计算第n项的值
16.滑动窗⼝最⼤值-给定⼀个数组nums，有⼀个⼤⼩为k的滑动窗⼝，从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗⼝中的k个数字。滑动窗⼝每
次只向右移动⼀位，求返回滑动窗⼝最⼤值
17.最⻓递增⼦序列-⼀个整数数组nums，找到其中⼀组最⻓递增⼦序列的值
```

