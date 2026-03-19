1. React 时间切片（Time Slicing）与调度策略
1.1 计算机单处理器调度策略
单处理器在任意时刻只能执行一个任务，操作系统通过调度算法决定哪个进程/线程获得 CPU。核心策略如下：
FCFS（先来先服务）：按到达顺序执行，简单但会导致"护航效应"——一个长任务阻塞后面所有短任务。
SJF（最短作业优先）：优先执行预计耗时最短的任务，平均等待时间最优，但可能导致长任务"饥饿"（永远得不到执行）。
优先级调度（Priority Scheduling）：每个任务有优先级，高优先级先执行。存在饥饿问题，通常通过"老化（Aging）"机制解决——随着等待时间增长，任务优先级逐渐提升。
时间片轮转（Round Robin, RR）：每个任务分配固定时间片（Time Quantum），时间片用完就让出 CPU，排到队尾。响应性好，但时间片大小是关键——太大退化为 FCFS，太小上下文切换开销过高。
多级反馈队列（MLFQ）：多个队列对应不同优先级，新任务进入最高优先级队列，用完时间片后降级到下一级。兼顾了响应性和吞吐量。
1.2 React Scheduler 使用的策略
React 的调度器融合了 优先级调度 + 时间片轮转 + 多级反馈队列 思想：
优先级调度：React 定义了 5 个优先级（源码中对应 Scheduler_ImmediatePriority 到 Scheduler_IdlePriority），每个优先级对应不同的过期时间（timeout）：
ImmediatePriority   → -1ms（立即过期，同步执行）
UserBlockingPriority → 250ms
NormalPriority       → 5000ms
LowPriority          → 10000ms
IdlePriority         → maxSigned31BitInt（永不过期）
时间片轮转：每个工作循环（workLoop）中，React 给任务分配约 5ms 的时间片。执行 Fiber 节点的 reconciliation 时，每处理完一个 Fiber 节点就检查 shouldYield()。如果时间片用完，就把控制权交还给浏览器，让浏览器处理渲染和用户交互，剩余工作在下一帧继续。
类 MLFQ 思想：React 内部维护两个队列——taskQueue（已就绪的任务，按过期时间排序的小顶堆）和 timerQueue（延迟任务，按开始时间排序）。当 timerQueue 中的任务到达开始时间后，会转移到 taskQueue。同时通过过期时间实现"老化"——等待越久的任务越接近过期，优先级实质上越高，避免低优先级任务饥饿。
1.3 为什么不用 requestIdleCallback
React 最初确实考虑过 requestIdleCallback（rIC），但最终自己实现了调度器，原因如下：
兼容性差：rIC 在 Safari 中长期不支持（直到近年才实验性支持），在 IE 中完全不支持。React 需要跨浏览器一致的行为。
触发频率太低：rIC 依赖浏览器"空闲时间"，在持续动画或高频交互场景下，可能长时间不触发回调。实测中 rIC 通常一帧只触发一次，且空闲时间不可控（0~50ms 波动大），导致任务执行时机不可预测。
时间片粒度不可控：rIC 提供的 IdleDeadline.timeRemaining() 最多约 50ms，但实际剩余时间取决于浏览器，React 无法精确控制时间片大小（React 需要稳定的 5ms）。
无法控制优先级：rIC 没有优先级概念，所有任务都是"空闲时执行"。React 需要精细的多级优先级控制。
React 的替代方案：使用 MessageChannel 实现调度。MessageChannel 属于宏任务，触发时机在每一帧渲染之后，不会像 setTimeout(fn, 0) 那样有 4ms 的最低延迟限制（浏览器对嵌套 setTimeout 有最低 4ms 的 clamp），能更高效地利用每一帧的剩余时间。具体做法是：
jsconst channel = new MessageChannel();
const port = channel.port2;
channel.port1.onmessage = performWorkUntilDeadline;
// 请求调度时
port.postMessage(null);

