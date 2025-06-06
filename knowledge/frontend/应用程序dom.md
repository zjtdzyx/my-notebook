# 应用程序dom

## 1.使用react实现一个计数器

以下是使用react和tailwind去实现的计数器

思路非常简单：需求就是实现计数的效果

数据为count  逻辑为increment decrement reset 然后完成数据、逻辑与UI的绑定即可

```tailwind
tailwind的基础使用：
1.layout 布局
flex flex-col默认横轴为主轴,col设主轴为y轴 items-center justify-center x/y轴的居中 space-x/y-<size> flex元素的距离设置
2.font
font-<family> font-<weight> text-color-<shade> text-<size>(xl) text-center 文字的类别 质量 颜色深度 大小 文字居中
3.box
m p trbl x/y 外边距与内边距
4.animation
hover:bg-color-<shade>  鼠标移动，背景颜色深度
focus:outline-none 移除按钮原本样式
focus:ring-<size>/color-<shade>/offset-<size>颜色深度与内偏移量
transiton 动画过渡，结合hover/focus实现平滑而不是突然变化
ease-in-out 中间变化快 两边变化慢
duration-<ms> 动画持续时间
```

```react
import React, { useState } from 'react';

function App() { // 修复：函数声明需要使用圆括号 ()
  // 设置count状态
  const [count, setCount] = useState(0); // 修复：移除了类型注释 <number,number>

  // 使用函数式写法，获取到最新的state
  const incrementCount = () => { // 修复：箭头函数语法
    setCount(prevCount => prevCount + 1); // 修复：使用 prevCount + 1 而不是 prevCount++
  };

  // 考虑特殊情况，计数器不能小于0
  const decrementCount = () => { // 修复：箭头函数语法
    setCount(prevCount => Math.max(0, prevCount - 1)); // 修复：使用 prevCount - 1 而不是 prevCount--
  };

  const resetCount = () => { // 修复：箭头函数语法
    setCount(0); // 修复：直接设置0，而不是赋值操作
  };

  return (
   <main className="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4 font-inter"> {/* 修复：class 改为 className，并添加基础样式 */}
     <h1 className="text-3xl font-bold text-gray-800 mb-6">React 计数器</h1> {/* 添加标题和样式 */}
     <p className="text-6xl font-extrabold text-blue-600 mb-8"> {/* 修复：移除了多余的 {{ }}，直接使用 {count} */}
      {count}
     </p>

    <div className="flex space-x-4"> {/* 按钮布局 */}
      <button
        onClick={incrementCount}
        className="px-6 py-3 bg-green-500 text-white font-semibold rounded-lg shadow-md hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2 transition ease-in-out duration-150"
      >
        增加
      </button>
      <button
        onClick={decrementCount}
        className="px-6 py-3 bg-red-500 text-white font-semibold rounded-lg shadow-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 transition ease-in-out duration-150"
      >
        减少
      </button>
      <button
        onClick={resetCount}
        className="px-6 py-3 bg-gray-500 text-white font-semibold rounded-lg shadow-md hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2 transition ease-in-out duration-150"
      >
        重置
      </button>
    </div>
   </main>
  );
}

export default App;

```

## 2.使用react实现一个todolist