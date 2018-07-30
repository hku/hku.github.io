---
title: react3d： A thin middleware to glue Threejs and React （part 2）
---
（上一篇 [part2](../react3d_2/index.md)）

## 五、 逐帧动画

在前一节中，我们重点阐述了为什么要在 camera 组件中完成 render loop，通过render loop 实现画布的逐帧重绘。下面我们具体来看怎样合理地实现动画逻辑。

Threejs中实现动画的基本思路是要把系统每一步的状态改变放到render loop中，即我们需要把所有3d对象的状态演化函数都添加到camera的renderFrames函数中：

```jsx
class Camera extends Object3D {
...
	renderFrames(){
		const camera = this.obj;
		const scene = this.scene;
		const webGLRenderer = this.webGLRenderer;
		
		//此处添加各个3d对象状态演化的逻辑！！！
		
		webGLRenderer.render(scene, camera);
		this.frameId = requestAnimationFrame(this.renderFrames)
	}
...
}
```

但是如果我们把所有3d对象的状态演化方式都“裸”写到renderFrames函数中，renderFrames的内容势必过于杂乱，且破坏了组件之间的模块隔离。我们希望各个组件的状态演化逻辑都封装在相应组件的内部。为此，我们利用函数的闭包特性，引入一个称为 Animations 的单例“收纳器”，它可以通过 add 方法进入每个组件内部，采集每个组件的状态演化的代码片段，然后通过 update 方法，统一在 renderFrames 运行所有组件的状态演化。 具体代码如下：

```jsx
const Animations = (function(){
  const _realtimes = [];
  return {
    add: function ( obj ) {
      
        const fresh = obj.tag?(_realtimes.map(o => o.tag || "").indexOf(obj.tag)==-1):true;
        if(fresh) {_realtimes.push( obj )}
    },
    remove: function ( obj ) {
      var i;
      if(typeof obj == "string") {
        i = _realtimes.map(o => o.tag || "").indexOf(obj);
      } else {
        i = _realtimes.indexOf( obj );
      }
      if ( i !== -1 ) {
        _realtimes.splice( i, 1 );
      }
    },
    update: function () {
      _realtimes.forEach(obj => {obj.update()});
    }
  };
})();

export {Animations}
```

同时对基类 Object3D 和 Camera 做如下修改：

```jsx
class Object3D extends React.Component {
	constructor(props) {
		super(props);
		const {update} = props;

		this.obj = this.objContructor(props);
		
		if (update instanceof Function) {
			this.updateObj = { update: () => update(this.obj) }
		}
	}

	componentDidMount(){
		const parent = this.context.parent;
		parent.add(this.obj);

		if (this.updateObj) {
			Animations.add(this.updateObj);
		} else {
			Animations.add(this);
		}

		this.objDidMount();
	}
	componentWillUnmount(){
		const parent = this.context.parent;
		parent.remove(this.obj);

		if (this.updateObj) {
			Animations.remove(this.updateObj);
		} else {
			Animations.remove(this);
		}

		this.objWillUnmount();
	}
	update(){
	}
	...
}
```

观察代码，也就是说我们可以把所有的状态演化逻辑写到组件的update方法中去（Animations.update 
会直接调用这个方法）。当然出于方便性考虑，我们也允许通过“update属性”来指定组件的状态演化逻辑。


```jsx
class Camera extends Object3D {
...
	renderFrames(){
		const camera = this.obj;
		const scene = this.scene;
		const webGLRenderer = this.webGLRenderer;
		
		Animations.update(); 
		
		webGLRenderer.render(scene, camera);
		this.frameId = requestAnimationFrame(this.renderFrames)
	}
...
}
```

注意到这种通过一个全局“收纳器”，将各个组件的update零碎片段收集起来，放到renderFrames统一执行的构造方式 和 redux 中 action 的作用非常神似。

好的，到目前为止，我们已通过 Animations，实现了逐帧动画的分拆与“模块化”。


## 六、 交互

动画实现以后，与此相关的另一个重要问题是怎样响应用户的交互操作。 不妨假设我们想要操作某个3d对象 A，并把定义操作行为的代码片段抽象为control组件。 注意到交互事件的载体是 canvas，因此显然如果这个组件放在 A 的内部，则我们可以通过context很方便的“隐式”获取 A 和 canvas，从而实现代码的封装。出于这种考虑，我们这样定义control (这里以常用的OrbitControls为例)：

```jsx
import ThreeOrbitControls from "three-orbitcontrols"

class OrbitControls extends React.Component {
	constructor(props) {
		super(props);
	}
	componentDidMount(){
		const canvas = this.context.canvas;
		const camera = this.context.parent;
		const controls = new ThreeOrbitControls(camera, canvas);
	}
	render(){
		return null;
	}
}

OrbitControls.contextTypes = {
	parent: React.PropTypes.object,
	canvas: React.PropTypes.object
};

export default OrbitControls;
```

有了这样定义的 OrbitControls，我们就可以直接通过在某个组件A内部嵌套该组件，以实现相应的交互功能。 例如：

```xml
<Camera>
	<OrbitControls/>
</Camera>
```
代码就是这么简练，就是这么干净！


## 六、 补间动画

补间动画是threejs中另一个常常需要实现的功能。补间动画和逐帧动画最大的区别是，逐帧动画需要指定动画每一步运动的方式，而补间动画是根据组件的始末状态自动实现过渡动画。在一个react应用中，明确知晓组件的始末状态是常见的情形，而react中并没有支持这种过渡动画的现成逻辑。

另一方面threejs中过渡动画相关常用的js库是tween.js，由于我们希望开发方式完全符合react的xml方式，使得代码更规整，更有“套路”可循，我们将tween的功能抽象为一个Tween 组件，并且满足以下直观好用的调用方式：

```jsx
<Tween data={data} view = {v => A}/>

```
其中data指定数据，view 是根据“补间值”生成相关组件的“工厂函数”。 为满足这一调用方式，我们定义了如下Tween组件

```jsx
import React from "react";
import TWEEN from "tween";

class Tween extends React.Component {
	constructor(props) {
		super(props);
		const {data} = props;
		this.state = Object.assign({}, data);
		this.tween = null;
		this.fireTween = this.fireTween.bind(this);
	}
	componentWillReceiveProps(nextProps) {
		const {data} = nextProps;
		const {data: preData} = this.props;
		for (let k in data) {
			if (preData[k] != data[k]) {
				this.fireTween(data)
				break;
			}
		}
	}
	fireTween(data){
		const newData = Object.assign({}, this.state);
		const me = this;
		me.tween = new TWEEN.Tween(newData).to(data, 1000).easing(TWEEN.Easing.Quadratic.Out).onUpdate(function() {
			me.setState(newData);
		}).start();
	}

	render(){

		const {view} = this.props;
		return view(this.state);
	}
}

export default Tween;
```
综上，我们已经在react的模块化框架内，实现了事件响应、逐帧动画和补间动画，于是整个react3d轻量级框架的也就诞生……

后面再整理下，并提高代码的健壮性，会把整个js库公布出来……就是酱紫，写完收工！