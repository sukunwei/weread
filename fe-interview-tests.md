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

3道错题的核心要点回顾：

第2题：Promise.then 和 queueMicrotask 都是微任务，按注册顺序执行，不存在谁优先的问题，所以是 3 在 4 前面
第4题：关键词"不触发重渲染 + 可变值"只有 useRef 满足，useMemo 是缓存计算结果，语义完全不同
第13题：TypeScript 显式指定泛型 <string> 后，传入 42 会在编译期直接报错，不会到运行时，也不会推断为 any

第三轮踩坑题完整解析
📊 成绩总览
踩坑题正确率60%属于正常水平，这些题的设计目的就是考察细节盲区。

---
第1题：变量提升陷阱 ❌
var a = 1;
function foo() {
  console.log(a);
  var a = 2;
}
foo();
你的思路可能是： 认为 a 会读取外部的 1 或者赋值后的 2。
实际执行过程：
function foo() {
  var a;              // ① 变量提升：声明被提到函数顶部
  console.log(a);     // ② 此时 a 已声明但未赋值 → undefined
  a = 2;              // ③ 赋值在这里才执行
}
var 的变量提升只提升声明，不提升赋值。函数内有 var a，所以 console.log(a) 读的是局部的 a（已声明未赋值），不会去找外部的 a = 1。
🔑 核心陷阱： 函数内 var 声明会"遮蔽"外部同名变量，但赋值不会提升。

---
第2题：数组 push 返回值 ❌
const arr = [1, 2, 3];
const result = arr.push(4);
console.log(result);、
你的思路可能是： 认为 push 没有返回值。
实际情况： push() 返回的是数组的新长度，不是新数组。
const arr = [1, 2, 3];
arr.push(4);       // 返回 4（新长度）
arr.pop();         // 返回 3（被移除的元素）
arr.unshift(0);    // 返回 4（新长度）
arr.shift();       // 返回 0（被移除的元素）
常见数组方法返回值对照：
This content is only supported in a Lark Docs
🔑 核心陷阱： push 返回长度不是数组，很多人凭直觉选A或C。

---
第3题：== 隐式转换 ✅
console.log([] == ![]);
This content is only supported in a Lark Docs
解析： 这是 JS 最反直觉的隐式转换之一，步骤如下：
[] == ![]
→ [] == false        // ![] 先执行，[]是truthy，![] = false
→ [] == 0            // false 转为数字 0
→ "" == 0            // [] 转为原始值，[].toString() = ""
→ 0 == 0             // "" 转为数字 0
→ true
🔑 核心知识： == 会触发一系列类型转换，生产代码永远用 ===。

---
第4题：Object.keys 顺序 ✅
const obj = { 2: 'a', 1: 'b', 'c': 'd', 'a': 'e' };
console.log(Object.keys(obj));
This content is only supported in a Lark Docs
解析： ES2015+ 规范明确了对象属性遍历顺序：
1. 整数索引键（如 '0', '1', '2'）→ 按数值升序
2. 字符串键 → 按插入顺序
3. Symbol 键 → 按插入顺序
所以：先 '1', '2'（整数升序），再 'c', 'a'（插入顺序）。
🔑 核心陷阱： 很多人以为对象属性"无序"，实际上 V8 等现代引擎严格遵循此规范。

---
第5题：箭头函数的 arguments ❌
const foo = () => {
  console.log(arguments);
};
foo(1, 2, 3);
This content is only supported in a Lark Docs
你的思路可能是： 认为箭头函数和普通函数一样有 arguments 对象。
实际情况： 箭头函数没有自己的 arguments 对象。
- 如果外层是普通函数，箭头函数会继承外层的 arguments
- 如果外层是全局作用域（模块环境），arguments 未定义 → ReferenceError
// 普通函数：有 arguments
function foo() { console.log(arguments); }
foo(1, 2, 3); // Arguments(3) [1, 2, 3]

// 箭头函数：没有 arguments
const bar = () => { console.log(arguments); }
bar(1, 2, 3); // ReferenceError: arguments is not defined

