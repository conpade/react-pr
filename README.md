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
      }} >change props cannot be success</Button>
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






### system : List ###

```js

import { inject, observer } from 'mobx-react';
import { computed } from 'mobx';
import { withRouter,Link } from 'react-router-dom';
import React, { Component,Fragment } from 'react';
import { ListFilter, BlockHeader, Panel, EditForm } from '@didi/zen-bc';
import apiSvc from '../../services/apiSvc';
import { Table, message } from 'antd';
import moment from 'moment';

import ListTable from '../../components/ListTable';

class List extends Component {

  state = {
    filterData : []
  };

  columns = [
    {
      title: 'ID',
      dataIndex: 'systemId'
    },
    {
      title: '名称',
      dataIndex: 'name'
    },
    {
      title: '描述',
      dataIndex: 'description'
    },
    {
      title: '更新时间',
      dataIndex: 'updateTime',
      render: (createTime) => moment(createTime).format('YYYY-MM-DD HH:mm:ss'),
    },
    {
      title: '操作',
      dataIndex: 'actions',
      render: (text,record) => {
        const actions = Array.isArray(text) && text.map((action, index) => {
            const {type,label} = action;
            if(type === 'DETAIL'){
              return <Link style={{marginRight:'10px'}} key={index} to={`system-detail?systemId=${record.systemId}`}>{label}</Link>
            }
            else if(type === 'UPDATE'){
              return <Link style={{marginRight:'10px'}} key={index} to={`system-edit?systemId=${record.systemId}`}>{label}</Link>
            }
          });
        return (
          <span >
            {actions}
          </span>
        );
      }
    },
  ];

  getDataFunction = function(current, pageSize, search){
    return apiSvc.fetchSystemList({
      current: current,
      pageSize,
      ...search
    })
  };



  render(){
    return (
      <ListTable title="System" filterData={false} columns={this.columns}
                 getDataFunction={this.getDataFunction}
      />
    );
  }
}

export default List;
```

### system modal : EditFilterForm ###

