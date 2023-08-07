#typescript #ts

本地跑的时候要安装`ts-node`和`typescript`
```bash
npx --package typescript tsc --init        
```
以上命令是初始化`tsconfig`

介绍了 `type`和i`nterface`的区别
1. `interface`专门用来声明对象结构的
2. `type`可以表示对象、原始数组、元组、联合类型、交叉类型等
3. `interface`可以通过多次声明相同名称进行合并
4. `type`可以通过`|`或者`&`来组合多个类型
5. `type`也可以为各种形状的类型定义别名
6. `type`可以作为计算属性
	```ts
	type Keys = 'a' | 'b' | 'c';
	type Obj = {
		[K in Keys]: number
	}
	// { a: number, b: number, c: number}
	```

介绍了`object`、`Object`和`{}`的区别
1. `object`是ts2.2版本后引入的类型，表示一个非原始类型的值，除了`number`、`boolean`、`string`、`symbol`、`null`、`undefined`
2. `Object`是js中所有对象的基类
	1. 可以将任何值包括原始类型赋值上去
3. `{}`是描述了没有特定属性的对象，可以代表任何值
实际中建议使用 更明确的接口或者类型别名

定义数组两种形式：
* Array[]
* string[]

定义字面联合类型：
```ts
const arr6: [string, number?, boolean?] = ['linbudu'];
type TupleLength = typeof arr6.length; // 1 | 2| 3
```

定义只读属性:
```ts
interface Car {
	readonly wheel: number;
}
```

### **Utility Types**
`TypeScript`提供了多种实用的工具类型：

#### Record<K, T>
---

* K是一个类型，`string`、`number`
* T与K类型对应的值的类型
```ts
type StringToNumberMap = Record<string, number>;
const ages: StringToNumberMap = {
    alice: 25,
    bob: 30
};

```
示例：
`Record<string, unknown>`、`Record<string, any>`
`unknown` 是一个安全的类型，你不能直接对其进行操作，除非你先检查或断言它的类型
与 `unknown` 不同，`any` 类型不受类型检查的约束，因此它不是类型安全的。你可以对 `any` 类型的变量进行任何操作而不会得到类型错误。

以下可以参考： 
https://www.typescriptlang.org/docs/handbook/utility-types.html#nonnullabletype

```typescript
# Record<K, T>：构建一个对象类型，该对象的属性的键都是 `K` 类型，属性的值都是 `T` 类型
const dictionary: Record<string, string> = {
  key1: "value1",
  key2: "value2"
};


# Partial<T>：让T中所有属性变为可选
type Todo = {
  title: string;
  description: string;
};

const updateTodo: Partial<Todo> = {
  title: "Update title"
};

# Readonly<T>: 使 `T` 中的所有属性变为只读
type Todo = {
  title: string;
};

const myTodo: Readonly<Todo> = {
  title: "Readonly title"
};

# Pick<T, K>: 从 `T` 中选择属性集 `K`
type Todo = {
  title: string;
  description: string;
  completed: boolean;
};

type TodoPreview = Pick<Todo, 'title' | 'completed'>;

# Omit<T, K>: 从 `T` 中移除属性集 `K`
type Todo = {
  title: string;
  description: string;
  completed: boolean;
};

type TodoPreview = Omit<Todo, 'description'>;

# Exclude<T, U>: 从 `T` 中排除那些可以赋值给 `U` 的类型
type T = string | number;
type Result = Exclude<T, string>;  // Result is number

# Extract<T, U>: 从 `T` 中提取那些可以赋值给 `U` 的类型
type T = string | number;
type Result = Extract<T, string>;  // Result is string

# NonNullable<T>: 从 `T` 中排除 `null` 和 `undefined`
type T = string | number | null | undefined;
type Result = NonNullable<T>;  // Result is string | number

# ReturnType<T>: 获取函数类型 `T` 的返回类型
type T = () => string;
type Result = ReturnType<T>;  // Result is string

# InstanceType<T>: 获取构造函数类型 `T` 的实例类型
class MyClass {
  x = 0;
  y = 0;
}

type T = typeof MyClass; // `T` 代表的是 `MyClass` 这个类本身的类型
type Instance = InstanceType<T>;  // Instance is MyClass

```