// 箭头函数替代方案：使用 rest 参数
const baz = (...args) => { console.log(args); }
baz(1, 2, 3); // [1, 2, 3]（真正的数组）
🔑 箭头函数没有的东西： 没有 this、没有 arguments、没有 prototype、不能 new。

---
第6题：Promise 值穿透 ✅
Promise.resolve(1)
  .then(2)
  .then(console.log);
This content is only supported in a Lark Docs
解析： .then() 的参数必须是函数。如果传入非函数（如数字 2），这个 .then() 会被忽略，值会"穿透"到下一个有效的 .then()。
Promise.resolve(1)
  .then(2)              // 2 不是函数，被忽略，值 1 穿透
  .then(console.log);   // 接收到穿透的 1，打印 1
🔑 核心陷阱： .then(非函数) 不报错，但会导致值穿透，很隐蔽的 bug。

---
第7题：const 声明对象 ✅
const obj = { price: 100 };
obj.price = 200;       // 第一行
obj = { price: 300 };  // 第二行
This content is only supported in a Lark Docs
解析： const 保护的是绑定（变量指向的引用），不是对象内容。
- obj.price = 200 → 修改对象属性 ✅（引用没变）
- obj = { price: 300 } → 重新赋值变量 ❌（改变了引用 → TypeError）
🔑 想要冻结对象内容，使用 Object.freeze(obj)。

---
第8题：Array.from 类数组 ✅
console.log(Array.from({ length: 3 }));
This content is only supported in a Lark Docs
解析： Array.from 接受类数组对象（有 length 属性），按索引读取值。{ length: 3 } 没有 0、1、2 属性，所以每个位置都是 undefined。
Array.from({ length: 3 });                    // [undefined, undefined, undefined]
Array.from({ length: 3 }, (_, i) => i);       // [0, 1, 2]  ← 配合 mapFn
Array.from({ 0: 'a', 1: 'b', length: 2 });   // ['a', 'b']
🔑 Array.from({ length: n }, mapFn) 是创建指定长度数组的常用技巧。

---
第9题：React setState 批处理 ✅
function Counter() {
  const [count, setCount] = useState(0);
  function handleClick() {
    setCount(count + 1);  // 0 + 1
    setCount(count + 1);  // 0 + 1（count 还是闭包中的 0）
    setCount(count + 1);  // 0 + 1
  }
}
This content is only supported in a Lark Docs
解析： 三次 setCount(count + 1) 中的 count 都是当前闭包中的 0，所以三次都是 setCount(0 + 1)，最终 count = 1。
如果要累加，必须使用函数式更新：
function handleClick() {
  setCount(prev => prev + 1);  // 0 → 1
  setCount(prev => prev + 1);  // 1 → 2
  setCount(prev => prev + 1);  // 2 → 3  ✅ 最终为 3
}
🔑 核心陷阱： 闭包捕获的是渲染时的 count 快照，不是最新值。需要基于前值更新时，必须用函数式。

---
第10题：typeof 与暂时性死区 ✅
console.log(typeof x);
let x = 1;
This content is only supported in a Lark Docs
解析： let/const 存在暂时性死区（TDZ），从块开始到声明语句之间，变量不可访问。
// 对比 var 和 let
console.log(typeof a);  // "undefined"（var 提升，typeof 对未声明变量安全）
var a = 1;

console.log(typeof b);  // ReferenceError（let 的 TDZ）
let b = 1;
特别注意：typeof 对未声明的变量返回 "undefined" 是安全的，但对 TDZ 中的变量会报错。这是个重要区别。
🔑 核心陷阱： typeof 并不是永远安全的，TDZ 会让它抛出 ReferenceError。