2. useContext / useMemo / useCallback
2.1 useContext
作用：消费 React Context，让组件直接读取最近的 Provider 提供的值，避免 props 逐层传递（prop drilling）。
实现原理：

调用 useContext(MyContext) 时，React 沿着 Fiber 树向上查找最近的 MyContext.Provider，读取其 value。
内部会在当前 Fiber 节点上建立对该 Context 的依赖关系。当 Provider 的 value 变化时，React 会遍历所有消费该 Context 的 Fiber 节点，标记它们需要更新。
对比使用的是 Object.is 浅比较。如果 Provider 的 value 是一个新的对象引用（即使内容相同），所有消费者都会重新渲染。

注意：useContext 本身不提供性能优化，当 Context value 变化时，所有消费该 Context 的组件都会重新渲染，无论它们是否实际用到了变化的部分。
2.2 useMemo
作用：缓存计算结果。只有当依赖项变化时才重新计算，否则直接返回缓存值。
实现原理：

首次渲染（mount）时，执行计算函数，将结果和依赖项存储在 Fiber 节点的 memoizedState 链表中。
后续渲染（update）时，用 Object.is 逐一比较新旧依赖项。如果所有依赖项都相同，直接返回缓存的值；否则重新执行计算函数并更新缓存。

js// 简化的内部逻辑
function useMemo(create, deps) {
  const hook = getOrCreateHook();
  if (hook.memoizedState !== null) {
    const [prevValue, prevDeps] = hook.memoizedState;
    if (areHookInputsEqual(deps, prevDeps)) {
      return prevValue; // 依赖没变，返回缓存
    }
  }
  const value = create();
  hook.memoizedState = [value, deps];
  return value;
}
性能提升：避免在每次渲染中都执行昂贵的计算；配合 React.memo 使用时，稳定的引用可以防止子组件不必要的重新渲染。
2.3 useCallback
作用：缓存函数引用。本质上是 useMemo 的语法糖。
实现原理：

useCallback(fn, deps) 等价于 useMemo(() => fn, deps)。
不缓存函数的执行结果，而是缓存函数本身的引用。依赖项不变时，返回同一个函数引用。

性能提升场景：

将回调函数传给使用 React.memo 包裹的子组件时，稳定的函数引用能避免子组件因为"每次渲染都拿到新函数"而重新渲染。
作为 useEffect 的依赖项时，稳定的引用能避免 effect 不必要的重新执行。

2.4 为什么能提升渲染性能
核心逻辑是：React 的重新渲染是递归的。父组件重新渲染时，默认所有子组件都会重新渲染。性能优化的本质就是减少不必要的重新渲染和重复计算：

useMemo 避免重复的昂贵计算，同时产生稳定的值引用。
useCallback 产生稳定的函数引用。
稳定的引用 + React.memo = 子组件可以通过浅比较 props 来跳过不必要的渲染。
useContext 本身不直接提升性能，但合理拆分 Context（将频繁变化和不常变化的值分开）可以缩小重新渲染的范围。


3. React 中的副作用（Side Effects）
3.1 什么是副作用
在函数式编程的语境下，"纯函数"是给定相同输入总是返回相同输出、且不产生外部可观察变化的函数。副作用就是纯函数之外的一切——任何与外部世界交互的操作：

DOM 操作（增删改节点、修改样式）
数据请求（fetch、XHR）
订阅（WebSocket、事件监听、定时器）
修改外部变量或全局状态
浏览器 API 调用（localStorage、history 等）
日志打印

3.2 React 组件中的副作用
React 的设计理念是：渲染应该是纯的（UI = f(state)），副作用需要被隔离和管理。
useEffect：在渲染完成后（DOM 更新后）异步执行。适用于数据获取、订阅、手动 DOM 操作等。不阻塞浏览器绘制。
jsxuseEffect(() => {
  const subscription = dataSource.subscribe(handleChange);
  return () => subscription.unsubscribe(); // 清理函数
}, [dataSource]);
useLayoutEffect：在 DOM 更新后、浏览器绘制前同步执行。适用于需要读取 DOM 布局信息并同步修改 DOM 的场景（如测量元素尺寸、防止闪烁）。会阻塞浏览器绘制。
3.3 副作用在 React 内部的处理
在 React 的 Fiber 架构中，副作用通过 effectTag（flags） 来标记：

