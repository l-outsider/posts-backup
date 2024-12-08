# React 项目规范

本人曾高强度参与 React 相关内容的开发，因此深知：

了解 React 开发的基础是一回事，如何将 React 优雅的运用到大型项目的工程化开发中又是另一回事。

如今编写此篇文档，意为记录一些最佳实践以指导 React 开发，请酌情参考，欢迎讨论。

本文将从以下几点入手：

1. react 目录结构的最佳实践。
2. react 项目配置的最佳实践。
3. typescript + React：如何将类型系统融入 React 以寻求更低的 error 率。
4. css + react：如何优雅的在 React 中使用 css、less、sass 等，避免 css 命名冲突的问题。
5. react 中 hook 的最佳实践。

## 1 react 目录结构的最佳实践 <a href="#id-1react-mu-lu-jie-gou-de-zui-jia-shi-jian" id="id-1react-mu-lu-jie-gou-de-zui-jia-shi-jian"></a>

对于大多的 react 项目，目录结构一般有以下内容：

```
src: 所有源代码的顶级目录。
  components: 存放所有React组件。
  pages 或 views: 存放页面级别的组件或路由对应的组件。
  layout: 一般用于存放全局的布局组件 例如Header、Footer等
  utils: 实用工具函数和类。
  services: API调用和数据处理逻辑。
  assets: 静态资源如图片、字体等。
  styles: 全局样式或共享样式。
  store 或 redux: Redux或状态管理相关的文件。
  context: 用于存放react context
  hooks: 自定义Hooks。
  tests: 单元测试和集成测试。
```

细节上，下面展示了一种比较好的最佳实践，用于组织目录结构：

```
chat-uikit-react
├── src/
│   ├── components/                        # 所有组件目录
│   │   └── ComponentName/                 # 组件目录 命名与组件一致 应该是一个名词 (使用 PascalCase 命名法)
│   │       ├── index.ts                   # Barrel export file
│   │       ├── ComponentName.tsx          # 主组件
│   │       ├── ComponentName.stories.tsx  # Storybook 文件 (命名与组件一致)
│   │       ├── ComponentName.scss         # 样式文件 (命名与组件一致)
│   │       └── SubComponentName           # 不对外暴露的子组件 结构与上面一致
│   │           ├── index.ts
│   │           ├── SubComponentName.tsx
│   │           ├── SubComponentName.stories.tsx
│   │           └── SubComponentName.scss
│   │
│   ├── hooks/                      # 自定义 React Hook 文件夹
│   │   ├── useHookName.ts          # Hook 文件名应该使用 'use' 作为前缀
│   │   └── index.ts                # Barrel export file
│   │
│   ├── contexts/                   # React contexts 文件夹
│   │   ├── ContextNameContext.tsx  # Context 文件 (PascalCase 命名法) 使用 'Context' 作为后缀
│   │   └── index.ts
│   │
│   ├── constants/                  # Constants 文件夹
│   │   ├── constantName.ts         # Constant file (camelCase 命名法)
│   │   └── index.ts
│   │
│   ├── utils/                      # Utilities 文件夹
│   │   ├── utilName.ts             # Utility file (camelCase 命名法)
│   │   └── index.ts
│   │
│   ├── types/                      # 全局的 TypeScript types 文件夹
│   │   ├── typeName.ts             # Type definition file (camelCase 命名法)
│   │   └── index.ts
│   │
│   └── index.ts    

命名规范：
  组件目录：大驼峰命名（如：ButtonGroup）
  组件文件：大驼峰命名（如：Button.tsx）
  子组件文件：大驼峰命名（如：ButtonIcon.tsx）
  Hook 文件：小驼峰命名且以 'use' 为前缀（如：useMessage.ts）
  工具文件：小驼峰命名（如：messageHelper.ts）
  样式文件：与组件同名，使用 .scss 扩展名
  类型文件：小驼峰命名（如：messageTypes.ts）
  Story 文件：与组件同名，添加 .stories.tsx 扩展名
  Context 文件：大驼峰命名且以 Context 为后缀（如：ThemeContext.tsx）
  常量文件：小驼峰命名（如：messageConstants.ts）
  
文件组织规则：
  每个组件都应该有自己的目录
  所有与组件相关的文件都应该在组件的目录下
  不对外暴露子组件应该与其父组件在同一目录下，如果向外暴露应该平铺在 src/components 下
  每个目录都应该有一个 index.ts 文件用于导出
  全局类型应该放在 types 目录下
  共享工具函数应该放在 utils 目录下
```

