# React教程

## 前言

学习一门框架的成本是昂贵的，不同人在学习框架的事情，选择的方式不同。有些人喜欢通过视频讲解的方式去学习，有些人喜欢去阅读官方文档去学习，有些人喜欢直接实现一个简单的demo应用程序去学习。但是对于初学者而言，阅读官方文档是非常吃力的，视频讲解是非常耗时的，我个人比较喜欢做一个todolist来完成框架的入门学习。这篇教程的教学方式是通过理解重要概念+demo实践的方式去完成框架的学习。在重要概念的讲解中，我会插入一些案例。因为目前hook式编程已经成为react的未来范式，所以全部的讲解都会尽可能的使用hook式的编程范式去讲解！哦对，本教程的前置条件是，已经完成了react的基础语法的学习(知道如何去用JSX，变量如何声明，如何去绑定数据和逻辑到UI中)

## 一、react的基础hooks

咱们知道，React Hook 是 React 16.8 以后引入的，它让函数组件也能拥有状态（state）和其他 React 特性，彻底改变了我们写组件的方式，让代码更简洁、可读性更强。

#### 1. `useState`：给函数组件一个“记忆”

`useState` 绝对是你最常用的 Hook，它能让你的函数组件拥有自己的状态。想象一下，你写一个按钮，点击一下，按钮上的数字就变大，这个变化的数字就是组件的状态。

**最优范式解读：**

- **声明和初始化：** `useState` 返回一个数组，第一个是当前状态值，第二个是更新状态的函数。

  JavaScript

  ```
  import React, { useState } from 'react';
  
  function Counter() {
    // 声明一个名为 'count' 的状态变量，初始值为 0
    const [count, setCount] = useState(0);
  
    const increment = () => {
      // 使用更新函数来改变状态
      setCount(count + 1); // 这种方式在多次更新时可能会有问题
      // 更推荐的更新方式：使用函数式更新，确保拿到最新的state
      // setCount(prevCount => prevCount + 1);
    };
  
    return (
      <div>
        <p>你点击了 {count} 次</p>
        <button onClick={increment}>点击我</button>
      </div>
    );
  }
  ```

- **函数式更新：** 当你的新状态依赖于上一个状态时，强烈建议你传入一个函数给 `setCount`。这样可以避免闭包陷阱，确保拿到最新的状态值，尤其是在批量更新或者异步更新的场景下。

  JavaScript

  ```
  // 更好的做法：使用函数式更新
  setCount(prevCount => prevCount + 1);
  ```

- **状态的不可变性：** 记住，永远不要直接修改状态！`useState` 维护的是状态的副本，你需要通过 `set` 函数来更新它。比如，如果你有一个对象状态，想要更新其中的某个属性，你需要创建一个新的对象，然后传入新的对象。

  JavaScript

  ```
  const [user, setUser] = useState({ name: '张三', age: 20 });
  
  const updateAge = () => {
    // 错误示范：直接修改
    // user.age = 21;
    // setUser(user);
  
    // 正确做法：创建新对象并更新
    setUser(prevUser => ({ ...prevUser, age: prevUser.age + 1 }));
  };
  ```

#### 2. `useEffect`：处理副作用的“管家”

`useEffect` 让你在函数组件中执行副作用操作，比如数据获取、订阅事件、手动改变 DOM 等。可以把它想象成 Class 组件中的 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 的结合体。

**最优范式解读：**

- **清理函数：** 如果你的副作用需要清理（比如取消订阅、清除定时器），`useEffect` 允许你返回一个函数。这个返回的函数会在组件卸载时或者下次副作用执行前运行。

​    import React, { useState, useEffect } from 'react';