Render 阶段（可中断）：遍历 Fiber 树进行 diff，给需要变更的 Fiber 节点打上标记（如 Placement、Update、Deletion）。这个阶段是纯的，不产生可见的副作用。
Commit 阶段（不可中断）：同步执行所有副作用，分三个子阶段：

Before Mutation：读取 DOM 快照（如 getSnapshotBeforeUpdate）
Mutation：执行 DOM 增删改
Layout：执行 useLayoutEffect，此时 DOM 已更新但浏览器还没绘制
之后浏览器绘制
异步调度执行 useEffect



3.4 React 18+ 的严格模式与副作用
在开发模式下，React 18 的 StrictMode 会故意双重调用 effect（mount → unmount → mount），目的是帮助开发者发现缺少清理逻辑的副作用 bug，为未来的并发特性（如 Offscreen API）做准备。

4. Vue $nextTick 的实现
4.1 核心概念
Vue 的 DOM 更新是异步的。当响应式数据变化时，Vue 不会立即更新 DOM，而是将更新推入一个队列，在当前同步代码执行完毕后的下一个"tick"中批量执行。$nextTick 就是让用户代码在 DOM 更新完成后执行。
4.2 Vue 2 的实现（降级策略）
Vue 2 的 nextTick 实现经历了多次迭代，最终版本采用从微任务到宏任务的降级链：
Promise.then（微任务）
→ MutationObserver（微任务）
→ setImmediate（宏任务，仅 IE/Node）
→ setTimeout(fn, 0)（宏任务，兜底）
优先使用微任务的原因：微任务在当前宏任务结束后、浏览器渲染前执行，能保证 DOM 更新在同一帧内完成，用户不会看到中间状态（不闪烁）。
具体实现细节：
js// Vue 2.6+ 简化逻辑
const callbacks = [];
let pending = false;

function flushCallbacks() {
  pending = false;
  const copies = callbacks.slice(0);
  callbacks.length = 0;
  for (let i = 0; i < copies.length; i++) {
    copies[i]();
  }
}

let timerFunc;

if (typeof Promise !== 'undefined') {
  // 优先 Promise.then（微任务）
  const p = Promise.resolve();
  timerFunc = () => p.then(flushCallbacks);
} else if (typeof MutationObserver !== 'undefined') {
  // MutationObserver 也是微任务
  let counter = 1;
  const observer = new MutationObserver(flushCallbacks);
  const textNode = document.createTextNode(String(counter));
  observer.observe(textNode, { characterData: true });
  timerFunc = () => { counter = (counter + 1) % 2; textNode.data = String(counter); };
} else if (typeof setImmediate !== 'undefined') {
  // setImmediate（宏任务，但比 setTimeout 优先级高）
  timerFunc = () => setImmediate(flushCallbacks);
} else {
  // 兜底 setTimeout
  timerFunc = () => setTimeout(flushCallbacks, 0);
}
Vue 2 历史上的一个坑：2.5.x 版本曾在某些场景下将微任务降级为宏任务（MessageChannel），因为纯微任务在某些边界场景下会导致事件冒泡时序问题（比如在事件回调中修改数据，微任务在冒泡完成前就执行了 DOM 更新）。但 2.6.x 又改回了优先微任务的策略。
4.3 Vue 3 的实现
Vue 3 大幅简化了实现，直接使用 Promise.then，不再做降级：
js// Vue 3 简化逻辑
const resolvedPromise = Promise.resolve();
let currentFlushPromise = null;