组件的入口文件应该统一导出组件

```
export { default } from './Button';
```

这样父组件可以更清爽、便捷的使用。

```
import Button from '@/components/Button';
// 而不是
import Button from '@/components/Button/Button';
```

## 2 react 项目配置的最佳实践 <a href="#id-2react-xiang-mu-pei-zhi-de-zui-jia-shi-jian" id="id-2react-xiang-mu-pei-zhi-de-zui-jia-shi-jian"></a>

对于 tsconfig 我认为有一个很重要的配置是要开启的："exactOptionalPropertyTypes": true。它可以保证类型的准确性，从而在编译阶段减少错误。

除此之外还有一些例如：

* target: 设置目标JavaScript版本，如es6。
* jsx: 设置JSX的编译方式，如react-jsx。
* strict: 开启严格类型检查。
* esModuleInterop: 允许在CommonJS模块中使用import语句。
* skipLibCheck: 跳过库文件的类型检查，可以加快编译速度。

更多可参考 [tsconfig-pei-zhi-xiang-jie.md](../builder/tsconfig-pei-zhi-xiang-jie.md "mention")

## 3 如何优雅地编写 react(tsx) 组件 <a href="#id-3-ru-he-you-ya-di-bian-xie-reacttsx-zu-jian" id="id-3-ru-he-you-ya-di-bian-xie-reacttsx-zu-jian"></a>

这里主要解释函数式组件的大致构造，以及书写组件的顺序。

