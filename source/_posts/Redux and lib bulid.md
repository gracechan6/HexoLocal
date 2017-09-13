---
title: React交互二：Redux 以及库打包方法介绍
date: 2016-10-27 10:36:13
tags:
---


## Redux
上一章介绍了Flux，但是Flux也有一个问题就是一个动作里面处理逻辑的时候在派发一个事件，那无法处理，系统会中断报错。而本章要讲的Redux就很好的解决了这个问题。Redux 是 JavaScript 状态容器，提供可预测化的状态管理。
可以让你构建一致化的应用，运行于不同的环境（客户端、服务器、原生应用），并且易于测试。不仅于此，它还提供 超爽的开发体验，比如有一个时间旅行调试器可以编辑后实时预览。
Redux 除了和 React 一起用外，还支持其它界面库。
它体小精悍（只有2kB）且没有任何依赖。

### Redux 和传统 Flux 框架的比较

viewer文件结构图
![viewer文件结构图](/images/Redux.jpg)

从图片中可以看出来区别就是dispatch那部分拆成了Reducer和MiddleWare，实际操作起来区别还是蛮大的。可以从下面看出来的。

### Action 发生了什么

直接看代码吧 /reduxEvent/actions/OPAction.js

``` bash
/**
 * Created by graceChen on 2016/7/8.
 */
var type="";
var OPAction = {
    doAction:function(type,text){
        this.type=type;
        return{
            type:type,
            text:text
        }
    }
};
module .exports=OPAction;
```

可以看出和Flux有细微的区别，Flux需要通过aciton触发store，而这里是通过定义不同的方法，把数据从应用（注：这里之所以不叫 view 是因为这些数据有可能是服务器响应，用户输入或其它非 view 的数据）传到 store 的有效载荷。它是 store数据的唯一来源。一般来说你会通过 store.dispatch() 将 action 传到 store。
在Action里创建了一个函数，只是简单的返回一个action，在传统的 Flux 实现中，当调用 action 创建函数时，一般会触发一个 dispatch，而不同的是，Redux 中只需把 action 创建函数的结果传给 dispatch() 方法即可发起一次 dispatch 过程。

### Reducer 怎么做
Action 只是描述了有事情发生了这一事实，并没有指明应用如何更新 state。而这正是 reducer 要做的事情。/reduxEvent/reducers/OPReducer.js

``` bash
/**
 * Created by graceChen on 2016/7/11.
 */
var initialState = {
    item:{},
    type:''
};
var OPReducer = function(state=initialState, action){
    var item = state.item;
    item[action.type]=action.text;
    return Object.assign({},state,{type:action.type,item:item});
    state.item[action.type] = action.text;
    state.type = action.type;
    return state;
};
module .exports= OPReducer;
```

在Redux里面，所有的 state 都被保存在一个单一对象中。上面写的就是根据项目的结构构思出来的对对象的一个比较简单的表现形式。
type:交互(事件)类型
item:不同类型里的不同数据(可以数组)

随着应用变得复杂，需要对 reducer 函数 进行拆分，拆分后的每一块独立负责管理 state 的一部分。这个时候就可以用到combineReducers，它的作用就是把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用 createStore。/reduxEvent/reducers/OPCombineReducers.js

```bash
/**
 * Created by graceChen on 2016/7/11.
 */
var OPReducer = require('./OPReducer');
//使用redux的combineReducers方法将所有reducer打包起来
var OPCombineReducers = Redux.combineReducers({
    OPReducer
});
module.exports = OPCombineReducers;
```
以逗号间隔一个一个引用进来就行，最后页面需要的时候直接引用一个OPCombineReducers即可。

### Store 怎么做以及维护
Store 就是用来维持应用所有的 state 树 的一个对象。 改变 store 内 state 的惟一途径是对它 dispatch 一个 action。
区分Flux:Redux没有Dispatcher且不支持多个store。相反，只有一个单一的store和一个根级的reduce函数（reducer）。随着应用不断变大，你应该把根级的 reducer 拆成多个小的 reducers，分别独立地操作state树的不同部分，而不是添加新的stores。这就像一个React应用只有一个根级的组件，这个根组件又由很多小组件构成。
/reduxEvent/store/OPStore.js