function nextTick(fn) {
  const p = currentFlushPromise || resolvedPromise;
  return fn ? p.then(fn) : p;
}
为什么 Vue 3 不需要降级：Vue 3 放弃了对 IE 的支持，所有现代浏览器都原生支持 Promise，不需要 fallback。
4.4 微任务 vs 宏任务的特点对比
微任务（Microtask）：Promise.then、MutationObserver、queueMicrotask。在当前宏任务的同步代码执行完毕后立即执行，在浏览器渲染（Layout/Paint）之前。会被连续清空（微任务中产生的新微任务也在本轮执行），如果不断产生微任务会阻塞渲染。
宏任务（Macrotask）：setTimeout、setInterval、setImmediate、MessageChannel、I/O、UI 渲染。每次事件循环只执行一个宏任务，执行完后检查微任务队列并清空，然后可能进行渲染，再执行下一个宏任务。不同宏任务之间浏览器有机会渲染。
Vue 选择微任务的核心原因：数据变更 → 触发 watcher 更新队列 → 在微任务中 flush 队列 → DOM 更新 → 浏览器渲染。整个过程在一帧内完成，用户感知不到中间状态。如果用宏任务，数据变更和 DOM 更新之间可能插入一次浏览器渲染，导致闪烁。


1. React 时间切片（Time Slicing）与调度策略
1.1 计算机单处理器调度策略
单处理器在任意时刻只能执行一个任务，操作系统通过调度算法决定哪个进程/线程获得 CPU。核心策略如下：
FCFS（先来先服务）：按到达顺序执行，简单但会导致"护航效应"——一个长任务阻塞后面所有短任务。
SJF（最短作业优先）：优先执行预计耗时最短的任务，平均等待时间最优，但可能导致长任务"饥饿"（永远得不到执行）。
优先级调度（Priority Scheduling）：每个任务有优先级，高优先级先执行。存在饥饿问题，通常通过"老化（Aging）"机制解决——随着等待时间增长，任务优先级逐渐提升。
时间片轮转（Round Robin, RR）：每个任务分配固定时间片（Time Quantum），时间片用完就让出 CPU，排到队尾。响应性好，但时间片大小是关键——太大退化为 FCFS，太小上下文切换开销过高。
多级反馈队列（MLFQ）：多个队列对应不同优先级，新任务进入最高优先级队列，用完时间片后降级到下一级。兼顾了响应性和吞吐量。
1.2 React Scheduler 使用的策略
React 的调度器融合了 优先级调度 + 时间片轮转 + 多级反馈队列 思想：
优先级调度：React 定义了 5 个优先级（源码中对应 Scheduler_ImmediatePriority 到 Scheduler_IdlePriority），每个优先级对应不同的过期时间（timeout）：
ImmediatePriority   → -1ms（立即过期，同步执行）
UserBlockingPriority → 250ms
NormalPriority       → 5000ms
LowPriority          → 10000ms
IdlePriority         → maxSigned31BitInt（永不过期）
时间片轮转：每个工作循环（workLoop）中，React 给任务分配约 5ms 的时间片。执行 Fiber 节点的 reconciliation 时，每处理完一个 Fiber 节点就检查 shouldYield()。如果时间片用完，就把控制权交还给浏览器，让浏览器处理渲染和用户交互，剩余工作在下一帧继续。
类 MLFQ 思想：React 内部维护两个队列——taskQueue（已就绪的任务，按过期时间排序的小顶堆）和 timerQueue（延迟任务，按开始时间排序）。当 timerQueue 中的任务到达开始时间后，会转移到 taskQueue。同时通过过期时间实现"老化"——等待越久的任务越接近过期，优先级实质上越高，避免低优先级任务饥饿。
1.3 为什么不用 requestIdleCallback
React 最初确实考虑过 requestIdleCallback（rIC），但最终自己实现了调度器，原因如下：
兼容性差：rIC 在 Safari 中长期不支持（直到近年才实验性支持），在 IE 中完全不支持。React 需要跨浏览器一致的行为。
触发频率太低：rIC 依赖浏览器"空闲时间"，在持续动画或高频交互场景下，可能长时间不触发回调。实测中 rIC 通常一帧只触发一次，且空闲时间不可控（0~50ms 波动大），导致任务执行时机不可预测。
时间片粒度不可控：rIC 提供的 IdleDeadline.timeRemaining() 最多约 50ms，但实际剩余时间取决于浏览器，React 无法精确控制时间片大小（React 需要稳定的 5ms）。
无法控制优先级：rIC 没有优先级概念，所有任务都是"空闲时执行"。React 需要精细的多级优先级控制。
React 的替代方案：使用 MessageChannel 实现调度。MessageChannel 属于宏任务，触发时机在每一帧渲染之后，不会像 setTimeout(fn, 0) 那样有 4ms 的最低延迟限制（浏览器对嵌套 setTimeout 有最低 4ms 的 clamp），能更高效地利用每一帧的剩余时间。具体做法是：
jsconst channel = new MessageChannel();
const port = channel.port2;
channel.port1.onmessage = performWorkUntilDeadline;
// 请求调度时
port.postMessage(null);