#### **Declare的含意** 
----

在`typescript`中主要用于声明变量、类型、函数等形状，但并不为其赋实际的值或者实现。
1. 声明模块，用来描述库的类型
	```ts
	declare module "jquery"
	```
2. 声明全局变量
	```ts
	declare const myGlobal: SomeType
	```
实际上使用`declare`仅仅是在告诉`typescript`：这东西是存在的，并且看起来是这样的。实际上的实现和值需要在其他地方提供，当使用`declare`描述模块或库的时候，需要将这些声明放在`.d.ts`文件中，并在`tsconfig.json`中配置以便能找到这些声明文件

#### **字面量类型和联合类型**
---

字面量类型主要包括**字符串字面量类型**、**数字字面量类型**、**布尔字面量类型**和**对象字面量类型**，它们可以直接作为类型标注：
```ts
const str1: "linbudu" = "linbudu599"; // ❌ 错误，只能赋值自身

interface Tmp { 
	user: { vip: true; expires: string; } | { vip: false; promotion: string; }; 
}
const tmp: Tmp = {
	user: {
		vip: false, // 上方Tmp中指定了如果有expires的话vip只能为true
		expires: 'assa' // 对象字面量只能指定已知属性，并且“promotion”不在类型“{ vip: true; expires: string; }”中
	}
}
```

> 通过将vip属性的类型设定为只能基于布尔值字面量的类型声明实现互斥

对象字面量类型其实就是一个对象leix的值，意味着其属性的值必须为该对象类型
```ts
interface Tmp {
  obj: {
    name: "linbudu",
    age: 18
  }
}

const tmp: Tmp = {
  obj: {
    name: "linbudu",
    age: 18
  }
}

```


联合类型代表了一组类型的可用集合，只要最终赋予的类型属于联合类型中的成员的一种，就可以认为符合这个联合类型。
```ts
interface Tmp {
	mixed: boolean | string | number | {} | (() => {}) | (1 | 2)
}
```

需要注意的是，**无论是原始类型还是对象类型的字面量类型，它们的本质都是类型而不是值**。它们在编译时同样会被擦除，同时也是被存储在内存中的类型空间而非值空间

#### 枚举
---
* 语法上枚举值是用=等号来进行赋值的
* 没有声明枚举的值会被默认使用数字枚举，并且从0开始，以1递增
* 如果某一个成员有赋值，该成员需要放在所有未赋值的成员之后
* 赋值也可以加函数求值
	```ts
	const returnNum = () => 100 + 499
	enum Items {
		Foo,
		Baz,
		Bar = 'bar',
		Zoo = returnNum(),
	}
	console.log(Items.Foo, Items.Bar, Items.Baz, Items.Zoo); //0 bar 1 599
	console.log(Items['bar']) // Bar
	// 转译为：
	var returnNum = function () { return 100 + 499; };
	var Items;
	(function (Items) {
		Items[Items["Foo"] = 0] = "Foo";
		Items[Items["Baz"] = 1] = "Baz";
		Items["Bar"] = "bar";
		Items[Items["Zoo"] = returnNum()] = "Zoo";
	})(Items || (Items = {}));
	```
* 枚举和对象的差异在于：对象是单向映射的，只能从键映射到键值，而枚举是双向映射的，可以从值映射到枚举键，需要**⚠️注意**的是：**仅未赋值的成员和赋值了数字的成员可以进行双向枚举，字符串仍然只会进行单次映射**
* 常量枚举只是多了一个`const`，和普通枚举差异主要是访问性和编译产物。常量枚举只能通过枚举成员访问枚举值（无法通过值访问成员）。同时编译产物中不会存在一个额外的辅助对象，对枚举成员的访问会被直接内联替换为枚举的值
	```ts
	// 编译前
	const enum Items2 {
		Baz ='baz'
	}
	const foolVal = Items2.Baz;
	// 编译后
	var foolVal = "baz" /* Items2.Baz */;
	```
* 实际上，常量枚举的表现、编译产物还受到配置项 `--isolatedModules` 以及 `--preserveConstEnums` 等的影响
* 通过ts自带的变量类型推导我们可以看出：const定义的变量的类型为其值的类型，而let定义的则为符合其属性结构的类型，并不会使用字面量类型。