---
第11题：JSON.stringify 丢失 ❌
const data = {
  a: undefined,
  b: function() {},
  c: Symbol('s'),
  d: NaN,
  e: Infinity
};
console.log(JSON.stringify(data));
This content is only supported in a Lark Docs
你的思路可能是： 认为 NaN 和 Infinity 会保持原样序列化。
实际规则：
This content is only supported in a Lark Docs
所以 a、b、c 被丢弃，d 和 e 变成 null，结果是 {"d":null,"e":null}。
// 注意：在数组中行为不同！
JSON.stringify([undefined, function(){}, Symbol('s')]);
// → '[null,null,null]'  数组中不丢弃，而是转为 null（保持索引）
🔑 核心陷阱： NaN/Infinity 不会直接序列化，而是变成 null。交易所传输价格数据时要特别注意边界值处理。

---
第12题：for...in 的类型陷阱 ❌
const arr = [10, 20, 30];
for (const val in arr) {
  console.log(typeof val);
}
This content is only supported in a Lark Docs
你的思路可能是： 认为 for...in 遍历数组时 val 是索引数字 0, 1, 2。
实际情况： for...in 遍历的是属性名（键），而对象的键永远是字符串。
const arr = [10, 20, 30];

// for...in → 遍历键（字符串）
for (const val in arr) {
  console.log(val, typeof val);  // "0" string, "1" string, "2" string
}

// for...of → 遍历值
for (const val of arr) {
  console.log(val, typeof val);  // 10 number, 20 number, 30 number
}
This content is only supported in a Lark Docs
🔑 核心陷阱： for...in 遍历数组时，索引是字符串 "0" "1" "2"，不是数字。且不建议用 for...in 遍历数组。

---
第13题：解构默认值触发条件 ❌
const { a = 1, b = 2 } = { a: undefined, b: null };
console.log(a, b);
This content is only supported in a Lark Docs
你的思路可能是： 认为 undefined 和 null 都会触发默认值。
实际规则： 解构默认值只在值为 undefined 时触发，null 不触发。
const { a = 1 } = { a: undefined };  // a = 1     ✅ 触发默认值
const { b = 2 } = { b: null };       // b = null  ❌ 不触发，null 是有效值
const { c = 3 } = { c: 0 };          // c = 0     ❌ 不触发
const { d = 4 } = { c: '' };         // d = 4     ✅ 触发（属性不存在 = undefined）
const { e = 5 } = { e: false };      // e = false ❌ 不触发
🔑 核心陷阱： null 和 undefined 在很多场景下行为不同。默认值、??运算符、可选链都只对 undefined/null 特殊处理，但具体行为各有差异。
补充：?? vs || 的区别：
0 || 'default'        // 'default'  ← || 把 0 当 falsy
0 ?? 'default'        // 0          ← ?? 只管 null/undefined

'' || 'default'       // 'default'
'' ?? 'default'       // ''

null ?? 'default'     // 'default'  ← 两者对 null 行为一致
null || 'default'     // 'default'

---
第14题：async 函数返回值 ❌
async function foo() {
  return 'hello';
}
const result = foo();
console.log(result);
This content is only supported in a Lark Docs
你的思路可能是： 认为 console.log 执行时 Promise 已经 fulfilled。
实际情况： 虽然 foo() 内部的 return 'hello' 是同步的，但 async 函数返回的 Promise 的 resolve 回调是作为微任务执行的。console.log(result) 是同步代码，在微任务之前执行，所以此时 Promise 仍然是 pending 状态。
async function foo() {
  return 'hello';
}

const result = foo();
console.log(result);           // Promise { <pending> }

// 要拿到值，必须 await 或 .then
const value = await foo();
console.log(value);            // 'hello'

foo().then(v => console.log(v)); // 'hello'
🔑 核心陷阱： async 函数永远返回 Promise，即使 return 的是同步值，在同步代码中打印时也是 pending。Promise { 'hello' } 是在调试器中展开后看到的，console.log 同步执行时还没 resolve。

---
第15题：事件冒泡与 stopPropagation ✅
parent.addEventListener('click', () => console.log('parent'));
child.addEventListener('click', (e) => {
  e.stopPropagation();
  console.log('child');
});
This content is only supported in a Lark Docs
解析： stopPropagation() 阻止事件继续向上冒泡。点击 child 时：
1. child 的事件处理器执行 → 打印 child
2. 冒泡被阻止 → parent 的处理器不执行
注意区分：
- stopPropagation() → 阻止冒泡，当前元素上的其他监听器仍执行
- stopImmediatePropagation() → 阻止冒泡 + 当前元素上的后续监听器也不执行

