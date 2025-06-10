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

```
import React, { useState } from 'react';

function App() {
  // 用于添加新事项的输入框值
  const [newItem, setNewItem] = useState('');
  // 存储所有待办事项的列表
  const [todoList, setTodoList] = useState(['学习 React Hook', '复习 Vue 3', '准备实习面试']);

  // 用于编辑功能的状态
  const [editingIndex, setEditingIndex] = useState(null); // 存储当前正在编辑的事项的索引，null 表示没有事项在编辑
  const [editingText, setEditingText] = useState('');    // 存储当前正在编辑的事项的临时文本

  // 处理添加新事项输入框内容变化的函数
  const handleNewItemChange = (event) => {
    setNewItem(event.target.value);
  };

  // 添加待办事项的函数
  const addItem = () => {
    if (newItem.trim()) { // 确保输入不为空
      setTodoList([...todoList, newItem.trim()]); // 使用展开运算符添加新项
      setNewItem(''); // 清空输入框
    }
  };

  // 删除待办事项的函数
  const removeItem = (indexToRemove) => {
    // 使用 filter 创建一个新数组，排除掉要删除的项
    const updatedList = todoList.filter((_, i) => i !== indexToRemove);
    setTodoList(updatedList);

    // 如果删除的是当前正在编辑的事项，则重置编辑状态
    if (editingIndex === indexToRemove) {
      cancelEdit();
    } else if (editingIndex !== null && editingIndex > indexToRemove) {
      // 如果删除的事项在正在编辑的事项之前，需要调整 editingIndex
      setEditingIndex(editingIndex - 1);
    }
  };

  // 进入编辑模式的函数
  const editItem = (index, itemText) => {
    setEditingIndex(index);    // 设置当前编辑的索引
    setEditingText(itemText);  // 将当前事项的文本赋值给编辑输入框
  };

  // 处理编辑输入框内容变化的函数
  const handleEditingTextChange = (event) => {
    setEditingText(event.target.value);
  };

  // 保存编辑的函数
  const saveEdit = () => {
    if (editingIndex !== null && editingText.trim()) {
      // 创建一个新数组，更新对应索引的待办事项文本
      const updatedList = todoList.map((item, i) =>
        i === editingIndex ? editingText.trim() : item
      );
      setTodoList(updatedList);
      // 重置编辑状态
      setEditingIndex(null);
      setEditingText('');
    } else if (editingIndex !== null && !editingText.trim()) {
      // 如果编辑后内容为空，选择删除该事项
      removeItem(editingIndex);
      // 因为 removeItem 内部会处理编辑状态的重置，这里不需要重复调用 cancelEdit
    }
  };

  // 取消编辑的函数
  const cancelEdit = () => {
    setEditingIndex(null); // 重置编辑索引
    setEditingText('');    // 清空编辑文本
  };

  return (
    <div className="p-4 max-w-md mx-auto bg-white rounded-xl shadow-md space-y-4 font-inter">
      <h1 className="text-2xl font-bold text-gray-800 text-center">我的待办事项 (React CRUD)</h1>

      {/* 添加事项区域 */}
      <div className="flex space-x-2">
        <input
          type="text"
          className="flex-grow p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          value={newItem}
          onChange={handleNewItemChange}
          onKeyPress={(e) => {
            if (e.key === 'Enter') {
              addItem();
            }
          }}
          placeholder="添加新的待办事项"
        />
        <button
          onClick={addItem}
          className="px-4 py-2 bg-blue-600 text-white font-semibold rounded-md shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
        >
          添加
        </button>
      </div>

      {/* 待办事项列表 (读取/显示功能) */}
      <ul className="space-y-2">
        {todoList.map((item, index) => (
          <li
            key={index} // 在 React 循环渲染列表时，key 是非常重要的
            className="flex items-center justify-between p-3 bg-gray-100 rounded-md shadow-sm"
          >
            {/* 条件渲染：根据是否处于编辑模式显示不同内容 */}
            {editingIndex === index ? (
              // 编辑模式 (Update)
              <div className="flex flex-grow items-center">
                <input
                  type="text"
                  value={editingText}
                  onChange={handleEditingTextChange}
                  onKeyPress={(e) => {
                    if (e.key === 'Enter') {
                      saveEdit();
                    } else if (e.key === 'Escape') {
                      cancelEdit();
                    }
                  }}
                  className="flex-grow p-2 border border-blue-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
                <div className="flex space-x-2 ml-2">
                  <button
                    onClick={saveEdit}
                    className="px-3 py-1 bg-green-500 text-white font-semibold rounded-md shadow-sm hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2"
                  >
                    保存
                  </button>
                  <button
                    onClick={cancelEdit}
                    className="px-3 py-1 bg-gray-500 text-white font-semibold rounded-md shadow-sm hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2"
                  >
                    取消
                  </button>
                </div>
              </div>
            ) : (
              // 显示模式 (Read)
              <>
                <span className="text-gray-700">{item}</span>
                <div className="flex space-x-2">
                  <button
                    onClick={() => editItem(index, item)}
                    className="px-3 py-1 bg-yellow-500 text-white font-semibold rounded-md shadow-sm hover:bg-yellow-600 focus:outline-none focus:ring-2 focus:ring-yellow-500 focus:ring-offset-2"
                  >
                    编辑
                  </button>
                  <button
                    onClick={() => removeItem(index)}
                    className="px-3 py-1 bg-red-500 text-white font-semibold rounded-md shadow-sm hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2"
                  >
                    删除
                  </button>
                </div>
              </>
            )}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;

```