```js
import React, { PureComponent,Fragment } from 'react';
import {EditForm,BlockHeader,Panel} from '@didi/zen-bc';
import moment from 'moment';
import apiSvc from '../../services/apiSvc';
import { Layout, Table, Tabs, Button, Modal, message } from 'antd';
import { withRouter,Link } from 'react-router-dom';
import AddFilterForm from './modals/AddFilterForm';
import EditFilterForm from './modals/EditFilterForm';
const TabPane = Tabs.TabPane;
const Content = Layout.Content;


class Edit extends PureComponent{
  constructor(props){
    super(props);
    let params = new URLSearchParams(this.props.history.location.search);
    this.systemId = params.get('systemId');
  }

  state = {
    propertyList : [],
    typeList: [],
    filterList:[],
    filterIdList:[],

    addModalVisible:false,
    selectedFilterIds:[],

    editModalVisible:false,
    editFilterId:0,
    filterDetailData:[],
    loadEditFactor:0,

    filterPropertyValueList : [],

    delModalVisible:false,
    delFilterId:0,

  }

  componentDidMount() {
    this.setTabData('filter');
  }

  setTabData = (activeTabKey)=>{
    const systemId = this.systemId;

    apiSvc.fetchSystemDetail({
      systemId
    }).then(res => {
      if (res.errno === 0) {
        let {...systemData} = res.data;

        this.form.setFieldsValue({
          ...systemData
        });
      }
    });

    this.loadSystemFilterList();

  }

  loadSystemFilterList = ()=>{
    const systemId = this.systemId;
    apiSvc.fetchSystemFilterList({
      systemId
    }).then(res => {
      if (res.errno === 0) {
        let filterIdList = [];
        res.data.items.forEach(function(item){
          filterIdList.push(item.filterId);
        });
        this.setState({
          filterIdList: filterIdList,
          filterList: res.data.items});
      }
    });
  }

  onOk = (result) => {
    console.log(result);
    apiSvc.editSystemFilter(result).then(res => {
      if(res.errno === 0){
        message.success('编辑成功！');
        this.jumpTo('system-list');
      }else{
        message.error(res.errmsg);
      }
    })
  }

  jumpTo = (path) => {
    const { history } = this.props;
    history.push(path);
  }

  /** ******************************** modal ********************************/

  handleAddModalCancel = () => {
    this.hideAddModal();
  }

  hideAddModal = () => {
    this.setState({
      addModalVisible:false
    });
  }

  hideEditModal = () => {
    this.setState({
      editModalVisible:false
    });
  }

  handleEditModalCancel = () => {
    this.setState({
      editModalVisible:false
    });
  }

  showAddModal = () => {
    this.setState({
      addModalVisible:true
    });
  }

  showEditModal = (systemId,filterId) => {
    const fid = filterId;
    this.setState({
      editModalVisible:true,
      editFilterId:fid,
    });
  }

  setSelectedFilterKeysFunction = (selectedFilterIds)=>{
    this.setState({
      selectedFilterIds
    });
  }

  setFilterValueFunction = (filterPropertyValueList)=>{
    this.setState({
      filterPropertyValueList
    });
  }

  handleAddModalOk =() =>{
    apiSvc.addFilter2System({systemId:this.systemId, filterId: this.state.selectedFilterIds}).then(()=>{this.loadSystemFilterList();});
    this.hideAddModal();
  }

  handleEditModalOk=()=>{
    this.setState({
      loadEditFactor:Math.random()
    });
    apiSvc.editSystemFilter({systemId:this.systemId, filterId: this.state.editFilterId, propertyList:this.state.filterPropertyValueList});
    this.hideEditModal();
  }

  handleDelModalOk =() =>{
    apiSvc.delSystemFilter({systemId:this.systemId, filterId: this.state.delFilterId}).then(()=>{this.loadSystemFilterList();});
    this.hideDelModal();
  }

  showDelModal = (systemId, filterId) =>{
    this.setState({
      delFilterId:filterId,
      delModalVisible:true,
    });
  }

  hideDelModal = () => {
    this.setState({
      delModalVisible:false
    });
  }

  handleDelModalCancel=()=>{
    this.hideDelModal();
  }

  /** ******************************** end of modal ********************************/

  render(){

    let columns = [
      {
        'id': 'systemId',
        'cardTitle': '基本信息',
        'label': '系统ID',
      },

      {
        'id': 'name',
        'label': '名称',
      },

      {
        'id': 'description',
        'label': '说明',
        'type': 'text',
      },

    ];

    const filterColumns = [
      {
        title: '名称',
        dataIndex: 'name'
      },
      {
        title: '描述',
        dataIndex: 'description'
      },
      {
        title: '创建时间',
        dataIndex: 'createTime',
        render: (time) => moment(time).format('YYYY-MM-DD HH:mm:ss'),
      },

      {
        title: '操作',
        dataIndex: 'actions',
        render: (text,record) => {
          return (
            <span>
              <Button
                key='edit'
                onClick={()=>{this.showEditModal(this.systemId, record.filterId)}}>

                编辑
              </Button>
              &nbsp;&nbsp;
              <Button
                key='delete'
                onClick={()=>{this.showDelModal(this.systemId, record.filterId)}}>

                删除
              </Button>
            </span>

          );
        }
      },
    ];

    return (
      <Content>
        <BlockHeader title="system编辑" />
        <Tabs
          defaultActiveKey="system"
        >
          <TabPane tab="system" key="system">
            <Panel className="outside-panel-wrapper">
              <EditForm
                editData={columns}
                onOk={this.onOk}
                ref={(node) => {this.form = node && node.formRef.props.form;}}
              />
            </Panel>
          </TabPane>

          <TabPane tab="filter" key="filter">
            <Panel className="outside-panel-wrapper">

              <Fragment>
                <BlockHeader
                  title=""
                  buttonData={[{
                    text: '添加filter',
                    type: 'primary',
                    icon: 'plus',
                    onClick: this.showAddModal
                  }]}
                />
              </Fragment>
              <Panel>
                <Table
                  columns={filterColumns}
                  bordered
                  loading={this.state.loading}
                  dataSource={this.state.filterList || []}
                  rowKey="filterId"
                  pagination={false}
                />
              </Panel>
            </Panel>
          </TabPane>

        </Tabs>

        <Modal
          visible={this.state.addModalVisible}
          title="添加Filter"
          onCancel={this.handleAddModalCancel}
          onOk={this.handleAddModalOk}
          width={"60%"}
        >
          <AddFilterForm
            setSelectedFilterKeysFunction={this.setSelectedFilterKeysFunction}
            disabledFilterIdList = {this.state.filterIdList}
            ref={(node) => {this.addFilterForm = node;}}
          />
        </Modal>

        <Modal
          visible={this.state.editModalVisible}
          title="编辑Filter"
          onCancel={this.handleEditModalCancel}
          onOk={this.handleEditModalOk}
          width={"60%"}
        >
          <EditFilterForm
            setFilterValueFunction={this.setFilterValueFunction}
            ref={(node) => {this.editFilterForm = node;}}
            filterId = {this.state.editFilterId}
            systemId = {this.systemId}
            loadEditFactor = {this.state.loadEditFactor}
          />
        </Modal>

        <Modal
          visible={this.state.delModalVisible}
          title="删除filter"
          onCancel={this.handleDelModalCancel}
          onOk={this.handleDelModalOk}
        >
          你确定要删除filter {this.state.delFilterId} 吗？
        </Modal>
      </Content>
    );
  }
}

export default Edit;
```

### system modal : EditFilterForm ###