```
// 1. 导入顺序：React相关 -> 第三方库 -> 内部依赖 -> 类型文件 -> 样式文件
import React, { useState } from 'react';
import cs from 'classnames';
import OtherComponent from './OtherComponent';
import safeParse from '../../utils';
import type { IThemeContext } from '../../contexts/ThemeContext';

import classes from './Component.module.scss';

// 2. 当前组件类型定义应该在当前文件内 增强代码的易读性
// 必选项在前，可选项在后
// 属性在前 方法在后
// 命名为：I + 组件名 + Props
interface IGreeingProps {
  // 3.接口类型书写标准指南
  // 要注意可选参数这4种写法的区别 这和'exactOptionalPropertyTypes'属性息息相关
  // 必须传age
  age: number;
  // 必须传age但值可以为undefined
  age: number | undefined;
  // 要么传number 要么不传
  age?: number;
  // 可传可不传且可以传入undeinfed
  age?: number | undfiend;
  // 定义对象禁止的两种方式
  payload: object;
  payload: {};
  // 定义一个未知对象的正确方式
  paylod: Record<string, any>;
  // 永远不要这样定义函数
  onSomething: Function;
  // 简单函数的正确写法
  onClick: () => void;
  // 输入事件的正确写法
  // 事件类型查询表
  // https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/forms_and_events/#list-of-event-types
  onChange: (event: React.ChangeEvent<HTMLInputElement>) => void;
  // 传入setState的正确写法
  setState: React.Dispatch<React.SetStateAction<YourType>>;
  // 样式
  styles?: React.CSSProperties;
  // class
  classNames?: string;
  // 传入 children 的建议写法 注意ReactNode和ReactElement的区别
  children?: React.ReactNode;
  // styles/classNames/children 三大通用属性放最后
}

type YourCustsomType = /* 定义类型应该使用 Type 结尾 */

// 4.不要在函数声明时就解构 props
// 5.过去有一种写法是 const Greeting: React.FC<IGreeingProps> = () => {}; React.FC or React.FunctionComponent 写法已经被废弃
const Greeting = (props: IGreeingProps) {

  // 6.首先解构props 并建议使用es语法给定默认值 而不是使用旧的defaultProps
  const {
    name = 'bob',
    age = 20,
    ...restProps,
  } = props;

  // 7.解构完成后，优先声明state并给定类型和初始值
  const [list, setList] = useState<string[]>();
  // 如果初始值没有，建议请声明联合null ｜ undefined 类型
  const [list, setList] = useState<string[] | null>(null);
  // 如果是对象类型可以强制声明 前提是你知道你的代码会在第一时间赋值
  const [user, setUser] = useState<User>({} as User);
  // 同理可以不声明联合null类型 用感叹号表示只在初始化时为null
  const [list, setList] = useState<string[]>(null!);

  // 7.1 自定义 Hook
  
  // 8.useContext

  // 9.state声明完成后，使用ref引用dom节点
  // 尽量使用明确的dom类型 而不是HTMLElement或Element
  // 建议使用null作为初始值 使用时判断divRef.current是否存在 否则抛出异常
  const divRef = useRef<HTMLDivElement>(null);
  // 10.声明可变ref
  const intervalRef = useRef<number | null>(null);

  // 11.声明useMemo
  
  // 12.声明useEffectLayout 注意结尾分号
  useEffectLayout(() => {
    if (!divRef.current) {
      throw new Error('divRef.current is null');
    }
    const rect = divRef.current.getBoundingClientRect();
  }, []);
  
  // 13.声明useEffect 注意结尾分号
  useEffect(() => {
    if (!divRef.current) {
      throw new Error('divRef.current is null');
    }
    intervalRef.current = setInterval(() => {
      divRef.current.style.backgroundColor = 'red';
    }, 1000);
    return () => {
      clearInterval(intervalRef.current);
    };
  }, []);

  // 14.声明useCallback函数

  // 15.声明自定义函数

  // 16.return值
}

导出应该在文件结尾统一导出，不应该在定义时导出
export default Greeting;
export type {
	/* */ 
}
```

Why children use ReactNode type?

