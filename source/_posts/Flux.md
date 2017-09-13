---
title: React交互一：Flux
date: 2016-10-27 10:23:38
tags:
---


## Flux

现在的很多在使用的系统都采取的是MVC模式，而前一篇介绍的React自称在MVC里面代表V 的部分，那么Flux就相当于添加M和C的部分。Flux是 Facebook 使用的一套前端应用的架构模式。

一个 Flux 应用主要包含四个部分：


* The Dispatcher
处理动作分发，维护 Store 之间的依赖关系

* The Stores
用来存储数据和处理逻辑部分

* The Views
控制React 组件，这一层可以看作 controller-views，作为视图同时响应用户交互

* The Actions
提供给 dispatcher 传递数据给 store


针对以上提出的概念，下面使用目前项目在使用的几个文件内容来实现这些功能，当然，网络上有很多这类开源的文件，实际项目中也是在这些开源的项目上再进行优化使用的。
单向数据流

### 单向数据流

先来了解下Flux的核心“单向数据流“怎么运作的：

` Action -> Dispatcher -> Store -> View `

更多时候 View 会通过用户交互触发 Action，所以一个简单完整的数据流类似这样：
viewer文件结构图

项目对于交互的整个流程如下：


* 首先要有action，通过定义一些 action creator 方法根据需要创建 Action 将数据提供给 dispatcher

* View 层（React组件）通过用户交互会触发 Action

* Dispatcher 会分发触发的 Action 给所有注册的 Store 的回调函数

* Store 回调函数根据接收的 Action 更新自身数据之后会触发一个 change 事件通知 View 数据更改了

* View 会监听这个 change 事件，拿到对应的新数据并调用 setState 更新组件 UI

所有的状态都由 Store 来维护，通过 Action 传递数据，构成了如上所述的单向数据流循环，所以应用中的各部分分工就相当明确，高度解耦了。
这种单向数据流使得整个系统都是透明可预测的。

### Dispatcher

一个应用只需要一个 dispatcher 作为分发中心，管理所有数据流向，分发动作给 Store，没有太多其他的逻辑（一些 action creator 方法也可以放到这里）。
Dispatcher 分发动作给 Store 注册的回调函数，这和一般的订阅/发布模式不同的地方在于：


* 回调函数不是订阅到某一个特定的事件/频道，每个动作会分发给所有注册的回调函数

* 回调函数可以指定在其他回调之后调用


参见comEventMessage/ComEventDispatcher.js
``` bash
var Dispatcher = require('flux').Dispatcher;
var ComEventDispatcher = new Dispatcher();
var ComEventStore = require('./ComEventListStore');
ComEventDispatcher.register(function (action) {
      ComEventStore.dispatch(action.actionType,action.text);
});//主要是这句话
module.exports = ComEventDispatcher;
```

### Action

首先要创建动作，通过定义一些 action creator 方法来创建，这些方法用来暴露给外部调用，通过 dispatch 分发对应的动作，所以 action creator 也称作 dispatcher helper methods 辅助 dipatcher 分发。 参见comEventMessage/ComEventAction.js

``` bash
var ComEventDispatcher = require('./ComEventDispatcher');
var ComEventAction = {
    dispatch: function (eventname,data) {
        ComEventDispatcher.dispatch({
            actionType: eventname,
            text: data
        });
    },
    stopDispatch:function(){
        ComEventDispatcher._stopDispatching();
    }
};
module.exports = ComEventAction;
```

这里的dispatch就是action creator，通过用户的交互触发，比如按钮点击。当然不单单用户触发才调用，也可以通过计时器的方式等的，然后分发给Store初始数据。通过这一个动作我们可以看出动作其实就是用来封装数据和传递数据的。

### Store

Store中包含应用的状态和处理逻辑，不同的Store可以管理应用中不同部分的状态。本项目中只使用了一个Store，直接在相应React组件中直接监听相应actionType，然后做出不同的事件处理。（处理逻辑和Flex事件的处理处理逻辑类似的）如comEventMessage/ComEventListStore.js

