

# react-activation

[中文文档](https://github.com/CJY0208/react-activation/blob/master/README_CN.md)

### react缓存组件或路由，类似于Vue中的 ```<keep-alive />```


### 1. 兼容性

- React v17+ (beta)
- React v16+
- Preact v10+
- 兼容 SSR

---
### 2. 安装
	yarn add react-activation
	# 或者
	npm install react-activation
---
### 3. 使用方式
**1.babel 配置文件 ```.babelrc``` 或者在```package.json```中增加 ```react-activation/babel``` 插件**

	##babelrc中
	
	{
	  "plugins": [
	    "react-activation/babel"
	  ]
	}
	
	##package.json中
	
	"babel": {
	    "plugins": [
	      "react-activation/babel",
	    ]
	  }
	 
**2.业务代码中，在不会被销毁的位置放置``` <AliveScope>``` 外层，一般为应用入口处**

*注意：与 react-router 或 react-redux 配合使用时，需要将``` <AliveScope>``` 放置在 ```<Router>``` 或 ```<Provider>``` 内部*
	
	##路由组件
	import React, {Component} from 'react';
	import {BrowserRouter as Router, Route, Switch} from "react-router-dom";
	import PrivateRoute from "../components/Hoc/PrivateRoute";
	import KeepAlive, { AliveScope } from 'react-activation';
	import Banner from '../components/Banner';
	export const successRouters = (
		<Router>
			<AliveScope>
				<Switch>
					<Route path="/" exact
		                       render={(props)=>{
		                           let Component = Pages.HomePage;
		                           return (
		                               <div className={'home'}>
		                                   <Banner/>
		                                   <KeepAlive>
		                                       <Component {...props} />
		                                   </KeepAlive>
		                               </div>
		
		                           )
		                       }}
		                />
		                <Route path="/login" exact component={Pages.LoginPage}/>
		                <PrivateRoute keep={true} keepName={'openWorkFlow'}
                    			exact path="/openWorkFlow" component={Pages.OpenWorkFlowPage}/>
				</Switch>
			</AliveScope>
		</Router>
	)
	
	
 **3. 用 ``<KeepAlive>`` 包裹需要保持状态的组件**
	
	##PrivateRoute组件   过滤路由封装组件  判断是否要缓存
	
	import React from "react";
	import {Route, withRouter} from "react-router-dom";
	import {connect} from "react-redux";
	import KeepAlive from 'react-activation';

	class PrivateRoute extends React.Component {
	
	  render() {
	    let {component: Component, path = "/", exact = false, strict = false,history,isAuthenticated, 			keep, keepName} = this.props;
	    if (!isAuthenticated) {
	      //报错 重新登里
	      history.push('/404')
	    }
	    return <Route path={path} exact={exact} strict={strict}
	                  render={(props)=>{
	                    if(keep){
	                      return (
	                          <KeepAlive id={keepName} name={keepName} when={true}>
	                            <Component {...props} />
	                          </KeepAlive>
	                      )
	                    }else{
	                      return (<Component {...props} />)
	                    }
	                  }}
	    />;
	  }
	}
	
---
### 3. 生命周期
```ClassComponent``` 可配合 ``withActivation`` 装饰器

使用 ```componentDidActivate``` 与 ```componentWillUnactivate``` 对应激活与缓存两种状态

```FunctionComponent``` 则分别使用 ```useActivate``` 与 ```useUnactivate``` hooks 钩子
	
	##例子来自react-activation文档（componentDidActivate没有打印出来）

	...
	import KeepAlive, { useActivate, useUnactivate, withActivation } from 'react-activation'
	
	@withActivation
	class TestClass extends Component {
	  ...
	  componentDidActivate() {
	    console.log('TestClass: componentDidActivate')
	  }
	
	  componentWillUnactivate() {
	    console.log('TestClass: componentWillUnactivate')
	  }
	  ...
	}
	...
	function TestFunction() {
	  useActivate(() => {
	    console.log('TestFunction: didActivate')
	  })
	
	  useUnactivate(() => {
	    console.log('TestFunction: willUnactivate')
	  })
	  ...
	}
	...
	function App() {
	  ...
	  return (
	    {show && (
	      <KeepAlive>
	        <TestClass />
	        <TestFunction />
	      </KeepAlive>
	    )}
	  )
	}
	...


---
###4.保存滚动位置（默认为 true）
```<KeepAlive />``` 会检测它的 ``children`` 属性中是否存在可滚动的元素，然后在 ```componentWillUnactivate``` 之前自动保存滚动位置，在 ```componentDidActivate``` 之后恢复保存的滚动位置

如果你不需要 ```<KeepAlive />``` 做这件事，可以将 ```saveScrollPosition``` 属性设置为 ```false```

	<KeepAlive saveScrollPosition={false} />
如果你的组件共享了屏幕滚动容器如 ```document.body``` 或 ```document.documentElement```, 将 ```saveScrollPosition``` 属性设置为 ```"screen" ```可以在 ```componentWillUnactivate``` 之前自动保存共享屏幕容器的滚动位置

	<KeepAlive saveScrollPosition="screen" />
	


---
###5. 多份缓存（根据id来区分路由实现多份缓存）
同一个父节点下，相同位置的 ``<KeepAlive>`` 默认会使用同一份缓存

例如下述的带参数路由场景，``/item`` 路由会按 ``id`` 来做不同呈现，但只能保留同一份缓存

	<Route
	  path="/item/:id"
	  render={props => (
	    <KeepAlive>
	      <Item {...props} />
	    </KeepAlive>
	  )}
	/>
类似场景，可以使用 ``<KeepAlive>`` 的 ``id`` 属性，来实现按特定条件分成多份缓存

	<Route
	  path="/item/:id"
	  render={props => (
	    <KeepAlive id={props.match.params.id}>
	      <Item {...props} />
	    </KeepAlive>
	  )}
	/>


---
###6. 缓存控制
####1. 自动控制缓存
给需要控制缓存的 ``<KeepAlive />`` 标签增加 ``when`` 属性，取值如下
当 ``when`` 类型为 ``Boolean`` 时

- true: 卸载时缓存
- false: 卸载时不缓存

```
	<KeepAlive when={true}>
```

当 ``when`` 类型为 ``Array`` 时

第 1 位参数表示是否需要在卸载时缓存

第 2 位参数表示是否卸载 <KeepAlive> 的所有缓存内容，包括 <KeepAlive> 中嵌套的所有 <KeepAlive>

	// 例如：以下表示卸载时不缓存，并卸载掉嵌套的所有 `<KeepAlive>`
	<KeepAlive when={[false, true]}>
	  ...
	  <KeepAlive>
	    ...
	    <KeepAlive>...</KeepAlive>
	    ...
	  </KeepAlive>
	  ...
	</KeepAlive>

当 ``when`` 类型为 ``Function`` 时
返回值为上述 ``Boolean`` 或 ``Array``，依照上述说明生效

####2. 手动控制缓存

1.给需要控制缓存的 ``<KeepAlive />`` 标签增加 ``name`` 属性

2.使用 ``withAliveScope`` 或 ``useAliveController`` 获取控制函数

+ drop(name)：
按 name 卸载缓存状态下的 <KeepAlive> 节点，name 可选类型为 String 或 RegExp，注意，仅卸载命中 <KeepAlive> 的第一层内容，不会卸载 <KeepAlive> 中嵌套的、未命中的 <KeepAlive>

+ dropScope(name)：
按 name 卸载缓存状态下的 <KeepAlive> 节点，name 可选类型为 String 或 RegExp，将卸载命中 <KeepAlive> 的所有内容，包括 <KeepAlive> 中嵌套的所有 <KeepAlive>

+ clear()：
将清空所有缓存中的 KeepAlive

+ getCachingNodes()：
获取所有缓存中的节点

```
...
import KeepAlive, { withAliveScope, useAliveController } from 'react-activation'
...
<KeepAlive name="Test">
  ...
    <KeepAlive>
      ...
        <KeepAlive>
          ...
        </KeepAlive>
      ...
    </KeepAlive>
  ...
</KeepAlive>
...
function App() {
  const { drop, dropScope, clear, getCachingNodes } = useAliveController()
	
  useEffect(() => {
    drop('Test')
    // or
    drop(/Test/)
    // or
    dropScope('Test')
	
    clear()
  })
	
  return (
    ...
  )
}
// or
@withAliveScope
class App extends Component {
  render() {
    const { drop, dropScope, clear, getCachingNodes } = this.props
	
    return (
      ...
    )
  }
}
...
```

###7. 原理概述
将 ``<KeepAlive />`` 的 ``children`` 属性传递到 ``<AliveScope />`` 中，通过 ``<Keeper />`` 进行渲染

``<Keeper />`` 完成渲染后通过 DOM 操作，将内容转移到 ``<KeepAlive />`` 中

由于`` <Keeper />`` 不会被卸载，故能实现缓存功能


###8. 使用注意

####1. 组件中不要使用```react-router-dom```的 ```withRouter```，否则使用```keepAlive```会报错，如果子组件没有```history```,可以由父组件传递给子组件。
####2. 当前页面使用了缓存，如果返回上一页或者跳转其他页面时想去掉缓存，在当前页面不能直接使用 ```drop(name)```(name是当前页面keepName)，可以在跳转或者返回的方法后面加上延时器```setTimeout(() => drop(name))```,  或者在跳转后的页面中执行```drop(name)```。在缓存对象只有在休眠状态下可以删除，激活状态不可以。

	/**
     * 点击左侧icon按钮返回上一页
     */
    onLeftClick(){
        let {drop} = this.props;
        this.props.history.goBack(); //返回上一页
        setTimeout(()=>{
            drop('EightWeekSalesForecastQuery') //删除缓存
        },200)
    }