[https://www.arahansen.com/how-children-types-work-in-react-18-and-typescript-4/](https://www.arahansen.com/how-children-types-work-in-react-18-and-typescript-4/)

### 组件模板（可复制粘贴） <a href="#zu-jian-mu-ban-ke-fu-zhi-nian-tie" id="zu-jian-mu-ban-ke-fu-zhi-nian-tie"></a>

```
import { useState } from 'react';
import cs from 'classnames';
import classes from './ComponentName.module.scss';

interface IComponentNameProps {
  prop1: string;
  prop2?: number;
  onEvent1: () => {};
  onEvent2?: () => {};
  classNames?: string;
  styles?: React.CSSProperties;
  children?: React.ReactNode;
}

const ComponentName = (props: IComponentNameProps) => {
  
  const {
    prop1,
    prop2 = defaultValue,
    onEvent1,
    ...restProps
  } = props;
  
  const [state, setState] = useState<Type>();
  
  const [_] = useCustomHook();
  
  const context = useCustomContext();
  
  const ref = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
  	/** */
  }, []);

  return (
    <div 
      className={cs(
        'component-name',  // 基础类名
        {
          'component-name--disabled': disabled,  // 条件类名
        },
        className  // 自定义类名放最后
      )}
      onClick={}
      {...restProps}
    >
      {children}
    </div>
  );
};

export type {
  IComponentNameProps
}
export default ComponentName;
```

### Props 定义命名规范 <a href="#props-ding-yi-ming-ming-gui-fan" id="props-ding-yi-ming-ming-gui-fan"></a>

其实 Props 的定义，对于组件库来说就是暴露给用户的接口，因此 Props 的命名、定义需要使用规范来约束。因为它们直接影响到组件的可用性和可读性。以下是一些通用的 Props 命名规范建议：

布尔值 Props：

* 可以直观的描述组件状态，如果不可以直观描述，使用 is-、has- 作为开头，或者使用 -able(ible)作为后缀。
* 例如：Button.fullWidth（按钮是否充满父容器）、Button.disabled（按钮是否禁用）、Input.autoFocus（Input 框是否自动聚焦）、DropdownMenu.isScrollable。
* 备注：其实这里我一开始认为应该无脑使用 is、has 来定义布尔类型的接口，但在我取经了 Web 界最流行的两个框架（material ui/antd）以及 Native UI 框架（TDesign）发现并不如此。

事件处理函数：

* 使用 on 前缀来表示事件处理函数。
* 例如：onClick、onChange、onSubmit、onLoad。
* 备注：对于基础组件，请使用 onClick、onChange 等直观的命名，对于较为复杂的组件事件，请参考回调函数。
* 定义：HTML 原生事件如 img.onLoad 或专门用于响应用户交互的事件。

回调函数：

* 使用 on 前缀来表示回调函数。
* 例如：onComplete、onVisibleChange、onXXXClick、onBeforeOpen。
* 备注：如何区分事件处理函数和回调，我认为 HTML 原生的事件或用户交互事件可以分类为事件处理，基于业务的分类为回调函数，例如 onVisibleChange，onSuccess。

状态或模式：

* 使用 mode、type、status 等后缀来表示状态或模式。
* 例如：displayMode、buttonType、statusType。

数据或内容：

* 使用 data、content、items 等后缀来表示数据或内容。
* 例如：userData、listItems、contentText。

样式相关：

* 使用 styles、classNames 等后缀来表示样式相关的 Props。
* 例如：buttonStyle、containerClassName。

尺寸和布局：

* 使用 size、width、height、margin、padding 等来表示尺寸和布局。

显隐控制：

* 使用 visible 来表示显隐控制。
* 特殊说明：**对于弹出类的组件，必须使用 open 来控制显隐。**

标识符：

* 使用 id、key、name 等来表示标识符。
* 例如：userId、itemKey、fieldName。

默认值：

* 使用 default 前缀来表示默认值。
* 例如：defaultValue、defaultChecked。

## 4 如何优雅地在 react 组件中使用 css <a href="#id-4-ru-he-you-ya-di-zai-react-zu-jian-zhong-shi-yong-css" id="id-4-ru-he-you-ya-di-zai-react-zu-jian-zhong-shi-yong-css"></a>

这里不得不提到两个东西

* classnames [github 地址](https://github.com/JedWatson/classnames#readme)
* css 使用 BEM 规范
* css module

### 4.1 classnames 的使用 <a href="#id-4.1classnames-de-shi-yong" id="id-4.1classnames-de-shi-yong"></a>

```
import React from 'react';
import cs from 'classnames';
import styles from './styles.module.scss';

interface IProps {
  classNames?: string;
}

const MyButton = (props: IProps) => {
  const { classNames } = props;
  const [isActive, setIsActive] = useState(false);

  return (
    // class='app button bar active item-btn'
    <button
      className={cs(classNames, styles.button, 'bar', undefined, {
        active: isActive,
      })}
      onClick={() => setIsActive(!isActive)}
    >
      Click me!
    </button>
  );
};

// 父组件
import styles from './index.module.scss';
<MyButton className={styles.app}></MyButton>
```

### 4.2 BEM 规范 <a href="#id-4.2bem-gui-fan" id="id-4.2bem-gui-fan"></a>

BEM（Block, Element, Modifier）是一种命名约定，用于编写可维护的 CSS。它通过将类名分为块、元素和修饰符三部分，帮助开发者创建结构清晰、可复用的代码。以下是 BEM 的基本概念：

1 Block（块）：

* 独立的功能模块，通常是一个组件的根元素。
* 例如：.button、.card。

2 Element（元素）：

* 块的组成部分，不能单独存在，必须依赖于块，如果脱离会产生歧义。
* 使用双下划线连接块和元素的名称。
* 例如：.button\_\_icon、.card\_\_title。

3 Modifier（修饰符）：

* 用于定义块或元素的不同状态或外观。
* 使用双横线连接块/元素和修饰符的名称。
* 例如：.button--primary、.card\_\_title--large。

BEM 命名示例：

```
<div class="button button--primary">
  <span class="button__icon"></span>
  <span class="button__text">Click me</span>
</div>
```

BEM 的优点：

* 可读性：类名清晰地描述了元素的层级和状态。
* 可复用性：通过修饰符可以轻松创建不同的样式变体。
* 可维护性：结构化的命名方式使得样式表更易于维护。
* 避免冲突：通过命名空间的方式减少了样式冲突的可能性。

BEM 的使用规则：

* 块、元素、修饰符的名称应简洁明了。
* 块和元素的名称之间用双下划线连接。
* 块/元素和修饰符的名称之间用双横线连接。
* 不要在修饰符中使用元素名，修饰符直接作用于块或元素。

BEM 是一种强大的 CSS 组织方法，特别适合大型项目和团队协作。通过 BEM，开发者可以更好地管理和扩展样式代码。

### 4.3 使用 css/scss module



### 4.4 如何在 scss 中使用自定义颜色 <a href="#id-4.3-ru-he-zai-scss-zhong-shi-yong-zi-ding-yi-yan-se" id="id-4.3-ru-he-zai-scss-zhong-shi-yong-zi-ding-yi-yan-se"></a>

首先项目需要提供基于 mixin 的写法。

```
// 引入scss mixin
@use "@tencentcloud/uikit-base-component-react/dist/styles/util" as *;

.div {
  // 与自定义颜色无关的内容
  height: 80px;
  width: 80px;
  display: flex;

  @include theme() {
    // 引入预定义好的颜色命名 会根据当前主题自动切换
    background-color: get(bg-primary);
    color: get(text-secondary);
  }
}
```

## 补充阅读 <a href="#bu-chong-yue-du" id="bu-chong-yue-du"></a>

### 如何优雅地在 react 组件中使用 各种 hook <a href="#ru-he-you-ya-di-zai-react-zu-jian-zhong-shi-yong-ge-zhong-hook" id="ru-he-you-ya-di-zai-react-zu-jian-zhong-shi-yong-ge-zhong-hook"></a>

灵魂提问：

1. 到底什么是 Hook？
2. Hook 和普通函数有什么区别？（业务代码中，如何判断你写的东西应该作为一个 Hook 还是普通函数？）

#### 5.1 到底什么是 Hook？ <a href="#id-5.1-dao-di-shen-me-shi-hook" id="id-5.1-dao-di-shen-me-shi-hook"></a>

Hook 英文解释：

Hook 是一个弯曲的金属或塑料片，for **catching** or **holding** things，or for hanging things up.

说简单点就是把一个东西吸附、抓取到另一件物体上。

那么编程中的 Hook 到底是什么呢？先按下不表。

### 5.2 Hook 发展的历史 <a href="#id-5.2hook-fa-zhan-de-li-shi" id="id-5.2hook-fa-zhan-de-li-shi"></a>

要想回答上面的问题，我们得先了解下 React 的发展历史。

早期的 React 组件都是类组件，类组件有生命周期，有状态，有 this，但是函数组件没有，函数组件只能通过 props 传递数据，函数组件不能有状态，不能有生命周期。

#### 5.2.1 早期的 React Class 组件 <a href="#id-5.2.1-zao-qi-de-reactclass-zu-jian" id="id-5.2.1-zao-qi-de-reactclass-zu-jian"></a>

先来看一个早期的 Class 组件：

```
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class ExampleClassComponent extends Component {
  // 构造函数
  constructor(props) {
    super(props);
    // 对应 useState
    // 状态管理
    this.state = {
      count: 0,
      theme: 'light',
    };

    // 对应 useRef
    // 用于保存一个可变的值，不会引起组件重新渲染
    this.inputRef = React.createRef();
  }

  // 对应 useEffect
  // 生命周期方法：组件挂载后执行
  componentDidMount() {
    console.log('Component did mount');
    document.title = `You clicked ${this.state.count} times`;

    // 模拟异步数据获取
    fetch('<https://api.example.com/data')>
      .then((response) => response.json())
      .then((data) => {
        console.log('Data fetched:', data);
      });

    // 添加事件监听器
    window.addEventListener('resize', this.handleResize);

    // 对应 useEffect 的清理函数
    // 在组件卸载前移除事件监听器
    this.cleanup = () => {
      window.removeEventListener('resize', this.handleResize);
    };

    // 对应 useRef
    // 自动聚焦输入框
    this.inputRef.current.focus();
  }

  // 对应 useEffect
  // 生命周期方法：组件更新后执行
  componentDidUpdate(prevProps, prevState) {
    if (prevState.count !== this.state.count) {
      document.title = `You clicked ${this.state.count} times`;
    }
  }

  // 对应 useEffect
  // 生命周期方法：组件卸载前执行
  componentWillUnmount() {
    console.log('Component will unmount');
    if (this.cleanup) {
      this.cleanup();
    }
  }

  // 对应 useState
  // 处理点击事件
  handleIncrement = () => {
    this.setState((prevState) => ({ count: prevState.count + 1 }));
  };

  // 对应 useEffect
  // 处理窗口大小变化
  handleResize = () => {
    console.log('Window resized');
  };

  // 渲染方法
  render() {
    const { count, theme } = this.state;

    return (
      <div className={theme}>
        <h1>Class Component Example</h1>
        <p>You clicked {count} times</p>
        <button onClick={this.handleIncrement}>Click me</button>
        <input type="text" ref={this.inputRef} placeholder="Focus me!" />
        <ThemeContext.Consumer>
          {(value) => (
            <p>Current theme: {value}</p>
          )}
        </ThemeContext.Consumer>
      </div>
    );
  }
}

// 对应 useContext
// 创建上下文
const ThemeContext = React.createContext('light');

export default ExampleClassComponent;
```

#### 5.2.2 早期的无状态函数式组件 Stateless Functional Components <a href="#id-5.2.2-zao-qi-de-wu-zhuang-tai-han-shu-shi-zu-jian-statelessfunctionalcomponents" id="id-5.2.2-zao-qi-de-wu-zhuang-tai-han-shu-shi-zu-jian-statelessfunctionalcomponents"></a>

早期的 React 中，纯函数组件（也称为无状态函数组件，Stateless Functional Components，简称 SFCs）是一种简单的组件形式，它们只负责根据传入的 props 渲染 UI，而不包含任何状态或生命周期方法。纯函数组件的主要优点是简洁和易于理解，同时也更容易优化。

```
import React from 'react';

const MyFunctionComponent = (props) => {
  return (
    <div>
      <h1>Hello, {props.name}!</h1>
      <p>Your age is {props.age}.</p>
    </div>
  );
};

export default MyFunctionComponent;
```

纯函数组件的特点

1. 简洁：\
   纯函数组件的定义非常简洁，只包含必要的逻辑。\
   没有 class 关键字，没有 this 关键字，没有生命周期方法。
2. 易读性：\
   由于没有状态和生命周期方法，纯函数组件更容易阅读和理解。\
   性能优化：
3. React 可以对纯函数组件进行一些优化，例如在某些情况下跳过不必要的渲染。

#### 5.2.3 现在的函数式组件 <a href="#id-5.2.3-xian-zai-de-han-shu-shi-zu-jian" id="id-5.2.3-xian-zai-de-han-shu-shi-zu-jian"></a>

React 16.8.0 版本中引入了 Hook，Hook 是 React 16.8.0 的新增特性，它可以让函数组件具有状态（state）和生命周期功能。

React 官方文档中是这样介绍 Hook 的：

> Hook 是一些可以让你在函数组件里 “钩入” React state 及生命周期等特性的函数。这使得你不使用 class 也能使用 React。

Hook 的出现，让函数组件拥有了状态和生命周期，让函数组件可以像类组件一样工作。

**回头再来回答上面的问题，最简单的话来狭义地总结 Hook：Hook 就是把 React 特性（state、生命周期、ref等）带到纯函数组件中的钩子函数。**

***

那么为什么 react 团队放弃了类组件，转而推动函数式组件？

这里有一篇非常好的文章可以参考：[why-react-hooks](https://ui.dev/why-react-hooks)

这里我结合经验总结一下：

1. 代码简洁性和可读性

类组件的代码通常较为冗长，需要使用 class 关键字、constructor、super、this 等语法，增加了代码的复杂性。\
类组件的状态管理和生命周期方法的管理较为复杂，容易出错。

1. 逻辑复用和组合

高阶组件（HOCs）和 Render Props 模式虽然可以复用逻辑，但会导致组件嵌套过深，难以维护。\
逻辑复用的方式不够直观，容易引入额外的复杂性。

自定义 Hooks 允许开发者提取和复用组件逻辑，而不需要引入额外的组件层次，可以轻松地在多个组件之间共享状态和行为。

1. 更好的状态管理

类组件中 this.state 是一大坨状态保存在一起的不太利于维护。\
并且类组件需要通过 this.setState 来更新状态，这涉及到 this 的绑定和异步更新的问题。\
而函数式组件是一个状态一个 set，独立管理。

1. 更好的性能优化

类组件中 PureComponent 和 shouldComponentUpdate 虽然可以优化性能，但需要手动实现和维护。\
函数组件中 useMemo 和 useCallback 提供了更灵活的性能优化方式，而不需要引入额外的类组件层次。

1. 函数组件无须关注生命周期

```
React.useEffect(() => {
    setLoading(true)

    fetchRepos(id)
        .then((repos) => {
        setRepos(repos)
        setLoading(false)
        });
}, [id]);
```

这段逻辑表示当 id 发生改变时，先置为 loading 状态，去发请求获取数据，然后更新状态。

总结：React 团队引入 Hooks 的主要目的是为了简化和改进组件的开发体验，解决类组件的一些常见问题，并使函数组件更加功能丰富和灵活。通过让函数组件可以像类组件一样工作，React 团队不仅提高了代码的简洁性和可读性，还提供了更好的逻辑复用和性能优化方式，同时也得到了社区和生态系统的广泛支持。这些改进使得 React 的开发变得更加高效和愉悦。

### 5.3 Hook 的使用注意事项 <a href="#id-5.3hook-de-shi-yong-zhu-yi-shi-xiang" id="id-5.3hook-de-shi-yong-zhu-yi-shi-xiang"></a>

Hook 本质就是 JavaScript 函数，但是在使用它时需要遵循两条规则：

1. 只在最顶层使用 Hook。不要在循环、条件判断或者子函数中调用 Hook。
2. 只在 React 函数中调用 Hook。不要在其他 JavaScript 函数中调用。

看几个错误的例子

例子1: 条件调用 hook

```
function Example() {
  if (condition) {
    // 2. 不要在循环、条件判断或者子函数中调用 Hook
    useSomething();
  }
}
```

例子2: 条件调用 hook （容易被忽略的case）

```
function Example() {
  if (condition) {
    return <div>Welcome!</div>;
  }
  const { errorText } = useSomething();
  return <div>{errorText}</div>;
}
```

例3: 在循环中调用 hook

```
function Example() {
    for (let i = 0; i < 5; i++) {
        // 2. 不要在循环、条件判断或者子函数中调用 Hook
        useSomething(i);
    }
}
```

例 4: 在子函数中调用 hook

```
function Example() {
    function handleSomething() {
        // 2. 不要在循环、条件判断或者子函数中调用 Hook
        useSomething();
    }
}
```

### 5.4 什么时候定义一个 Hook？ 什么时候定义一个方法？ <a href="#id-5.4-shen-me-shi-hou-ding-yi-yi-ge-hook-shen-me-shi-hou-ding-yi-yi-ge-fang-fa" id="id-5.4-shen-me-shi-hou-ding-yi-yi-ge-hook-shen-me-shi-hou-ding-yi-yi-ge-fang-fa"></a>

先给大家看一个极端但正确的例子

```
function useMath () {
    function add (a, b) {
        return a + b;
    }
    function sub (a, b) {
        return a - b;
    }
    return {
        add,
        sub
    }
}

function Example () {
    const { add, sub } = useMath();
    return (
        <div>
            {add(1, 2)}
            {sub(1, 2)}
        </div>
    )
}
```

这是一个形为 Hook，但实际上只是一个普通的数学计算工具函数，完全没有必要定义成这个“形状”，对于其他的 React 开发者，会造成歧义。

我们之前提到 hook 是包含 React 特性的函数，那么我这里给一个建议或规范，那就是：

自定义 hook 所抛出的内容中，至少一个是有状态的（stated），可以被 useEffcet 的依赖项捕获从而触发重新渲染，否则，还不如直接定义一个方法。

```
function App () {
    const { something } = useSomething();
    const [ something ] = useSomething();
    
    useEffect(() => {
        // 执行一些逻辑
    }, [something]);
}
```

再看一个正确的例子

```
import { useState, useEffect } from 'react';

function useUser() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // 模拟从 API 获取用户数据
    fetchUser().then(setUser);
  }, []);

  function logout() {
    setUser(null);
  }

  return { user, logout };
}

function UserProfile() {
  const { user, logout } = useUser();

  if (!user) {
    return <p>Loading...</p>;
  }

  return (
    <div>
      <p>Welcome, {user.name}!</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

### 5.1 useContext <a href="#id-5.1-usecontext" id="id-5.1-usecontext"></a>

在 react 中 useContext 是一种十分常见的状态管理方案，它的特点是可以进行状态隔离。从下图中可以看到，只有由当前 Context 创建的 Provider 内的子组件才能访问 Context 中的值。

下面是 ReactContext 的常见用法。一般来说一个 Context 可以提供对应的 Provider 和 Consumer，其中 Provider 作为容器，一般用于容纳需要消费对应 Context 的子组件，以及进行数据隔离。

其中 Consumer 其实使用场景比较少，因为它的用法需要和 JSX 绑定，其实不太好用。因此一般是使用 react 提供的 hook - useContext 获取 Context。

但是，可以看到即使是推荐的这种方法，也需要在 A 文件创建 Context 然后在 B 文件使用 useContext 这个 hook 去拿去 Context 中的值，不是很方便。

因此在下面提供一种很优雅的 React Context 使用范式，在同一个 jsx 文件中同时抛出对应的组件和获取 context 的 hook。

最终效果如下所示，在应对大型 context 的状态管理时，这种方式的优越性将会显而易见。

***

未完待续

### 5.n 自定义 hook <a href="#id-5.n-zi-ding-yi-hook" id="id-5.n-zi-ding-yi-hook"></a>

使用 as const 作为返回值。

[https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns\_by\_version/#typescript-34](https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_version/#typescript-34)

```
function useLoading() {
  const [isLoading, setState] = useState(false);

  const load = (aPromise: Promise<any>) => {
    setState(true);
    return aPromise.finally(() => setState(false));
  };

  return [isLoading, load] as const; // infers [boolean, typeof load] instead of (boolean | typeof load)[]
}
```
