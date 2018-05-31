# coder-mis 前端教程 #

## 路由 ##
要让我们的页面能够被访问到，首先我们需要配置好路由器，
找到 src/router/routers.js （省略了部分import）

```js
import * as path from '../path';
import Test from '../pages/test/Test'

const Root = () => (
  <div>
    <Switch>
      <Route
        path="/"
        render={props => (
          <App>
            <Switch>

              {<Route path={path.Test} component={Test} />}

              {/*<Route render={() => <Redirect to="/" />} />*/}
            </Switch>
          </App>
        )}
      />
      <Route render={() => <Redirect to="/" />} />
    </Switch>
  </div>
);

export default Root;
```

这里我们配置了一个测试的route，path是路径，component则是要加载的组件。

```js
{<Route path={path.Test} component={Test} />}
```
在顶部的import，我们可以看到来源。

path 来自 src/path/index.js

```js
export const Test = '/test';
```

Test组件来自 src/pages/test/Test

## 组件 ##
组件是react的概念，它必须要有render方法，输出一些内容，其它都不是必须的。
```js
import React, { Component,PureComponent } from 'react';
class Test extends PureComponent{
  render(){
    return <div>12</div>
  }
}

export default Test;
```

[效果](http://172.23.37.204:3001/test)

### JSX ###
```js
<div>12</div>
```
是一个JSX表达式，它很像HTML，但其实并不是，不过基本用法和HTML类似
可以嵌套，可以添加属性
并且支持嵌入JS表达式 使用 {} 括起来即可
```js
class TabPane extends PureComponent{
	<TabPane tab="system" key="system">
		<Panel className="outside-panel-wrapper">
		  <EditForm
		    editData={columns}
		    onOk={this.onOk}
		    ref={(node) => {this.form = node && node.formRef.props.form;}}
		  />
		</Panel>
	</TabPane>
}
```

### 箭头方法 ###

提一下箭头方法
ref=后面就是箭头方法，它自动绑定this到当前的词法作用域，也就是当前的TabPane对象。
参数只有一个的时候可以不加（），函数中只有一个返回表达式时，可以不写{}和return
```js
x => x*2
x => {x+=2;return x;}
(x,y) => x*y
(x,y) => {return x*y}
```
### props 参数 ###
this.props代表从父组件传入的参数，通过它，我们可以对组件进行抽象。

```js
import React, { Component,PureComponent } from 'react';
class Props extends Component{
  render(){
    return <div>your name : {this.props.name}</div>
  }
}

export default Props;
```

## 动态组件与state ##
组件本身的属性都是不可变的，所以引入state，用来保存组件的内部状态，一旦state改变，react会自动重新渲染
我们通过前面的Props类来实现一个动态的组件
```js
import React, { Component,PureComponent } from 'react';
import Props from './Props'
import { Layout, Table, Tabs, Button, Modal, message } from 'antd';

class Test extends PureComponent{

  state = {
    name : '1',
  }

  render(){
    return <div>
      <div>12</div>

      <br /><br /><br /><br />
      <Props name={this.state.name} />
      <Button onClick={()=>{
        let name = '1';
        name += this.state.name;
        this.setState({name});
      }} >change props</Button>

      <Button onClick={()=>{
        this.setState({name:this.state.name ++ });
      }} >change props can't be success</Button>
    </div>
  }
}

export default Test;
```
要注意，第1个button能够引发重新渲染，第2个不行，为什么？
原因是，只有state的引用改变才能引发重新渲染

## state提升，组件之间的交互 ##
上面的例子是父组件state来控制子组件的改变
那么2个组件如果互相影响？

把state放到它们的父组件，把state作为props传入子组件，然后向子组件传入能够引发state变化的函数，操作子组件时，通过该函数修改父组件的state，就可以进行组件之间的交互。

```js
import React, { Component,PureComponent } from 'react';
import Props from './Props'
import { Layout, Table, Tabs, Button, Modal, message } from 'antd';

class Textarea extends PureComponent{

  state = {
    name : '1',
  }

  onClick = (e)=>{
    let name = e.target.value
    this.props.changeCallback( name );
  }

  render(){
    return <div>
      <textarea onChange={this.onClick}>请输入name</textarea>
    </div>
  }
}

export default Textarea;
```
