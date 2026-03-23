第1题：JavaScript 基础
javascriptconsole.log(typeof null === typeof undefined);
A. true　　B. false　　C. TypeError　　D. undefined

第2题：事件循环
javascriptconsole.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
queueMicrotask(() => console.log('4'));
console.log('5');
输出顺序是？
A. 1 5 3 4 2　　B. 1 5 2 3 4　　C. 1 3 4 5 2　　D. 1 5 4 3 2

第3题：TypeScript
以下哪个类型工具可以将对象类型的所有属性变为可选？
A. Readonly<T>　　B. Partial<T>　　C. Required<T>　　D. Pick<T, K>

第4题：React Hooks
在React中，以下哪个Hook用于在不触发重新渲染的情况下保存一个可变值？
A. useState　　B. useMemo　　C. useRef　　D. useCallback

第5题：CSS 布局
要实现一个元素在其父容器中水平垂直居中，以下哪种 Flexbox 组合是正确的？
A. display: flex; justify-content: center; align-items: center;
B. display: flex; justify-content: space-between; align-items: flex-start;
C. display: flex; flex-direction: column; align-items: flex-end;
D. display: flex; justify-content: flex-start; align-items: center;

第6题：WebSocket
在交易所实时行情推送场景中，WebSocket相比HTTP轮询的核心优势是？
A. 安全性更高
B. 全双工通信，延迟更低
C. 兼容性更好
D. 不需要服务器支持

第7题：闭包
javascriptfor (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
输出结果是？
A. 0 1 2　　B. 3 3 3　　C. undefined undefined undefined　　D. 0 0 0

第8题：React 性能优化
以下哪个方法可以避免React子组件在父组件重新渲染时进行不必要的渲染？
A. useEffect　　B. React.memo　　C. useContext　　D. ReactDOM.render

第9题：HTTP 缓存
浏览器强缓存命中时，HTTP状态码是？
A. 200　　B. 301　　C. 304　　D. 不发送请求，直接读取缓存（状态码200 from cache）

第10题：XSS 防御
以下哪种做法最能有效防御存储型XSS攻击？
A. 仅在前端对用户输入做长度限制
B. 对用户输入在服务端进行HTML转义，并在前端使用textContent而非innerHTML
C. 使用HTTP GET代替POST
D. 启用CORS

第11题：Promise
javascriptPromise.all([
  Promise.resolve(1),
  Promise.reject('error'),
  Promise.resolve(3)
]).then(res => console.log(res))
  .catch(err => console.log(err));
输出是？
A. [1, 'error', 3]　　B. error　　C. [1, undefined, 3]　　D. TypeError

第12题：虚拟列表
交易所订单簿（Order Book）展示数万条数据时，使用虚拟列表的核心原理是？
A. 将所有DOM一次性渲染，通过CSS隐藏不可见部分
B. 只渲染可视区域内的DOM节点，滚动时动态替换
C. 使用Canvas绘制全部内容
D. 利用Web Worker在后台渲染DOM

第13题：TypeScript 泛型
typescriptfunction identity<T>(arg: T): T {
  return arg;
}
const result = identity<string>(42);
以上代码会发生什么？
A. 正常运行，result为42
B. 编译错误，42不能赋值给string类型
C. 运行时抛出TypeError
D. result类型为any

第14题：跨域
以下哪个HTTP响应头用于允许跨域请求？
A. Content-Type　　B. Access-Control-Allow-Origin　　C. X-Frame-Options　　D. Cache-Control

第15题：React Fiber
React Fiber架构的核心目标是？
A. 替代Virtual DOM
B. 实现增量渲染，使渲染任务可中断和恢复
C. 完全消除重新渲染
D. 将React改为编译型框架

第16题：BigInt / 精度
交易所前端处理价格 0.1 + 0.2 时，以下哪个结果是JavaScript给出的？
A. 0.3　　B. 0.30000000000000004　　C. NaN　　D. 0.29999999999999999

第17题：Web Worker
Web Worker的主要限制是？
A. 无法执行异步操作
B. 无法直接访问DOM
C. 只能运行同步代码
D. 不能与主线程通信

第18题：状态管理
在大型交易所前端项目中，以下哪个方案最适合管理高频更新的全局行情数据？
A. 将所有数据放在React组件的useState中逐层props传递
B. 使用Context + useReducer，每次行情变动触发全局重渲染
C. 使用外部状态库（如Zustand/Jotai）配合选择性订阅，避免无关组件重渲染
D. 将数据存入localStorage，组件定时轮询读取

第19题：Content Security Policy (CSP)
CSP的主要作用是？
A. 加速页面加载
B. 限制页面可加载的资源来源，防止XSS等注入攻击
C. 压缩HTTP响应体
D. 控制浏览器缓存策略

第20题：Tree Shaking
Webpack/Vite中Tree Shaking能生效的前提是？
A. 代码使用CommonJS模块（require/module.exports）
B. 代码使用ES Module（import/export），且无副作用
C. 开启source map
D. 使用动态import import()

以上就是全部20题，请把你的答案发给我（如 1-B, 2-A, 3-B...），我会统一批改并详细解析！