``` bash
var todoReducer = require('../reducers/OPCombineReducers');
var OPStore = Redux.createStore(todoReducer);
module .exports =OPStore;
``` 

createStore() 的第二个参数是可选的,用于设置state初始状态。这对开发同构应用时非常有用，服务器端redux应用的state结构可以与客户端保持一致那么客户端可以将从网络接收到的服务端 state 直接用于本地数据初始化。

``` bash
let store = createStore(todoApp, window.STATE_FROM_SERVER);
```

### View 页面展示
这里只显示关键代码，实际项目中应用比较多比较广，全部js代码比较长，就不全贴出来了。

``` bash
refreshDataHandler:function(){
       //判断dispatch中的type是否和本地监听的事件名一致，如果一致则正常操作，否则不操作。
       if (ReduxEvent.Action.type != EventName.TIME_CHANGE)
           return;
       console.log("时间轴事件派发开始："+ReduxEvent.Action.type);
       var timeValue = ReduxEvent.Store.getState().OPReducer.item[EventName.TIME_CHANGE];
       var timeGap = timeValue.TimeGap;
       if (timeGap*1%5==0) {
           this.updateDevicesStaticInfo();
       }
   },
	componentDidMount: function() {
       this.unsubscribe = ReduxEvent.Store.subscribe(this.refreshDataHandler);//注册监听事件 以及返回注销方法
   },
   componentWillUnmount: function() {
       this.unsubscribe();//注销事件
   },
```

ReduxEvent.Store 这个是用直接打包出来的库直接引用的，下面会有介绍。html页面直接引用了一个js，即ReduxEvent.js文件，上面的各种js文件都没有引用，因为压缩打包成一个了。

> 参考文献
[Redux 中文文档](http://cn.redux.js.org/index.html)
[react+redux教程](http://www.cnblogs.com/lewis617/p/5145073.html)

## 库打包
这里直接简单介绍下Redux的打包方法，已经写好了一个比较完善的机制，不需要再去修改它，直接可以各个项目中使用的，那就可以直接打包成一个库文件，到时候直接引用即可，方便简单，省时省力。

### 先定义一个ReduxEvent文件，把需要用到的Store和Action引用在一起，方便调用

/reduxEvent/ReduxEvent.js

``` bash
/**
 * Created by graceChen on 2016/7/20.
 */
var Store = require("./store/OPStore");
var Action = require("./actions/OPAction");
var ReduxEvent={
    Store:Store,
    Action:Action
}
module.exports = ReduxEve
```

### 另外写一个library.config.js打包文件

``` bash
/**
 * 文件创建日期：2016/7/20.
 * 作者：gracechan
 * 功能描述：打包库
 * 编写配置参考：http://fakefish.github.io/react-webpack-cookbook/Authoring-libraries.html
 */
module.exports = {
    externals:{
        //第三方或者本地组件在此添加将不会被webpack打包，然后需要在页面中自行引入js库
        "react":'React',
        "react-dom":'ReactDOM',
        "redux":"Redux",
        "flux":"Flux",
    },
    entry:{//指定打包的入口文件，每有一个键值对，就是一个入口文件
        ReduxEvent: ["./common/reduxEvent/ReduxEvent.js"]
    },
    output: { //配置打包结果，path定义了输出的文件夹，filename则定义了打包结果文件的名称
        filename: "../react/[name].js",
        libraryTarget: 'umd',
        library: '[name]', //库名 引用里面文件的时候也是以库名来引用的
    },
    resolve: {//定义了解析模块路径时的配置，常用的就是extensions，可以用来指定模块的后缀，这样在引入模块时就不需要写后缀了，会自动补全
        extensions: ['','.js','.jsx']
    },
    module: {//定义了对模块的处理逻辑，这里可以用
        loaders: [//loaders定义了一系列的加载器，以及一些正则。当需要加载的文件匹配test的正则时，就会调用后面的loader对文件进行处理，这正是webpack强大的原因。
            { test: /\.js$/, loader: 'jsx-loader' },
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```
这个文件看起来应该很熟悉了，就是一开始第一篇文章《React结合Webpack使用》里面介绍的文件。

打开命令窗口，定位到这个打包文件目录下，运行 npm run wlib 即可打包文件。完成后可以去react文件夹下查看是否输出成功，然后在相应的html文件里面直接引用该文件即可使用！~