2. useContext / useMemo / useCallback
2.1 useContext
作用：消费 React Context，让组件直接读取最近的 Provider 提供的值，避免 props 逐层传递（prop drilling）。
实现原理：

调用 useContext(MyContext) 时，React 沿着 Fiber 树向上查找最近的 MyContext.Provider，读取其 value。
内部会在当前 Fiber 节点上建立对该 Context 的依赖关系。当 Provider 的 value 变化时，React 会遍历所有消费该 Context 的 Fiber 节点，标记它们需要更新。
对比使用的是 Object.is 浅比较。如果 Provider 的 value 是一个新的对象引用（即使内容相同），所有消费者都会重新渲染。

注意：useContext 本身不提供性能优化，当 Context value 变化时，所有消费该 Context 的组件都会重新渲染，无论它们是否实际用到了变化的部分。
2.2 useMemo
作用：缓存计算结果。只有当依赖项变化时才重新计算，否则直接返回缓存值。
实现原理：

首次渲染（mount）时，执行计算函数，将结果和依赖项存储在 Fiber 节点的 memoizedState 链表中。
后续渲染（update）时，用 Object.is 逐一比较新旧依赖项。如果所有依赖项都相同，直接返回缓存的值；否则重新执行计算函数并更新缓存。

js// 简化的内部逻辑
function useMemo(create, deps) {
  const hook = getOrCreateHook();
  if (hook.memoizedState !== null) {
    const [prevValue, prevDeps] = hook.memoizedState;
    if (areHookInputsEqual(deps, prevDeps)) {
      return prevValue; // 依赖没变，返回缓存
    }
  }
  const value = create();
  hook.memoizedState = [value, deps];
  return value;
}
性能提升：避免在每次渲染中都执行昂贵的计算；配合 React.memo 使用时，稳定的引用可以防止子组件不必要的重新渲染。
2.3 useCallback
作用：缓存函数引用。本质上是 useMemo 的语法糖。
实现原理：

useCallback(fn, deps) 等价于 useMemo(() => fn, deps)。
不缓存函数的执行结果，而是缓存函数本身的引用。依赖项不变时，返回同一个函数引用。

性能提升场景：

将回调函数传给使用 React.memo 包裹的子组件时，稳定的函数引用能避免子组件因为"每次渲染都拿到新函数"而重新渲染。
作为 useEffect 的依赖项时，稳定的引用能避免 effect 不必要的重新执行。

2.4 为什么能提升渲染性能
核心逻辑是：React 的重新渲染是递归的。父组件重新渲染时，默认所有子组件都会重新渲染。性能优化的本质就是减少不必要的重新渲染和重复计算：

useMemo 避免重复的昂贵计算，同时产生稳定的值引用。
useCallback 产生稳定的函数引用。
稳定的引用 + React.memo = 子组件可以通过浅比较 props 来跳过不必要的渲染。
useContext 本身不直接提升性能，但合理拆分 Context（将频繁变化和不常变化的值分开）可以缩小重新渲染的范围。


