### **为什么是使用 render 函数，而不是熟悉的 template 语法编写呢？**

因为 Vue3.0 默认的包是不支持模板编译功能的。也就是说， template 语法现在还不能用。在 Vue3.0 中编译功能推荐在构建阶段完成，而不是放到浏览器中运行。如果希望在浏览器中的话，可以选择 ./node_modules/vue/dist/vue.global.js 这个包。
![[Pasted image 20230726225125.png]]
Vue的SFC是需要编译后才能运行的，因此得装vite-plugin-vue类似的插件，需要在编译阶段转换为ts（渲染函数）的形式才可以运行，vue插件提供了模板的编译，还支持vue单文件（SFC）组件的编译

---

### **在引用 .vue 模块时候，编辑器中 import 语句会有红色的警告**

这是因为Typescript 默认是不支持 `.vue` 类型的模块的。可以通过添加一个模块的类型定义来解决这个问题。
```js
// src/shims-vue.d.ts
declare module "*.vue" { import { DefineComponent } from "vue"; const component: DefineComponent<{}, {}, any>; export default component; }
```

---

### **为什么组合函数写法中要多写一个包裹函数？**

```javascript
// userStore.js 
export const useUserStore = defineStore(...) 
export function useUserStoreWrapper() { return useUserStore() }
```
主要原因是组合函数的返回值类型问题，在setup中使用组合函数，会直接返回ref，而在其他地方调用的话仅会返回具体的值而不是ref类型
以下为示例：
```javascript
// 1. useUserStore内部包含了ref类型的状态:
const user = ref({})
// 2. 它的返回值是:
return { user } // 返回ref类型

// 3. 在组件中使用时,可以直接接收这个ref:
// 组件setup 
const {user} = useUserStore() // 直接是ref

// 4. 但是如果在另一个组合函数中使用:
// 组合函数
function useAuth() {
  const {user} = useUserStore() // 错误!无法直接得到ref
}
// 5. 这是因为组合函数只能返回普通对象,不能返回ref对象:
// 正确
function useAuth() {
  return { user: '123' } 
}

// 错误
function useAuth() {
  return { user: ref('123') } // 无法返回ref
}

// 6. 所以需要对ref进行解包:
// 组合函数
function useAuth() {
  const {user} = useUserStoreWrapper() // 解包后获得值
  return { user }
}
```
因为组合函数对返回值类型的限制,无法直接返回ref,只能通过解包来使用其它组合函数的ref。这就是组合函数中需要使用包裹层的主要原因