```js
import { inject, observer } from 'mobx-react';
import { computed } from 'mobx';
import { withRouter,Link } from 'react-router-dom';
import React, { Component,PureComponent,Fragment } from 'react';
import { ListFilter, BlockHeader, Panel, EditForm } from '@didi/zen-bc';
import apiSvc from '../../../services/apiSvc';
import { Table, message, Input } from 'antd';

class EditFilterForm extends PureComponent {

  state = {
    systemId:0,
    filterPropertyValueList : [],
    propertyList: [],
    filterId:0,
  }

  static SYSTEM_DEFAULT = '默认值(system级别，填写该值会覆盖filter中配置的默认值)'

  componentWillReceiveProps(nextProps) {
    console.log('componentWillReceiveProps',nextProps.filterId,this.props.loadEditFactor,nextProps.loadEditFactor);
    if(nextProps.loadEditFactor !== this.props.loadEditFactor || nextProps.filterId != this.state.filterId){
      this.setState({
        filterId: nextProps.filterId
      });
      this.loadFilter(this.state.systemId,nextProps.filterId);
    }
  }

  componentDidMount() {

    const systemId = this.props.systemId;
    const filterId = this.props.filterId;

    this.setState({systemId});

    this.loadFilter(systemId,filterId);

  }

  loadFilter =(systemId, filterId)=>{
    console.log('loadFilter');

    apiSvc.fetchSystemFilterDetail({
      systemId : systemId,
      filterId : filterId,
    }).then(res => {
      if(res.errno === 0){
        let {propertyList,...otherData} = res.data;

        this.form.setFieldsValue({
          ...otherData,propertyList
        });

        let targetPropertyList = [];


        propertyList.forEach(function(item){
          let targetProperty = [];

          // targetProperty.push({key:'proper',value:item.filterPropertyId});
          targetProperty.push({key:'名称',value:item.name});
          targetProperty.push({key:'默认值（filter级别）',value:item.defaultValue});
          targetProperty.push({key:EditFilterForm.SYSTEM_DEFAULT,value:item.value,propertyId:item.filterPropertyId});
          targetProperty.push({key:'是否必填',value:item.isRequired});

          targetPropertyList.push(targetProperty);

        });

        this.setState({
          propertyList:targetPropertyList
        });

        // preset
        let filterPropertyValueList = [];
        targetPropertyList.forEach((item)=>{
          item.forEach((item) => {
            if(item.key === EditFilterForm.SYSTEM_DEFAULT){
              filterPropertyValueList.push({propertyId:item.propertyId,value:item.value,change:false});
            }
          });
        });

        this.setState({filterPropertyValueList});
        this.props.setFilterValueFunction(filterPropertyValueList)

      }
    });
  }

  render(){
    const propertyColumns = [
      {
        title: "属性",
        dataIndex: 'key'
      },
      {
        title: "属性值",
        dataIndex: 'value',
        render: (text, record) => {

          if(record.key === EditFilterForm.SYSTEM_DEFAULT){

            return <Input name={record.propertyId} defaultValue={text} onChange={ (e)=>{

              let key = e.target.name;
              let value = e.target.value;

              let filterPropertyValueList = this.state.filterPropertyValueList.slice();

              console.log('start change',filterPropertyValueList, key, value);

              filterPropertyValueList = filterPropertyValueList.map(function(item, index){
                if(item.propertyId === Number(key) ){
                  console.log('enter');
                  return {propertyId:item.propertyId, value: value, change:true};
                }
                else
                  return item;
              });


              console.log('change result:',filterPropertyValueList);
              this.setState({filterPropertyValueList});
              this.props.setFilterValueFunction(filterPropertyValueList)


            } } />
          }else{
            return text;
          }
        }
      },
    ];


    const {propertyList} = this.state;
    let propertyListTable = propertyList.map((item, index) =>
    {
      let tableKey = index;
      // item.forEach(function(item, index){
      //   if(item['key'] === 'propertyId'){
      //     tableKey = item['value'];
      //   }
      // });


      return <Table
        columns={propertyColumns}
        bordered
        style={{marginTop:'10px'}}
        dataSource={item || []}
        rowKey="key"
        key={tableKey}
        pagination={false}
      />
    }
    );


    let data = [
      {
        label: '过滤器名称',
        id: 'name'
      },
      {
        label: '描述',
        id: 'description'
      },
      {
        label: '过滤器类型',
        id: 'type'
      },
      {
        label: '属性',
        id: 'propertyList',
        type: 'custom',
        content: <div>{propertyListTable}</div>
      }
    ];

    return (<Panel className="outside-panel-wrapper">
      <EditForm
        editData={data}
        buttonData={[]}
        ref={(node) => {this.form = node && node.formRef.props.form;}}
      />
    </Panel>);
  }
}

export default EditFilterForm;
```