3. React 中的副作用（Side Effects）
3.1 什么是副作用
在函数式编程的语境下，"纯函数"是给定相同输入总是返回相同输出、且不产生外部可观察变化的函数。副作用就是纯函数之外的一切——任何与外部世界交互的操作：

DOM 操作（增删改节点、修改样式）
数据请求（fetch、XHR）
订阅（WebSocket、事件监听、定时器）
修改外部变量或全局状态
浏览器 API 调用（localStorage、history 等）
日志打印

3.2 React 组件中的副作用
React 的设计理念是：渲染应该是纯的（UI = f(state)），副作用需要被隔离和管理。
useEffect：在渲染完成后（DOM 更新后）异步执行。适用于数据获取、订阅、手动 DOM 操作等。不阻塞浏览器绘制。
jsxuseEffect(() => {
  const subscription = dataSource.subscribe(handleChange);
  return () => subscription.unsubscribe(); // 清理函数
}, [dataSource]);
useLayoutEffect：在 DOM 更新后、浏览器绘制前同步执行。适用于需要读取 DOM 布局信息并同步修改 DOM 的场景（如测量元素尺寸、防止闪烁）。会阻塞浏览器绘制。
3.3 副作用在 React 内部的处理
在 React 的 Fiber 架构中，副作用通过 effectTag（flags） 来标记：

Render 阶段（可中断）：遍历 Fiber 树进行 diff，给需要变更的 Fiber 节点打上标记（如 Placement、Update、Deletion）。这个阶段是纯的，不产生可见的副作用。
Commit 阶段（不可中断）：同步执行所有副作用，分三个子阶段：

Before Mutation：读取 DOM 快照（如 getSnapshotBeforeUpdate）
Mutation：执行 DOM 增删改
Layout：执行 useLayoutEffect，此时 DOM 已更新但浏览器还没绘制
之后浏览器绘制
异步调度执行 useEffect



3.4 React 18+ 的严格模式与副作用
在开发模式下，React 18 的 StrictMode 会故意双重调用 effect（mount → unmount → mount），目的是帮助开发者发现缺少清理逻辑的副作用 bug，为未来的并发特性（如 Offscreen API）做准备。

4. Vue $nextTick 的实现
4.1 核心概念
Vue 的 DOM 更新是异步的。当响应式数据变化时，Vue 不会立即更新 DOM，而是将更新推入一个队列，在当前同步代码执行完毕后的下一个"tick"中批量执行。$nextTick 就是让用户代码在 DOM 更新完成后执行。
4.2 Vue 2 的实现（降级策略）
Vue 2 的 nextTick 实现经历了多次迭代，最终版本采用从微任务到宏任务的降级链：
Promise.then（微任务）
→ MutationObserver（微任务）
→ setImmediate（宏任务，仅 IE/Node）
→ setTimeout(fn, 0)（宏任务，兜底）
优先使用微任务的原因：微任务在当前宏任务结束后、浏览器渲染前执行，能保证 DOM 更新在同一帧内完成，用户不会看到中间状态（不闪烁）。
具体实现细节：
js// Vue 2.6+ 简化逻辑
const callbacks = [];
let pending = false;

function flushCallbacks() {
  pending = false;
  const copies = callbacks.slice(0);
  callbacks.length = 0;
  for (let i = 0; i < copies.length; i++) {
    copies[i]();
  }
}

let timerFunc;