````
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000);

    // 返回一个清理函数
    return () => {
      clearInterval(intervalId); // 组件卸载时或effect重新执行前清除定时器
    };
  }, []); // 空依赖数组表示只在组件挂载时执行一次（相当于 componentDidMount）

  return <div>计时器: {count}</div>;
}
```
````

- **依赖数组：** `useEffect` 的第二个参数是依赖数组。

  - **空数组 `[]`：** 表示只在组件挂载时执行一次，相当于 `componentDidMount`。这是处理一次性数据获取的常见方式。
  - **省略依赖数组：** 每次组件渲染都会执行（非常不推荐，容易造成无限循环或性能问题）。
  - **包含依赖项：** 只有当依赖项发生变化时，`useEffect` 才会重新执行。这是优化性能的关键。

  JavaScript

  ```
  // 当 userId 变化时才重新获取用户数据
  useEffect(() => {
    // fetchData(userId);
  }, [userId]);
  ```

  **陷阱与规避：** 如果你发现 `useEffect` 在无限循环或者行为异常，通常是因为依赖数组没有正确设置，或者在 `useEffect` 内部创建了会引起循环的函数或对象。

- **分离关注点：** 如果你有多个不相关的副作用，可以声明多个 `useEffect` Hook。这比在一个 `useEffect` 里处理所有事情更清晰。

#### 3. `useContext`：解决组件层级传递问题的“直通车”

`useContext` 可以让你订阅 React Context，避免了逐层传递 `props` 的“`prop drilling`”问题。它提供了一种在组件树中共享数据的方式，比如主题、用户认证信息等。

**最优范式解读：**

- **创建 Context：** 首先你需要创建一个 Context。

  JavaScript

  ```
  // theme-context.js
  import React from 'react';
  export const ThemeContext = React.createContext('light'); // 默认值
  ```

- **提供 Context：** 在父组件中使用 `Context.Provider` 包裹需要访问这个 Context 的子组件，并通过 `value` prop 提供数据。

  JavaScript

  ```
  // App.js
  import React from 'react';
  import { ThemeContext } from './theme-context';
  import Toolbar from './Toolbar';
  
  function App() {
    return (
      <ThemeContext.Provider value="dark"> {/* 提供 'dark' 主题 */}
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
  ```

- **消费 Context：** 在子组件中使用 `useContext` Hook 来获取 Context 的值。

  JavaScript

  ```
  // Toolbar.js
  import React, { useContext } from 'react';
  import ThemedButton from './ThemedButton';
  import { ThemeContext } from './theme-context'; // 引入Context
  
  function Toolbar() {
    const theme = useContext(ThemeContext); // 获取Context的值
    return (
      <div style={{ background: theme === 'dark' ? '#333' : '#eee', color: theme === 'dark' ? '#fff' : '#000' }}>
        <p>当前主题: {theme}</p>
        <ThemedButton />
      </div>
    );
  }
  ```

#### 4. `useRef`：获取 DOM 元素或在渲染间共享可变值

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内保持不变。它有两个主要用途：

- **访问 DOM 元素：** 这是最常见的用途，可以让你直接操作 DOM（比如获取 input 焦点）。
- **存储可变值：** 在多次渲染之间共享可变值，而不会引起组件重新渲染（不像 `useState`）。

**最优范式解读：**

- **访问 DOM：**

  JavaScript

  ```
  import React, { useRef, useEffect } from 'react';
  
  function MyInput() {
    const inputRef = useRef(null); // 创建一个 ref
  
    useEffect(() => {
      // 组件挂载后，ref.current 指向真实的 DOM 节点
      if (inputRef.current) {
        inputRef.current.focus(); // 让 input 自动获取焦点
      }
    }, []); // 只在组件挂载时执行一次
  
    return <input ref={inputRef} type="text" />; // 将 ref 绑定到 DOM 元素
  }
  ```

- **存储可变值：** 比如，你想在组件渲染时不触发重新渲染的情况下存储一个 ID。

  JavaScript

  ```
  import React, { useRef, useState } from 'react';
  
  function CountRender() {
    const [count, setCount] = useState(0);
    const renderCount = useRef(0); // 创建一个 ref 来记录渲染次数
  
    // renderCount.current 的变化不会引起组件重新渲染
    renderCount.current = renderCount.current + 1;
  
    return (
      <div>
        <p>点击次数: {count}</p>
        <p>组件渲染次数: {renderCount.current}</p>
        <button onClick={() => setCount(count + 1)}>增加点击</button>
      </div>
    );
  }
  ```

#### 5. `useCallback` 和 `useMemo`：性能优化的“利器”

这两个 Hook 都是用来优化性能的，避免不必要的重新计算和渲染，但它们优化的对象不同。

**最优范式解读：**

- useCallback：记住函数

  useCallback 返回一个 memoized（缓存的）回调函数。只有当它的依赖项发生变化时，它才会返回一个新的函数。这在将回调函数作为 props 传递给子组件时非常有用，可以防止子组件不必要的重新渲染（尤其是当子组件使用了 React.memo 时）。

  JavaScript

  ```
  import React, { useState, useCallback, memo } from 'react';
  
  // 子组件，使用了 memo 优化，只有当 props 改变时才重新渲染
  const MyButton = memo(({ onClick, text }) => {
    console.log('MyButton rendered');
    return <button onClick={onClick}>{text}</button>;
  });
  
  function ParentComponent() {
    const [count, setCount] = useState(0);
    const [name, setName] = useState('张三');
  
    // 只有当 count 变化时，这个函数才会重新创建
    const handleClick = useCallback(() => {
      setCount(prevCount => prevCount + 1);
    }, [count]); // 依赖 count
  
    // 这个函数每次渲染都会重新创建
    const handleOtherClick = () => {
      console.log('Other click');
    };
  
    return (
      <div>
        <p>Count: {count}</p>
        <input type="text" value={name} onChange={(e) => setName(e.target.value)} />
        {/* 当 ParentComponent 重新渲染时，如果 name 改变了，handleClick 不会重新创建 */}
        <MyButton onClick={handleClick} text="点击我" />
        {/* 每次 ParentComponent 重新渲染，handleOtherClick 都会重新创建，可能导致 MyButton 重新渲染 */}
        {/* <MyButton onClick={handleOtherClick} text="另一个按钮" /> */}
      </div>
    );
  }
  ```

  **使用场景：** 当你的函数作为 props 传递给经过 `React.memo` 优化的子组件时。

- useMemo：记住值

  useMemo 返回一个 memoized（缓存的）值。只有当它的依赖项发生变化时，它才会重新计算这个值。这对于执行昂贵计算的函数非常有用，可以避免在每次渲染时都重新计算。

  JavaScript

  ```
  import React, { useState, useMemo } from 'react';
  
  function ExpensiveCalculationComponent({ num }) {
    // 模拟一个耗时的计算
    const expensiveValue = useMemo(() => {
      console.log('Performing expensive calculation...');
      let result = 0;
      for (let i = 0; i < num * 100000000; i++) { // 模拟大量计算
        result += i;
      }
      return result;
    }, [num]); // 只有当 num 变化时才重新计算
  
    return (
      <div>
        <p>传入的数字: {num}</p>
        <p>昂贵计算结果: {expensiveValue}</p>
      </div>
    );
  }
  
  function App() {
    const [count, setCount] = useState(0);
    const [inputNum, setInputNum] = useState(1);
  
    return (
      <div>
        <button onClick={() => setCount(count + 1)}>增加计数 ({count})</button>
        <input
          type="number"
          value={inputNum}
          onChange={(e) => setInputNum(parseInt(e.target.value))}
        />
        {/* 只有当 inputNum 变化时，ExpensiveCalculationComponent 内部的昂贵计算才会重新执行 */}
        <ExpensiveCalculationComponent num={inputNum} />
      </div>
    );
  }
  ```

  **使用场景：** 当你的组件需要渲染一个计算成本很高的数据时，或者当一个值作为 props 传递给经过 `React.memo` 优化的子组件时。

关于 useCallback 和 useMemo 的忠告：

不要过度优化！只有当你真正遇到了性能瓶颈时才考虑使用它们。过多的 useCallback 和 useMemo 反而会增加代码的复杂性，并且它们本身也有一定的开销。通常，先用好 useEffect 的依赖数组，再考虑这两个。

#### 总结与感悟

学习 React Hook 就像是学习一套新的武功秘籍，掌握了它们，你就能更优雅、更高效地编写 React 组件。

- **`useState`** 是状态管理的基石。
- **`useEffect`** 是处理各种副作用的瑞士军刀，但要警惕依赖数组的坑。
- **`useContext`** 是跨组件传递数据的利器。
- **`useRef`** 让你能直接操作 DOM 或存储可变值。
- **`useCallback` 和 `useMemo`** 是性能优化的杀手锏，但要慎用。

在面试中，除了知道它们怎么用，更重要的是能讲清楚为什么要用它们，以及它们在不同场景下的最优实践。多思考一下这些 Hook 背后的原理，它们帮你解决了什么问题，以及在什么情况下使用它们能让你的代码更健壮、更高效。

## 二、react应用程序的常见性能优化策略

React 的性能优化，核心思想就是：**减少不必要的渲染，减少计算量。**

#### 1. 组件渲染优化：只在必要时更新

这是 React 性能优化的重中之重，也是最直接能看到效果的。

- **`React.memo`（针对函数组件）：**

  - **作用：** 如果你的函数组件在给定相同的 `props` 的情况下渲染结果相同，你可以使用 `React.memo` 包裹它。这样，React 将跳过渲染该组件的操作，并复用最近一次渲染的结果。

  - 最优范式：

    JavaScript

    ```
    import React, { memo } from 'react';
    
    // 只有当 props.name 或 props.age 变化时，UserDisplay 才会重新渲染
    const UserDisplay = memo(({ name, age }) => {
      console.log('UserDisplay rendered');
      return (
        <div>
          <p>姓名: {name}</p>
          <p>年龄: {age}</p>
        </div>
      );
    });
    
    function ParentComponent() {
      const [count, setCount] = useState(0);
      const user = { name: '张三', age: 25 }; // 假设这里user对象每次渲染都会重新创建
    
      return (
        <div>
          <button onClick={() => setCount(count + 1)}>Count: {count}</button>
          {/* 即使 ParentComponent 重新渲染，如果 user.name 和 user.age 没变，UserDisplay 也不会渲染 */}
          <UserDisplay name={user.name} age={user.age} />
        </div>
      );
    }
    ```

  - **注意：** 如果 `props` 中包含函数或对象，每次父组件渲染时，这些函数或对象都会是新的引用，即使其内容未变，也会导致 `memo` 失效。这时就需要结合 `useCallback` 或 `useMemo`。

- **`PureComponent`（针对 Class 组件）：**

  - **作用：** 与 `React.memo` 类似，`PureComponent` 会对 `props` 和 `state` 进行浅比较。如果 `props` 或 `state` 没有发生变化，则组件不会重新渲染。

  - 最优范式：

    JavaScript

    ```
    import React, { PureComponent } from 'react';
    
    class MyPureComponent extends PureComponent {
      render() {
        console.log('MyPureComponent rendered');
        return <div>{this.props.value}</div>;
      }
    }
    ```

  - **局限性：** 同样，如果 `props` 或 `state` 包含引用类型（对象、数组、函数），浅比较可能不符合预期，导致不必要的渲染或该渲染却不渲染的问题。

- **`shouldComponentUpdate`（针对 Class 组件，高级用法）：**

  - **作用：** 这是一个生命周期方法，你可以手动控制 Class 组件何时重新渲染。它接收 `nextProps` 和 `nextState` 作为参数，如果你返回 `false`，组件将不会重新渲染。

  - 最优范式：

    JavaScript

    ```
    class MyCustomComponent extends React.Component {
      shouldComponentUpdate(nextProps, nextState) {
        // 只有当 prop.id 发生变化时才重新渲染
        return nextProps.id !== this.props.id;
      }
    
      render() {
        console.log('MyCustomComponent rendered');
        return <div>ID: {this.props.id}</div>;
      }
    }
    ```

  - **注意：** 实现 `shouldComponentUpdate` 需要非常小心，手动实现浅比较或深比较都有其复杂性，容易引入 bug。通常推荐使用 `PureComponent` 或 `React.memo`。

#### 2. 数据处理与计算优化：避免重复计算

- **`useMemo`：缓存计算结果**

  - **作用：** 用于缓存计算结果。只有当 `useMemo` 的依赖项发生变化时，它才会重新计算并返回新的值。

  - 最优范式：

     避免在每次渲染时都执行昂贵的计算。

    JavaScript

    ```
    import React, { useState, useMemo } from 'react';
    
    function UserList({ users, filterText }) {
      // 只有当 users 或 filterText 变化时，filteredUsers 才会重新计算
      const filteredUsers = useMemo(() => {
        console.log('Filtering users...');
        return users.filter(user => user.name.includes(filterText));
      }, [users, filterText]);
    
      return (
        <ul>
          {filteredUsers.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      );
    }
    ```

- **`useCallback`：缓存函数引用**

  - **作用：** 用于缓存函数引用。只有当 `useCallback` 的依赖项发生变化时，它才会返回一个新的函数引用。

  - 最优范式：

     结合 

    ```
    React.memo
    ```

     或 

    ```
    PureComponent
    ```

     使用，防止子组件因为父组件重新渲染而导致函数 prop 引用变化，从而引发不必要的重新渲染。

    JavaScript

    ```
    import React, { useState, useCallback, memo } from 'react';
    
    const MyButton = memo(({ onClick, label }) => {
      console.log('Button rendered');
      return <button onClick={onClick}>{label}</button>;
    });
    
    function ParentComponent() {
      const [count, setCount] = useState(0);
    
      // 只有当 count 变化时，handleClick 才会是新的函数引用
      const handleClick = useCallback(() => {
        setCount(prevCount => prevCount + 1);
      }, [count]); // 依赖 count
    
      return (
        <div>
          <p>Count: {count}</p>
          {/* 即使 ParentComponent 重新渲染，如果 handleClick 引用没变，MyButton 也不会重新渲染 */}
          <MyButton onClick={handleClick} label="Increment Count" />
        </div>
      );
    }
    ```

#### 3. 虚拟 DOM 优化：批处理与键（Keys）

- **批处理更新（Batching Updates）：**

  - **原理：** React 18 默认支持自动批处理更新。这意味着，在一次事件循环中（例如，在一个事件处理器内部），即使你多次调用 `setState`，React 也会将这些更新合并成一次，只进行一次重新渲染。这大大减少了不必要的渲染次数。
  - **了解即可：** 在 React 18 之前，只有在 React 事件处理器中的更新才会被批处理。现在，无论更新来自哪里（事件处理器、`Promises`、`setTimeout` 等），都会默认进行批处理，大大简化了性能优化。

- **列表渲染的 `key` prop：**

  - **作用：** 当渲染列表时，为每个列表项提供一个稳定、唯一的 `key` 值至关重要。`key` 帮助 React 识别哪些项发生了变化、被添加或被删除，从而高效地更新虚拟 DOM。

  - 最优范式：

     使用数据项本身的唯一 ID 作为 

    ```
    key
    ```

    。

    JavaScript

    ```
    function ItemList({ items }) {
      return (
        <ul>
          {items.map(item => (
            // item.id 应该是唯一且稳定的
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      );
    }
    ```

  - **禁忌：** 绝对不要使用数组索引 `index` 作为 `key`，除非你的列表是静态的、永不改变顺序且不会增删项。否则会导致渲染错误和性能问题。

#### 4. 代码分割（Code Splitting）与懒加载（Lazy Loading）：减小打包体积

- **作用：** 将应用代码分割成更小的块，按需加载，而不是一次性加载所有代码。这可以显著减少首次加载时间。

- **核心 API：** `React.lazy` 和 `Suspense`。

- 最优范式：

  JavaScript

  ```
  import React, { lazy, Suspense } from 'react';
  
  // 使用 React.lazy 动态导入组件
  const AboutPage = lazy(() => import('./AboutPage'));
  const ContactPage = lazy(() => import('./ContactPage'));
  
  function App() {
    return (
      <div>
        <h1>My App</h1>
        {/* 使用 Suspense 包裹懒加载的组件，提供加载状态 */}
        <Suspense fallback={<div>Loading...</div>}>
          <AboutPage />
          <ContactPage />
        </Suspense>
      </div>
    );
  }
  ```

- **应用场景：** 路由级别组件懒加载、大型模块按需加载。

#### 5. 图片优化：减少资源加载时间

- **图片压缩：** 使用工具压缩图片大小。
- **图片懒加载：** 使用 `loading="lazy"` 属性或第三方库，只加载进入视口的图片。
- **响应式图片：** 使用 `srcset` 和 `<picture>` 标签根据设备屏幕大小加载不同尺寸的图片。
- **WebP/AVIF 格式：** 优先使用这些新一代图片格式，它们通常具有更好的压缩率。

#### 6. 避免不必要的组件层级：简化 DOM 结构

- **作用：** 减少组件的嵌套层级和 DOM 节点的数量可以减少渲染和更新的开销。

- 范式：

   使用 

  ```
  React.Fragment
  ```

   或简写 

  ```
  <></>
  ```

   代替多余的 

  ```
  div
  ```

  。

  JavaScript

  ```
  // 之前
  function MyComponent() {
    return (
      <div>
        <ChildA />
        <ChildB />
      </div>
    );
  }
  
  // 之后
  function MyComponent() {
    return (
      <> {/* 或者 <React.Fragment> */}
        <ChildA />
        <ChildB />
      </>
    );
  }
  ```

#### 7. 使用 CDN：加速静态资源加载

- 将 React 库、React DOM 库以及其他第三方库部署到 CDN 上，利用 CDN 的全球分布式网络加速资源加载。

#### 8. 服务端渲染 (SSR) / 静态站点生成 (SSG)：提升首屏加载速度和 SEO

- **作用：** 在服务器端预先渲染 React 组件，直接返回 HTML 给浏览器，减少了客户端渲染的时间，提升了首屏加载速度和搜索引擎优化（SEO）。
- **常用框架：** Next.js、Gatsby。

#### 9. 性能分析工具：定位瓶颈

- **React Developer Tools Profiler：** 这是 React 官方提供的强大工具，可以在浏览器开发者工具中分析组件的渲染时间、更新原因等，帮你找到性能瓶颈。
- **Lighthouse：** Chrome 浏览器内置的审计工具，可以评估网页的性能、可访问性、最佳实践和 SEO 等。

### 总结与面试建议

React 性能优化是一个持续的过程，没有一劳永逸的解决方案。在面试中，除了列举这些手段，更重要的是展现你对性能优化的**思考方式**和**实际解决问题的能力**：

1. **分层理解：** 从组件渲染、数据处理、代码加载等不同层面去思考优化。
2. **核心原则：** 记住“减少不必要的渲染，减少计算量”。
3. **何时优化：** 强调**不要过早优化**！先完成功能，再用分析工具定位瓶颈，有针对性地优化。过度优化反而会增加代码复杂性，降低可维护性。
4. **Hands-on 经验：** 如果有实际项目中的优化经验，那是最好的加分项。比如你用 `React.memo` 解决了某个列表的性能问题，或者通过代码分割减少了首屏加载时间。



## 三、react的fiber架构

在咱们深入 Fiber 之前，先想想一个问题：**为什么 React 需要 Fiber？**

#### 1. 为什么需要 Fiber？——旧架构的痛点

在 Fiber 之前，React 的协调（Reconciliation）过程是基于“栈（Stack）”的。这意味着，当组件状态发生变化时，React 会**同步地、递归地**遍历整个组件树，计算出需要更新的部分，然后一次性地更新 DOM。

这种同步递归的模式有一个巨大的缺点：

- **不可中断：** 一旦更新开始，就必须一口气执行完，直到整个组件树的更新完成。
- **阻塞主线程：** 如果组件树很大，或者更新很复杂，这个过程就会占用 JavaScript 主线程很长时间，导致浏览器无法响应用户操作（比如点击、输入），页面就会出现卡顿、不流畅的感觉，也就是所谓的“掉帧”。

想象一下，你正在做一桌大餐，旧的 React 就像一个厨师，一旦开始切菜，就必须把所有菜都切完才能开始炒。如果菜太多，客人就得饿着肚子等很久。这就是旧架构的痛点。

为了解决这个痛点，React 团队彻底重写了协调器，引入了 **Fiber 架构**。

#### 2. 什么是 Fiber？——可中断的工作单元

Fiber 可以理解为 React 协调器（Reconciler）的全新实现。它将组件的渲染过程拆分成一个个**可中断、可恢复的工作单元**。

- **Fiber Node (Fiber 节点)：** 在 Fiber 架构中，每个 React 元素（比如你写的 `<Div>`、`<MyComponent>`）在内部都会对应一个 Fiber 节点。这个 Fiber 节点是一个 JavaScript 对象，包含了组件的类型、属性、状态、以及与其他 Fiber 节点（父、子、兄弟）的连接关系。它就像一个“任务清单”上的一个任务项。
- **Fiber 树：** 所有的 Fiber 节点会构成一棵 Fiber 树，它就是组件树的运行时表示。这棵树会不断地被构建和更新。

**核心理念：** React 不再是同步递归地处理组件树，而是将工作拆分成小块，每处理完一小块就检查一下有没有更高优先级的任务（比如用户输入），如果有，就暂停当前工作，先去处理紧急任务，处理完再回来继续之前的工作。

#### 3. Fiber 架构的核心优势：让应用“会呼吸”

正是因为这种“可中断、可恢复”的特性，Fiber 带来了巨大的优势：

- 可中断与可恢复 (Interruptible and Resumable)：
  - 这是 Fiber 最核心的能力。React 可以在渲染过程中暂停，把控制权交还给浏览器，让浏览器处理高优先级的事件（比如动画、用户输入），避免卡顿。
  - 等浏览器空闲时，React 再从上次暂停的地方继续渲染。这就让用户界面感觉更加流畅、响应迅速。
- 优先级调度 (Priority Scheduling)：
  - 不同的更新可以有不同的优先级。例如，用户输入的响应（如打字）优先级最高，动画次之，网络请求返回的数据更新可能优先级较低。
  - Fiber 可以根据优先级来决定先处理哪些任务，后处理哪些任务，确保关键的用户交互始终保持流畅。
- 副作用列表 (Effect List)：
  - Fiber 在渲染阶段（稍后会讲）会构建一个副作用列表（Effect List）。这个列表包含了所有需要对 DOM 进行操作（新增、删除、更新）的节点。
  - 在提交阶段，React 只需要遍历这个副作用列表，一次性地完成所有 DOM 操作，大大减少了 DOM 操作的次数和性能开销。
- 错误边界 (Error Boundaries)：
  - Fiber 使得错误边界（Error Boundaries）的实现成为可能。如果组件树的某个子树在渲染过程中发生错误，Fiber 可以捕获这个错误，然后卸载错误的子树，而不会导致整个应用崩溃。这大大提升了应用的健壮性。

#### 4. Fiber 的工作流程：两大阶段

Fiber 的工作可以分为两个主要阶段：

**阶段一：协调/渲染阶段 (Reconciliation/Render Phase) - “计划与比较”**

- 过程：

   这个阶段是可中断的。React 会遍历 Fiber 树，从根节点开始向下，对每个 Fiber 节点执行“工作”（Work）。

  - **构建新的 Fiber 树：** React 会根据最新的 `state` 和 `props`，与当前的 Fiber 树进行比较（Diff 算法），找出需要更新的 Fiber 节点，并生成一个新的 Fiber 树（被称为“WorkInProgress”树）。
  - **标记副作用：** 在比较过程中，如果发现某个节点需要进行 DOM 操作（比如新增、删除、更新属性、更新文本内容），就会在这个 Fiber 节点上打上“副作用标记”（Effect Tag），并将其添加到副作用列表中。
  - **不操作 DOM：** 在这个阶段，React **不会进行任何真实的 DOM 操作**。它只是在内存中构建和标记。

- **中断与恢复：** 这个阶段，React 会周期性地检查剩余的时间片，如果时间片用完，或者有更高优先级的任务，它就会暂停当前的工作，把控制权交给浏览器。等到下次空闲时，再从上次暂停的地方继续。

**阶段二：提交阶段 (Commit Phase) - “执行与更新”**

- **过程：** 这个阶段是不可中断的。一旦进入提交阶段，React 会遍历在协调阶段生成的副作用列表，并一次性地执行所有真实的 DOM 操作（插入、更新、删除等）。
- **同步执行：** 这个阶段必须同步完成，因为一旦开始修改 DOM，就不能中断，否则可能导致 UI 不一致。
- **生命周期/Hook 触发：** 在这个阶段，一些生命周期方法（如 `componentDidMount`、`componentDidUpdate`）和 `useEffect` 的回调函数也会被执行，因为 DOM 已经准备就绪。

**形象比喻：**

想象一下，你是一个餐厅老板，需要给顾客准备饭菜。

- **旧的 React (Stack Reconciler)：** 你是同步厨师。接到订单后，你一个人从买菜、洗菜、切菜、炒菜、摆盘，全程一口气干完，期间无论顾客多催，你都不能停下来。如果菜多，顾客就得等很久。

- 新的 React (Fiber Reconciler)：

   你是一个项目经理。

  - **协调/渲染阶段 (计划与比较)：** 你接到订单后，会先列一个详细的“任务清单”（WorkInProgress Fiber 树），记录每道菜需要做什么（比如：这道菜要切土豆、那道菜要炒肉）。这个清单你可以边做边检查，如果发现有顾客很着急（高优先级），你就放下手头的任务，先去处理紧急的，比如给他们上杯水。你只是在脑子里和纸上规划，不实际动手炒菜。
  - **提交阶段 (执行与更新)：** 当你确认所有菜品的“计划”都排好，并确定哪些菜需要实际操作（副作用列表）后，你就会进入厨房，找来一个厨师，把任务清单交给Ta。厨师会一次性地把所有需要实际操作的菜全部炒好、摆盘上桌。这个阶段是不可中断的，因为厨师一旦开始炒菜就不能停，否则菜就糊了。

#### 5. Fiber Node 的结构（简单了解）

每个 Fiber 节点大致包含以下核心属性：

- `type`：组件类型（函数、类、原生 DOM 元素等）。
- `props`：组件的 `props`。
- `stateNode`：对于类组件，指向组件实例；对于 DOM 元素，指向 DOM 节点。
- `child`：指向第一个子 Fiber 节点。
- `sibling`：指向下一个兄弟 Fiber 节点。
- `return`：指向父 Fiber 节点（构成单向链表，方便回溯）。
- `memoizedState`：存储 Hook 的链表（关键！）。
- `memoizedProps`：上次渲染的 `props`。
- `flags`：副作用标记，指示这个 Fiber 节点需要执行的 DOM 操作类型（新增、删除、更新等）。

通过 `child`、`sibling`、`return` 属性，Fiber 节点构建了一个链表结构，React 可以在内存中高效地遍历和操作这棵树。

#### 6. Fiber 与 Hook/性能优化的联系

虽然 Fiber 是 React 内部的实现，但它对我们写 React 代码和理解性能优化至关重要：

- **Hook 的基石：** Fiber 架构的可中断性，以及它在每个 Fiber 节点上存储 `memoizedState`（一个 Hook 链表）的能力，正是 Hook 能够实现的关键。如果 Fiber 不支持暂停和恢复，Hook 的状态就无法在渲染之间保持一致。
- **理解 `useEffect` 的执行时机和清理：** 掌握 Fiber 的协调和提交阶段，能更好地理解 `useEffect` 为什么会在 DOM 更新之后执行，以及它的清理函数为什么会在下一次 `effect` 执行前或组件卸载时执行。
- **强化 `React.memo` / `useCallback` / `useMemo` 的必要性：** 这些性能优化 Hook 的作用，就是帮助 Fiber 在**协调阶段**跳过不必要的比较和计算。如果 `props` 或计算结果没有变化，Fiber 就可以直接复用旧的 Fiber 节点，节省了大量计算资源，让应用更流畅。
- **`key` 的重要性：** `key` 帮助 Fiber 在协调阶段高效地识别列表项的变化，避免了不必要的 DOM 重建，从而提升列表渲染性能。
- **`Suspense` 和并发模式：** `Suspense`（用于数据加载等待）和 React 18 的并发模式，正是基于 Fiber 架构的可中断、可调度能力才能实现。

### 总结与未来展望

理解 Fiber 架构，就像是拿到了 React 引擎的说明书。你可能不需要亲自去“修理”引擎，但知道它如何工作，能让你更好地“驾驶”它，写出更高效、更稳定的 React 应用。

在面试中，当你能清晰地解释 Fiber 解决了什么问题，它是如何通过可中断的工作单元、优先级调度来提升用户体验，以及它如何支撑 Hook 和其他性能优化手段时，你展示的不仅是知识，更是对技术深度的追求。

Fiber 架构是 React 迈向未来（并发模式、服务端组件等）的关键一步，理解它，你就站在了前端技术的前沿。继续加油，你一定能完全掌握它，并在面试中大放异彩！



## 四、在react中使用redux

首先，咱们得明确一个问题：**为什么在 React 中需要 Redux？**

React 组件本身有自己的状态（`useState`），父子组件之间也能通过 `props` 传递数据。但当你的应用越来越大，组件层级越来越深时，你可能会遇到这些问题：

1. **Prop Drilling（逐级透传）：** 状态需要从顶层组件一层一层地通过 `props` 传递给深层子组件，代码变得非常冗长和难以维护。
2. **兄弟组件通信困难：** 两个没有直接父子关系的组件需要共享状态或互相通信时，变得非常复杂。
3. **状态逻辑分散：** 应用中的状态散落在各个组件内部，难以统一管理和调试。

Redux 就是为了解决这些问题而生的**状态管理库**。它提供了一个**可预测的状态容器**，让你的应用状态集中存放、统一管理，并以可预测的方式进行修改。

#### 1. Redux 的核心三要素（工作流）

理解 Redux 的核心，就是理解它的**单向数据流**。

- Store（数据仓库）：

   整个应用的

  唯一数据源

  。它是一个 JavaScript 对象，包含了应用的所有状态。

  - 就好比你家的“总账本”，所有资产负债都在这里记录。

- Action（动作）：

   一个普通的 JavaScript 对象，用于

  描述发生了什么

  。它至少包含一个 

  ```
  type
  ```

   字段，表示动作的类型，可以有其他字段携带数据。

  - 好比你写的“购物清单”或“银行转账单”，只描述你“想做什么”或“发生了什么”。
  - 例子：`{ type: 'ADD_TODO', text: '学习 Redux' }`

- Reducer（处理器）：

   一个

  纯函数

  。它接收当前的 

  ```
  state
  ```

   和一个 

  ```
  action
  ```

   作为参数，然后根据 

  ```
  action
  ```

   的类型，

  返回一个新的 `state`

  。

  - 它绝对不能直接修改原始 `state` 对象！

  - 好比银行的“会计师”，接到你的转账单（Action），根据当前的账本（State），计算出新的余额，然后写在新的账本上（返回新 State）。他不会撕掉旧账本的某一页去修改。

  - 例子：

    JavaScript

    ```
    function counterReducer(state = { value: 0 }, action) {
      switch (action.type) {
        case 'INCREMENT':
          return { value: state.value + 1 };
        case 'DECREMENT':
          return { value: state.value - 1 };
        default:
          return state; // 默认情况下返回当前 state
      }
    }
    ```

- Dispatch（派发）：

   唯一触发状态更新的方式。通过调用 

  ```
  store.dispatch(action)
  ```

   将 

  ```
  action
  ```

   发送给 Store。Store 会将 

  ```
  action
  ```

   传递给 Reducer，由 Reducer 计算出新的状态。

  - 好比你把“购物清单”交给店家，店家开始处理。

**数据流图示（口述）：**

```
你的 React 组件 (UI)
      |
      | (用户交互，例如点击按钮)
      V
Action (描述发生了什么)
      |
      | dispatch(action)
      V
Store (接收到 Action)
      |
      | 将当前 State 和 Action 传给 Reducer
      V
Reducer (纯函数，根据 Action 类型计算新 State)
      |
      | 返回新的 State
      V
Store (用新 State 更新自己)
      |
      | (Store 发生变化，通知订阅者)
      V
你的 React 组件 (UI 重新渲染，显示新 State)
```

#### 2. 在 React 中集成 Redux (`react-redux`)

为了让 React 组件能够“感知”Redux Store 的存在并与之交互，我们需要使用官方提供的集成库 `react-redux`。它提供了 Hook API，让函数组件使用 Redux 变得非常简洁。

**核心 Hook：**

- `Provider` 组件：

   这是 

  ```
  react-redux
  ```

   提供的最外层组件。你需要用它包裹你的整个 React 应用（通常在 

  ```
  src/index.js
  ```

   或 

  ```
  src/App.js
  ```

  ），并把你的 Redux Store 作为 

  ```
  prop
  ```

   传递给它。

  - **作用：** 它让所有被包裹的 React 组件都能通过 Context 访问到 Redux Store。

  JavaScript

  ```
  // src/index.js 或 src/App.js
  import React from 'react';
  import ReactDOM from 'react-dom/client'; // 或 'react-dom'
  import { Provider } from 'react-redux';
  import store from './app/store'; // 你的 Redux Store
  import App from './App';
  
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(
    <Provider store={store}>
      <App />
    </Provider>
  );
  ```

  

- 

- `useSelector` Hook：

   在函数组件中，用于从 Redux Store 的状态中

  读取数据

  。

  - 它接收一个“选择器”函数作为参数，这个函数会接收整个 Redux State 作为输入，并返回你需要的特定数据。
  - 当选择器返回的值发生变化时，组件会自动重新渲染。

  JavaScript

  ```
  import React from 'react';
  import { useSelector } from 'react-redux';
  
  function CounterDisplay() {
    // 从 Redux state 中选择 count.value
    const count = useSelector(state => state.counter.value); // 假设你的 reducer 名字是 'counter'
  
    return <div>当前计数: {count}</div>;
  }
  ```

- `useDispatch` Hook：

   在函数组件中，用于获取 

  ```
  dispatch
  ```

   函数，从而

  派发 Action

   来更新 Redux Store 的状态。

  JavaScript

  ```
  import React from 'react';
  import { useDispatch } from 'react-redux';
  
  function CounterControls() {
    const dispatch = useDispatch();
  
    return (
      <div>
        <button onClick={() => dispatch({ type: 'INCREMENT' })}>增加</button>
        <button onClick={() => dispatch({ type: 'DECREMENT' })}>减少</button>
      </div>
    );
  }
  ```

#### 3. 现代 Redux 最佳实践：Redux Toolkit (RTK)

传统的 Redux 配置和编写方式会产生大量的“样板代码”（boilerplate），比如手动定义 Action Types、Action Creators、编写 Switch Case 臃肿的 Reducer。这让很多初学者望而却步，也让开发效率降低。

**Redux Toolkit (RTK)** 是 Redux 官方推荐的工具集，它旨在简化 Redux 的开发流程，减少样板代码，并提供内置的最佳实践。**现在学习 Redux，强烈推荐直接从 Redux Toolkit 开始！**

**RTK 的核心优势：**

- **减少样板代码：** 自动化创建 Action Types、Action Creators 和 Reducers。
- **内置 Immer：** 让你可以在 Reducer 中“看似”直接修改 State，而实际上 Immer 会在底层帮你生成不可变的 State。
- **更好的开发体验：** 统一的配置和模块化的设计。

**RTK 核心 API：**

- **`configureStore`：** 替代传统的 `createStore`，提供更好的默认配置（如 Redux DevTools 集成、`redux-thunk` 中间件等）。
- **`createSlice`：** 这是 RTK 的“明星”API。它允许你定义一个“切片”（slice），一个切片就包含了 `name`、`initialState`、`reducers`（处理同步逻辑）和 `extraReducers`（处理异步逻辑）等。它会自动生成 Action Creators 和 Reducer。
- **RTK Query：** (简要提及) RTK Query 是 RTK 的一部分，专门用于简化数据获取和缓存，几乎可以替代很多场景下的 `redux-thunk` 或 `redux-saga`。

**使用 `createSlice` 的例子 (Counter 应用)：**

1. **定义 Slice：**

   JavaScript

   ```
   // src/features/counter/counterSlice.js
   import { createSlice } from '@reduxjs/toolkit';
   
   export const counterSlice = createSlice({
     name: 'counter', // slice 的名称，也会用于生成 action type 前缀
     initialState: {
       value: 0,
     },
     reducers: {
       // 在这里定义同步的 action 和对应的 reducer 逻辑
       increment: (state) => {
         // Immer 使得你可以“直接”修改 state，底层会自动生成新状态
         state.value += 1;
       },
       decrement: (state) => {
         state.value -= 1;
       },
       incrementByAmount: (state, action) => {
         state.value += action.payload; // action.payload 是传递过来的数据
       },
     },
   });
   
   // 导出 action creators
   export const { increment, decrement, incrementByAmount } = counterSlice.actions;
   
   // 导出 reducer
   export default counterSlice.reducer;
   ```

2. **配置 Store：**

​    // src/app/store.js

import { configureStore } from '@reduxjs/toolkit';

import counterReducer from '../features/counter/count1erSlice'; // 导入上面定义的 reducer

````
export default configureStore({
  reducer: {
    counter: counterReducer, // 将 reducer 挂载到 store 的 counter 属性上
    // 可以有多个 reducer，每个对应一个应用的状态切片
  },
});
```
````

1. 在 React 组件中使用：

   JavaScript

   ```
   // src/features/counter/Counter.js
   import React from 'react';
   import { useSelector, useDispatch } from 'react-redux';
   import { increment, decrement, incrementByAmount } from './counterSlice'; // 导入 action creators
   
   export function Counter() {
     const count = useSelector((state) => state.counter.value); // 从 store 中获取 counter.value
     const dispatch = useDispatch();
   
     return (
       <div>
         <p>当前计数: {count}</p>
         <div>
           <button onClick={() => dispatch(increment())}>增加</button> {/* 直接调用 action creator */}
           <button onClick={() => dispatch(decrement())}>减少</button>
           <button onClick={() => dispatch(incrementByAmount(5))}>增加 5</button> {/* 传递 payload */}
         </div>
       </div>
     );
   }
   ```

#### 4. 何时使用 Redux？

虽然 Redux 功能强大，但并不是所有 React 应用都需要它。

**推荐使用 Redux 的场景：**

- **应用状态复杂且庞大：** 多个组件需要共享大量状态，或者状态更新逻辑非常复杂。
- **状态变化频繁且难以追踪：** 需要明确地知道状态是如何以及何时改变的，方便调试。
- **需要时间旅行调试、状态快照等高级调试功能。**
- **应用规模较大，需要统一的状态管理规范。**
- **需要处理大量异步数据流（配合 Redux Thunk 或 Redux Saga / RTK Query）。**

**不推荐使用 Redux 的场景：**

- **小型应用或简单组件：** 仅仅是父子组件之间的状态传递，或者组件内部的简单状态，`useState` 和 `useContext` 就足够了，没必要引入 Redux 增加复杂度。
- **只关心 UI 状态（如弹窗是否打开）：** 这些状态通常可以放在组件内部管理。

#### 总结与鼓励

Redux 及其现代化的 Redux Toolkit，为 React 应用提供了强大且可预测的状态管理解决方案。理解它的核心概念、单向数据流，并熟练运用 `react-redux` 的 Hook API，特别是拥抱 Redux Toolkit，将大大提升你开发大型复杂 React 应用的能力。

别被一开始的 Redux 概念图吓到，只要跟着 RTK 的最佳实践走，你会发现 Redux 变得非常直观和高效。多写代码，多思考状态的流转，你很快就能驾驭 Redux，让你的 React 应用状态管理得井井有条，丝滑流畅！

继续加油！你已经掌握了 React 的核心，Redux 只是锦上添花，助你更上一层楼！





## 五、在react中使用react-router

#### 1. 为什么我们需要 React Router？

传统的网页应用，每次点击链接都会向服务器发送请求，然后服务器返回一个全新的 HTML 页面，导致页面白屏和加载延迟。而单页应用（SPA）的核心理念是：只加载一次 HTML 页面，然后通过 JavaScript 动态地更新页面内容。

- **问题：** 既然只有一个 HTML 页面，那用户怎么能在“不同页面”之间切换呢？比如从“首页”到“关于我们”页面，URL 怎么变化？

- 解决方案：

   这就是 

  客户端路由

   的作用。React Router 就是最流行的客户端路由库之一，它帮助我们：

  - **将 URL 映射到 React 组件：** 当 URL 变化时，它会渲染对应的组件。
  - **实现无刷新导航：** 页面内容在客户端动态更新，避免了白屏。
  - **管理历史记录：** 支持浏览器的前进、后退按钮。

#### 2. React Router 的核心概念和组件

React Router v6+ 引入了一些新的概念和更简洁的 API。

- **`BrowserRouter`：**

  - **作用：** 它是整个路由功能的“容器”或者说“路由器”。你需要在你的 React 应用的最顶层（通常是 `App.js` 的 `return` 语句中，或者 `index.js` 的 `ReactDOM.render` 部分）用它包裹住所有需要路由功能的组件。
  - **原理：** 它使用 HTML5 History API (PushState, ReplaceState) 来保持 UI 与 URL 的同步，是 Web 应用最常用的路由器。

  JavaScript

  ```
  // src/index.js (或者 App.js)
  import React from 'react';
  import ReactDOM from 'react-dom/client';
  import { BrowserRouter } from 'react-router-dom'; // 导入 BrowserRouter
  import App from './App';
  
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(
    <React.StrictMode>
      <BrowserRouter> {/* 用 BrowserRouter 包裹你的应用 */}
        <App />
      </BrowserRouter>
    </React.StrictMode>
  );
  ```

- **`Routes`：**

  - **作用：** 它是一个容器，里面包含了一组 `Route` 组件。`Routes` 的智能之处在于，它会遍历其内部的 `Route` 组件，然后只渲染**第一个**匹配当前 URL 的 `Route`。
  - **特点：** 解决了 v5 中 `Switch` 的痛点，更简洁。

  JavaScript

  ```
  // src/App.js
  import { Routes, Route } from 'react-router-dom';
  import HomePage from './pages/HomePage';
  import AboutPage from './pages/AboutPage';
  import ContactPage from './pages/ContactPage';
  import NotFoundPage from './pages/NotFoundPage';
  
  function App() {
    return (
      <Routes> {/* Routes 容器 */}
        <Route path="/" element={<HomePage />} /> {/* 匹配 "/" 渲染 HomePage */}
        <Route path="/about" element={<AboutPage />} /> {/* 匹配 "/about" 渲染 AboutPage */}
        <Route path="/contact" element={<ContactPage />} /> {/* 匹配 "/contact" 渲染 ContactPage */}
        <Route path="*" element={<NotFoundPage />} /> {/* 匹配所有未定义的路径，通常用于 404 页面 */}
      </Routes>
    );
  }
  ```

- **`Route`：**

  - 作用：

     定义一个路由规则。它有两个主要属性：

    - `path`：一个字符串，表示要匹配的 URL 路径。
    - `element`：当 `path` 匹配时，要渲染的 React 元素（通常是你的组件）。

  - **注意：** v6 移除了 `component` 和 `render` prop，统一使用 `element` prop。

  JavaScript

  ```
  <Route path="/products/:productId" element={<ProductDetail />} /> {/* 带有动态参数 */}
  ```

- **`Link`：**

  - **作用：** 这是在 React 应用中进行页面导航的**首选方式**。它会渲染成一个 `<a>` 标签，但点击时不会触发浏览器进行完整的页面刷新，而是由 React Router 接管，进行客户端路由跳转。
  - **属性：** 最常用的是 `to`，指定要跳转的路径。

  JavaScript

  ```
  import { Link } from 'react-router-dom';
  
  function Navigation() {
    return (
      <nav>
        <Link to="/">首页</Link>
        <Link to="/about">关于我们</Link>
        <Link to="/contact">联系方式</Link>
      </nav>
    );
  }
  ```

#### 3. React Router v6+ 的核心 Hook

在函数组件中，我们通常使用 Hook 来获取路由相关的信息或进行编程导航。

- **`useParams()`：获取 URL 中的动态参数**

  - **作用：** 当你的路由路径中包含动态部分（例如 `/users/:id`），`useParams` Hook 可以让你轻松地获取这些动态参数的值。

  - 例子：

    JavaScript

    ```
    // 定义路由：<Route path="/users/:userId" element={<UserProfile />} />
    import { useParams } from 'react-router-dom';
    
    function UserProfile() {
      const { userId } = useParams(); // URL 是 /users/123，userId 就会是 "123"
      return <h2>用户 ID: {userId}</h2>;
    }
    ```

- **`useNavigate()`：进行编程式导航**

  - **作用：** 当你需要在事件处理器（如表单提交后）、`useEffect` 中或任何不能直接使用 `Link` 的地方进行页面跳转时，可以使用 `useNavigate`。它返回一个 `Maps` 函数。

  - 例子：

    JavaScript

    ```
    import { useNavigate } from 'react-router-dom';
    
    function LoginForm() {
      const navigate = useNavigate();
    
      const handleSubmit = (e) => {
        e.preventDefault();
        // ... 处理登录逻辑 ...
        const success = true; // 假设登录成功
        if (success) {
          navigate('/dashboard'); // 导航到仪表盘页面
          // navigate(-1); // 返回上一页
          // navigate('/some-path', { replace: true }); // 替换当前历史记录而不是推入新记录
        }
      };
    
      return (
        <form onSubmit={handleSubmit}>
          {/* ... 表单内容 ... */}
          <button type="submit">登录</button>
        </form>
      );
    }
    ```

- **`useLocation()`：获取当前 URL 的信息**

  - **作用：** 返回一个 `location` 对象，包含当前 URL 的各种信息，如 `pathname` (路径), `search` (查询参数), `hash` (哈希值), `state` (通过 `Maps` 传递的状态)。

  - 例子：

    JavaScript

    ```
    import { useLocation } from 'react-router-dom';
    
    function CurrentPageInfo() {
      const location = useLocation();
      console.log(location.pathname); // 当前路径，如 "/about"
      console.log(location.search);   // 查询参数，如 "?name=Alice"
      console.log(location.state);    // navigate 传过来的 state
    
      return (
        <p>你当前在: {location.pathname}</p>
      );
    }
    ```

- **`useOutlet()`：渲染嵌套路由的子组件**

  - **作用：** 在父级路由组件中，`useOutlet`（或直接使用 `<Outlet />` 组件）用于渲染其匹配的子路由组件。这是实现**嵌套路由**的关键。

#### 4. 嵌套路由：构建复杂布局

嵌套路由允许你将父组件的布局和子组件的特定内容结合起来。

- 口述图示：嵌套路由的结构

  想象你有一个用户管理页面 /users。这个页面有导航栏和侧边栏，而它的子路由 /users/profile 和 /users/settings 只改变右侧的内容区域。

  ```
  // App.js (主路由文件)
  <Routes>
    <Route path="/" element={<HomePage />} />
    <Route path="/users" element={<UserLayout />}> {/* 父级路由，渲染 UserLayout */}
      <Route index element={<UserDashboard />} /> {/* 默认子路由，当 path="/users" 时渲染 */}
      <Route path="profile" element={<UserProfile />} /> {/* 完整路径 /users/profile */}
      <Route path="settings" element={<UserSettings />} /> {/* 完整路径 /users/settings */}
    </Route>
    {/* ... 其他路由 ... */}
  </Routes>
  
  // UserLayout.js (父级路由组件)
  import { Outlet, Link } from 'react-router-dom';
  
  function UserLayout() {
    return (
      <div>
        <h1>用户管理</h1>
        <nav>
          <Link to="/users">仪表盘</Link> | {' '}
          <Link to="/users/profile">个人资料</Link> | {' '}
          <Link to="/users/settings">设置</Link>
        </nav>
        <hr />
        <div style={{ border: '1px solid gray', padding: '10px' }}>
          {/* Outlet 会在这里渲染匹配到的子路由组件 */}
          <Outlet />
        </div>
      </div>
    );
  }
  
  // UserDashboard.js, UserProfile.js, UserSettings.js 都是普通的 React 组件
  ```

- **注意 `index` 路由：** `index` 路由表示当父级路由路径完全匹配（没有子路径）时，默认渲染的子组件。

- **子路由 `path`：** 子路由的 `path` 不需要以 `/` 开头，它会自动相对于父路由的 `path`。例如，`path="profile"` 意味着完整路径是 `/users/profile`。

#### 5. 重要的考虑和最佳实践

- 404 Not Found 页面：

  - 使用 `Routes` 容器的最后一个 `Route`，将 `path` 设置为 `*` (星号，通配符)。它会匹配所有未被前面路由匹配的路径。
  - `<Route path="*" element={<NotFoundPage />} />`

- 路由守卫（Authentication / Protected Routes）：

  - React Router 本身不提供内置的路由守卫功能，但你可以通过**在 `element` Prop 中嵌套一个高阶组件 (HOC) 或一个条件渲染组件**来实现。

  - 例如：

    JavaScript

    ```
    // ProtectedRoute.js
    import { Navigate } from 'react-router-dom';
    import { useAuth } from './authContext'; // 假设你有一个认证 context
    
    function ProtectedRoute({ children }) {
      const { isAuthenticated } = useAuth(); // 检查用户是否认证
      if (!isAuthenticated) {
        return <Navigate to="/login" replace />; // 未认证则重定向到登录页
      }
      return children; // 认证通过则渲染子组件
    }
    
    // App.js
    <Routes>
      <Route path="/login" element={<LoginPage />} />
      <Route path="/dashboard" element={
        <ProtectedRoute>
          <DashboardPage />
        </ProtectedRoute>
      } />
    </Routes>
    ```

- **使用 `useResolvedPath` 或 `useMatch` 进行更高级的路径匹配和解析。**

- 性能优化：

   结合 

  ```
  React.lazy
  ```

   和 

  ```
  Suspense
  ```

   对路由组件进行

  懒加载

  ，可以显著提升应用的首次加载速度。

  JavaScript

  ```
  import React, { lazy, Suspense } from 'react';
  import { Routes, Route } from 'react-router-dom';
  
  const HomePage = lazy(() => import('./pages/HomePage'));
  const AboutPage = lazy(() => import('./pages/AboutPage'));
  
  function App() {
    return (
      <Suspense fallback={<div>加载中...</div>}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/about" element={<AboutPage />} />
        </Routes>
      </Suspense>
    );
  }
  ```

### 总结与展望

React Router 是构建现代单页应用的基石。它让你的应用在没有页面刷新的情况下实现复杂的导航，大大提升了用户体验。

- **核心组件：** `BrowserRouter` (路由器容器), `Routes` (路由集合), `Route` (单个路由规则), `Link` (声明式导航)。
- **核心 Hook：** `useParams` (获取路径参数), `useNavigate` (编程导航), `useLocation` (获取当前路由信息), `useOutlet` (渲染嵌套子路由)。
- **最佳实践：** 拥抱 v6 的简洁 API，合理使用嵌套路由，利用 `*` 路径处理 404，通过 `lazy`/`Suspense` 进行代码分割，并考虑如何实现路由守卫。

掌握 React Router，你就能搭建起复杂且用户体验极佳的单页应用。多动手实践，尝试不同的路由场景，你很快就能成为路由高手！

继续加油！你的前端技能树正在茁壮成长！



## 六、编码react计数器和todolist

提示：计数器和todolist可以在完成react的基础语法学习后就进行尝试！