``` bash
var EventEmitter = require('events').EventEmitter;
var assign = require('object-assign');
var ComEventListStore = assign({}, EventEmitter.prototype, {
  event_data: {},
  getAll: function () {//获取所有数据
    return this.event_data;
  },
  getOne:function(key){//依据key查找单个
    if(arguments[1]=="layer"){
      return ComEventData[key];
    }else{
      return this.event_data[key];
    }
  },
  dispatch: function (eventname,data) {
    if(data){
      //alert(eventname+";"+data);
      this.event_data[eventname]=data;
      ComEventData[eventname]=data;
    }
    this.emit(eventname);
  },
  addEventListener: function(eventname,callback) {
    this.on(eventname, callback);//注册监听
  },
  removeEventListener: function(eventname,callback) {
    this.removeListener(eventname, callback);
  }
});
module.exports = ComEventListStore;
```

在 Store 注册给 dispatcher 的回调函数中会接受到分发的 action，因为每个 action 都会分发给所有注册的回调，所以回调函数里面要判断这个 action 的 type 并调用相关的内部方法处理更新 action 带过来的数据，再通知 view 数据变更。

Store 里面不会暴露直接操作数据的方法给外部，暴露给外部调用的方法都是 Getter 方法，没有 Setter 方法，唯一更新数据的手段就是通过在 dispatcher 注册的回调函数。

### View

View 就是 React 组件，从 Store 获取状态（数据），绑定 change 事件处理。如 customer_components/bottom_component/BottomTrafficButtonMenu.js

``` bash
/**
 * 文件创建日期：2016/7/22.
 * 作者：
 * 功能描述：
 *
 */
var  lastType = null;
var BottomTraficButtonMenu=React.createClass({
    getInitialState: function () {
        return {
        }
    },
    clearChoosed: function () {
        if(lastType){
            lastType.src=lastType.src.replace("_hover.png",".png");
        }
    },
    componentDidMount: function() {
        lastType = $("#ALLSubject")[0];
        FluxComEvent.store.addEventListener(EventName.ILLEGAL_CLICK,this.clearChoosed);
    },
    componentWillUnmount: function() {
        FluxComEvent.store.removeEventListener(EventName.ILLEGAL_CLICK,this.clearChoosed);
    },
    subjectSwitchHandler:function(event){
        if(lastType){
            lastType.src=lastType.src.replace("_hover.png",".png");
        }
        lastType = event.currentTarget,
            lastType.src=lastType.src.replace(".png","_hover.png");
        $("#wfDSO").css("color", "#FFFFFF");
        FluxComEvent.action.dispatch(EventName.BOTTOM_TYPE_CLICK, event.target.id);
    },
    render:function(){
        return(
            <div id='div_subjectSwitch' className="div_subjectSwitch_style">
                <img id='ALLSubject' className="menuImg_style" src='./modules/dsp/gisapi/assets/brn_1_hover.png' title='整体指标'  onClick={this.subjectSwitchHandler}/>
                <img id='CONGESTION' className="menuImg_style" src='./modules/dsp/gisapi/assets/brn_2.png' title='拥堵专题'  onClick={this.subjectSwitchHandler} />
                <img id='SEGMENT_FLOW'  className="menuImg_style" src='./modules/dsp/gisapi/assets/brn_4.png' title='流量专题'  onClick={this.subjectSwitchHandler }/>
                <img id='SEGMENT_SPEED' className="menuImg_style" src='./modules/dsp/gisapi/assets/brn_3.png' title='速度专题'  onClick={this.subjectSwitchHandler} />
                <img id='TIMING'  className="menuImg_style" src='./modules/dsp/gisapi/assets/brn_6.png' title="时间" onClick={this.subjectSwitchHandler}  />
                <img id='SATURATION' className="menuImg_style" src='./modules/dsp/gisapi/assets/brn_5.png' title='饱和度专题'  onClick={this.subjectSwitchHandler} />
            </div>
        )
    }
});
module.exports = BottomTraficButtonMenu;
```

一个View可能关联多个Store来管理不同事件的处理。需要注意的就是，当组件移除的时候事件监听也应该一起移除掉。


> 更多资料
[Flux chat ](https://speakerdeck.com/fisherwebdev/fluxchat)很简洁明了的一个 Slide