if (typeof Promise !== 'undefined') {
  // 优先 Promise.then（微任务）
  const p = Promise.resolve();
  timerFunc = () => p.then(flushCallbacks);
} else if (typeof MutationObserver !== 'undefined') {
  // MutationObserver 也是微任务
  let counter = 1;
  const observer = new MutationObserver(flushCallbacks);
  const textNode = document.createTextNode(String(counter));
  observer.observe(textNode, { characterData: true });
  timerFunc = () => { counter = (counter + 1) % 2; textNode.data = String(counter); };
} else if (typeof setImmediate !== 'undefined') {
  // setImmediate（宏任务，但比 setTimeout 优先级高）
  timerFunc = () => setImmediate(flushCallbacks);
} else {
  // 兜底 setTimeout
  timerFunc = () => setTimeout(flushCallbacks, 0);
}
Vue 2 历史上的一个坑：2.5.x 版本曾在某些场景下将微任务降级为宏任务（MessageChannel），因为纯微任务在某些边界场景下会导致事件冒泡时序问题（比如在事件回调中修改数据，微任务在冒泡完成前就执行了 DOM 更新）。但 2.6.x 又改回了优先微任务的策略。
4.3 Vue 3 的实现
Vue 3 大幅简化了实现，直接使用 Promise.then，不再做降级：
js// Vue 3 简化逻辑
const resolvedPromise = Promise.resolve();
let currentFlushPromise = null;

function nextTick(fn) {
  const p = currentFlushPromise || resolvedPromise;
  return fn ? p.then(fn) : p;
}
为什么 Vue 3 不需要降级：Vue 3 放弃了对 IE 的支持，所有现代浏览器都原生支持 Promise，不需要 fallback。

4.4 微任务 vs 宏任务的特点对比
微任务（Microtask）：Promise.then、MutationObserver、queueMicrotask。在当前宏任务的同步代码执行完毕后立即执行，在浏览器渲染（Layout/Paint）之前。会被连续清空（微任务中产生的新微任务也在本轮执行），如果不断产生微任务会阻塞渲染。
宏任务（Macrotask）：setTimeout、setInterval、setImmediate、MessageChannel、I/O、UI 渲染。每次事件循环只执行一个宏任务，执行完后检查微任务队列并清空，然后可能进行渲染，再执行下一个宏任务。不同宏任务之间浏览器有机会渲染。
Vue 选择微任务的核心原因：数据变更 → 触发 watcher 更新队列 → 在微任务中 flush 队列 → DOM 更新 → 浏览器渲染。整个过程在一帧内完成，用户感知不到中间状态。如果用宏任务，数据变更和 DOM 更新之间可能插入一次浏览器渲染，导致闪烁。

5. 控制反转（IoC）与依赖注入（DI）
5.1 控制反转（Inversion of Control）
传统模式：代码主动创建和管理依赖。"我需要什么，我自己去 new"。
jsclass UserService {
  constructor() {
    this.db = new MySQLDatabase();     // 紧耦合，主动控制依赖
    this.logger = new FileLogger();
  }
}
控制反转：把"创建和管理依赖"的控制权从组件内部转移到外部容器或框架。"我需要什么，你给我"。
控制反转是一种设计原则，依赖注入是实现它的具体方式之一。其他实现方式还有服务定位器（Service Locator）、事件驱动等。
5.2 依赖注入（Dependency Injection）
依赖注入是控制反转最常见的实现方式，核心思想是：不在组件内部创建依赖，而是从外部注入。
三种注入方式：
js// 1. 构造函数注入（最常用）
class UserService {
  constructor(db, logger) {
    this.db = db;
    this.logger = logger;
  }
}
const service = new UserService(new PostgresDB(), new ConsoleLogger());

// 2. Setter 注入
class UserService {
  setDatabase(db) { this.db = db; }
}

// 3. 接口注入（TypeScript/Java 中通过接口约束）
5.3 前端中的具体场景
React Context 就是一种依赖注入：
jsx// Provider 注入依赖
<ThemeContext.Provider value={theme}>
  <App />
</ThemeContext.Provider>

// 消费者获取依赖，无需关心来源
function Button() {
  const theme = useContext(ThemeContext);
}
Vue 的 provide/inject：
js// 祖先组件提供依赖
provide('apiService', new ApiService());

