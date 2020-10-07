# react react-router-dom 踩坑——根据是否登录进行页面展示
我做了一个简单的前端，为了省力，把登录成功的`token`保存到`localStorage`里，我在一个受保护的页面被访问之前在`useEffec`中判断是否存在`token`，如果存在就可以渲染，否则用`react-router-dom`的`Redirect`进行重定向。但是似乎出了一点问题。

大概代码逻辑如下：
```js
export default function Dashboard() {
  let [isLogin, setIslogin] = React.useState(false);
  React.useEffect(() => {
    if (localStorage.getItem("token")) {
      setIslogin(true);
      console.log("ok");
    }
    console.log(isLogin);
  }, [isLogin]);

  if (!isLogin) {
    return <Redirect
      to={{
        pathname: "/login",
      }}
    />
  } else {
    return (
      <h1>您有权限且正在访问一个受保护的页面</h1>
    );
  }
}
```

且控制台的打印有点出乎意料:
```bash
ok
false
```

问题1：我记得使用了useEffect的函数组件的行为比较奇特，在官网上说好像什么每一个函数组件都只是一个快照什么...记不清了，反正挺玄幻的。就是useEffect的第二个参数，只要一变就会生成一个新的组件覆盖原来的。**比如在本例中，你在函数体重设置`isLogin`为false，所以就先生成了这么一个\<Dashboard/\>，但是你在useEffect中又改为true了，那么就会生成一个新的把旧的覆盖掉，也就是说，理论上这个 `console.log(isLogin);`会执行两次，**

问题2：关于hook中的setxxxx，好像每次执行都挺他妈“异步”的，就是每次执行完都没什么用，感觉跟每执行一样，在知乎上看是因为什么闭包？没搞懂

关于今天这个问题，我初步分析了一下，原因如下：
>   这样是不行的，因为，在第一次渲染组件的时候isLogin是false，当改为true了以后，触发了第二次渲染，但是在第二次渲染之前，就被重定向走了，没有触发二次渲染。

这也解释了控制台为什么只打印了一遍。

我后来改了一些内容
```js
let [isLogin, setIslogin] = React.useState(localStorage.getItem("token") ? false : true);
```
这样改了以后就好了。

其实我还在想能不能这样，在上级的路由渲染的时候就决定是否重定向。

```js
<Route path="/dashboard">
	// if login
	<Dashboard />
	// else
	<Redirect to={{
        pathname: "/login",
      }} />
</Route>
```
就像`react-router-dom`中提到的，直接写一个身份认证路由。

## 网上的参考
外网文章救我狗命：
[https://bezkoder.com/react-hooks-jwt-auth/](https://bezkoder.com/react-hooks-jwt-auth/)
这文章写得也太尼玛好了。看就完事了。