---
第16题：Map 键的比较 ✅
const map = new Map();
map.set({}, 'a');
map.set({}, 'b');
console.log(map.size);
This content is only supported in a Lark Docs
解析： Map 的键比较使用 SameValueZero 算法（类似 ===）。两个 {} 是不同的对象引用，所以是两个不同的键。
const obj = {};
map.set(obj, 'a');
map.set(obj, 'b');  // 同一个引用，覆盖
console.log(map.size); // 1

// 特殊情况：NaN
map.set(NaN, 'x');
map.set(NaN, 'y');  // 覆盖（Map中 NaN === NaN）
console.log(map.size); // 1
🔑 对象作为 Map 键时，必须用同一引用才能取到值。

---
第17题：CSS 选择器优先级 ✅
This content is only supported in a Lark Docs
解析： 优先级计算规则：(ID数, Class数, Element数)，从左到右依次比较。
- A: 1个ID + 1个class + 1个元素 = (1,1,1)
- C: 1个ID + 1个class + 2个元素 = (1,1,2)
A 和 C 的 ID 数和 Class 数相同，但 C 多一个元素选择器，所以 C > A。
🔑 记忆口诀： !important > 内联style > ID > Class/属性/伪类 > 元素/伪元素

---
第18题：WeakRef ✅
let obj = { data: 'important' };
const weakRef = new WeakRef(obj);
obj = null;
This content is only supported in a Lark Docs
解析： WeakRef 不阻止垃圾回收。obj = null 后，原对象失去强引用，可能被GC回收。但GC的时机是不确定的：
- GC 还没执行 → weakRef.deref() 返回原对象
- GC 已执行 → weakRef.deref() 返回 undefined
🔑 WeakRef 和 FinalizationRegistry 一般用于缓存场景，不要依赖其确定性行为。

---
第19题：模板字面量标签函数 ❌
function tag(strings, ...values) {
  console.log(strings.length, values.length);
}
tag`a${1}b${2}c`;
This content is only supported in a Lark Docs
你的思路可能是： 认为 strings 包含所有字符（5个字符 a, 1, b, 2, c）。
实际规则： 模板字面量被插值表达式分割成字符串片段：
tag`a${1}b${2}c`

strings = ['a', 'b', 'c']     // 3个静态字符串片段
values  = [1, 2]               // 2个插值表达式
关键公式：strings.length = values.length + 1（永远多一个）
原因：N 个插值将模板分成 N+1 段，就像 2 刀切面包切出 3 片。
tag`${1}${2}`
// strings = ['', '', '']  → 3个（两端空串 + 中间空串）
// values  = [1, 2]        → 2个

tag`hello`
// strings = ['hello']  → 1个
// values  = []         → 0个
🔑 核心陷阱： strings 不是逐字符拆分，而是按 ${} 位置切割成片段。

---
第20题：可选链与 nullish 合并 ✅
const user = { profile: { name: '' } };
const result = user.profile?.name ?? 'Anonymous';
console.log(result);
This content is only supported in a Lark Docs
解析：
- user.profile?.name → ''（空字符串，不是 null/undefined）
- '' ?? 'Anonymous' → ''（?? 只在 null/undefined 时使用右侧值）
如果用 || 结果就不同了：
'' ?? 'Anonymous'   // ''          ← ?? 不把空串当"空"
'' || 'Anonymous'   // 'Anonymous' ← || 把空串当 falsy
🔑 交易所场景中，价格为 0 时用 || 会被误替换为默认值，必须用 ??。

---
📋 全部错题汇总
This content is only supported in a Lark Docs

---
📊 三轮总成绩
This content is only supported in a Lark Docs
整体评价：基础扎实，进阶知识掌握良好，在 JS 隐式行为和边界情况上有提升空间。建议重点复习：类型转换规则、数组方法返回值、箭头函数限制、解构/??/|| 的 null vs undefined 区别。