// 后代组件注入
const api = inject('apiService');
Angular 的 DI 系统：Angular 是前端框架中 DI 最完整的实现，通过装饰器和模块系统实现自动注入：
ts@Injectable({ providedIn: 'root' })
class UserService {
  constructor(private http: HttpClient) {} // 自动注入
}
Node.js 后端（NestJS）：
ts@Injectable()
class CatsService {
  constructor(private catsRepository: CatsRepository) {}
}
5.4 IoC/DI 的核心价值

解耦：组件不依赖具体实现，只依赖抽象（接口）。
可测试：轻松替换依赖为 mock 对象，单元测试更简单。
可扩展：切换实现（如从 MySQL 换到 PostgreSQL）只需修改注入配置，不改业务代码。
关注点分离：组件只关心"做什么"，不关心依赖"怎么创建"。


6. 手写 asyncPool
6.1 题目理解
asyncPool(poolLimit, array, iteratorFn) 实现一个异步并发池：以不超过 poolLimit 的并发数执行 iteratorFn，处理 array 中的每个元素，全部完成后返回结果数组。
6.2 ES7 async/await 实现（推荐）
jsasync function asyncPool(poolLimit, array, iteratorFn) {
  const results = [];          // 存储所有结果
  const executing = new Set(); // 当前正在执行的 Promise 集合

  for (const [index, item] of array.entries()) {
    // 创建当前任务的 Promise
    const p = Promise.resolve().then(() => iteratorFn(item, index, array));

    results.push(p);
    executing.add(p);

    // 任务完成后从执行集合中移除
    const clean = () => executing.delete(p);
    p.then(clean, clean);

    // 如果达到并发上限，等待最快完成的任务
    if (executing.size >= poolLimit) {
      await Promise.race(executing);
    }
  }

  // 等待所有任务完成
  return Promise.all(results);
}
6.3 纯 Promise 实现（不依赖 async/await）
jsfunction asyncPool(poolLimit, array, iteratorFn) {
  let i = 0;
  const results = [];
  const executing = [];

  function enqueue() {
    // 所有任务已入队，返回 Promise.all 等待全部完成
    if (i === array.length) {
      return Promise.all(results);
    }

    const item = array[i];
    const index = i++;

    const p = Promise.resolve()
      .then(() => iteratorFn(item, index, array));

    results.push(p);

    // 任务完成后从执行队列中移除
    const e = p.then(() => executing.splice(executing.indexOf(e), 1));
    executing.push(e);

    // 如果达到并发上限，等待一个完成后再继续入队
    let r = Promise.resolve();
    if (executing.length >= poolLimit) {
      r = Promise.race(executing);
    }

    return r.then(() => enqueue());
  }

  return enqueue();
}
6.4 使用示例
js// 模拟异步任务
const timeout = (ms) => new Promise(resolve =>
  setTimeout(() => resolve(ms), ms)
);

// 同时最多执行 2 个任务
asyncPool(2, [1000, 5000, 3000, 2000], timeout)
  .then(results => {
    console.log(results); // [1000, 5000, 3000, 2000]
  });

// 执行时序：
// t=0:     开始 1000ms, 5000ms       （2个并发）
// t=1000:  1000ms完成, 开始 3000ms    （2个并发）
// t=4000:  3000ms完成, 开始 2000ms    （2个并发）
// t=5000:  5000ms完成                 （1个并发）
// t=6000:  2000ms完成, 全部结束
6.5 关键设计点

为什么用 Promise.race：当并发池满时，需要等待"任意一个"任务完成才能继续放入新任务，Promise.race 正好返回最先完成的 Promise。
为什么要从 executing 中移除已完成的任务：如果不移除，Promise.race 会立即返回（已 resolved 的 Promise 再 race 还是立即 resolve），导致并发限制失效。
错误处理：当前实现中，如果某个任务抛出错误，Promise.all 会直接 reject。如果需要容错（类似 Promise.allSettled），可以在 iteratorFn 外层 catch 错误。
保序：结果数组中的顺序与输入数组一致（通过 results.push 顺序保证），而非任务完成的顺序。
