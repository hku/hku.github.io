---
title: react3d： A thin middleware to glue Threejs and React （part 2）
---
（上一篇 [part1](../react3d/index.md)）

## 三、 核心组件

在编写react3d库的其他核心组件之前，我们再啰嗦几句。 [part1](../react3d/index.md) 中比较特别的地方是getChildContext的使用。

我们知道React 中的数据流由父元素向子元素进行单向传递。这种传递一般是通过“属性”（attributes）进行“显式”的传递。但是在某些场合下，我们希望数据传递是“隐式”的，从而把代码封装的更好。
例如在[part1](../react3d/index.md)的讨论中， 一个组件**始终**需要向其子元素传递它所代理的threejs对象，因此我们希望将这种反复出现的传递pattern，以“隐式”的方式进行。 getChildContext 正是为了实现这种数据“隐式”传递的需要。关于React Context，[这篇文章](https://medium.com/differential/react-context-and-component-coupling-86e535e2d599)的介绍比较详细。

好，言归正传。 我们介绍完 `react3d.Object3D` 实现的思想后，下面我们将以 `react3d.Object3D` 作为基础类，构建其他核心组件


#### **Scene**

首先是整个3d场景，Threejs 中称为 Scene。 出于两点考虑: (1) Scene 是整个场景的根元素 (2) 为了保持简洁性，我们希望 `react3d.Scene` 也承载创建 canvas 的作用。由于这些特殊性，我们把 Scene 当做独立于 `react3d.Object3D` 以外的另一个基础类：

```jsx
import React from "react";
import {Scene as ThreeScene} from "three"

class Scene extends React.Component {
	constructor(props){
		super(props);
		const {width, height, style} = props;
		this.obj = new ThreeScene();
		this.canvas = document.createElement("canvas");
		this.canvas.width = width;
		this.canvas.height = height;
		this.canvas.style = style;
	}
	componentDidMount(){
		const box = this.refs.container3d;
		box.appendChild(this.canvas);
	}
	componentWillUnmount(){
		const box = this.refs.container3d;
		box.removeChild(this.canvas);
	}
	render(){
		const {width, height, style} = this.props;
		return <div ref="container3d">{this.props.children}</div>
	}
	getChildContext() {
	    return {
	    	parent: this.obj
	    };
	}
}

Scene.childContextTypes = {
  	parent: React.PropTypes.object
};
export default Scene
```


#### **Camera**

3d场景的另一个核心组件是Camera，可以直接继承 `react3d.Object3D`:

```jsx
import {PerspectiveCamera} from "three";
import Object3D from "Object3D.jsx";

class Camera extends Object3D {
	objContructor(props){
		const {fov, aspect, near, far, x, y, z} = props;
		const camera = new PerspectiveCamera(fov, aspect, near, far);
		return camera;
	}

}
export default Camera;
```

#### **Light**

渲染3d场景的另一个重要概念是灯光，我们以 PointLight 为例，显然它也可以直接继承`react3d.Ojbect3D`:

```jsx
import Object3D from "Object3D.jsx";
import {PointLight as ThreePointLight} from "three";

class PointLight extends Object3D {
	objContructor(props){
		const {color, intensity, distance, decay, x, y, z} = props;
		const light = new ThreePointLight( color, intensity, distance, decay);
		light.position.set( x, y, z);
		return light;
	}
}
export default PointLight;
```

我们可以顺着这个思路，定义其他常见的3d组件，例如 Box，Sphere 等。 so far, so good!


## 四、 Renderer 的设计

前面的组件中缺少了重要的一环，就是渲染器 renderer。 当然我们是故意为之的，因为这里有点麻烦……

在 Threejs 中，渲染器（Renderer）用来直接绘制 canvas，出于惯性思维，我们很自然想把 renderer 当做某种最外层的概念，包住整个Scene （毕竟我们是要渲染整个Scene嘛！）。我们知道 Threejs 中的Renderer定义依赖于scene 和 camera，而camera 和 scene相对于renderer是"内层元素"。 注意React 中对象的传递只能从父元素传向子元素，想要逆向传播的话，我们需要使用ref来进行“显示”的引用。这么表述是有点拗口，我们直接上代码，你可以体会一下，

```jsx

import React from "react"

import {WebGLRenderer} from "three"

import Scene from "Scene.jsx"
import Camera from "Camera.jsx"

class Space extends React.Component {
	constructor(props){
		super(props)
		this.frameId = null
		this.renderFrames = this.renderFrames.bind(this)
	}
	renderFrames(){
		const scene = this.scene;
		const camera = this.camera;
		const webGLRenderer = this.webGLRenderer;
		webGLRenderer.render(scene, camera);
		this.frameId = requestAnimationFrame(this.renderFrames)
	}

	componentDidMount(){
		this.webGLRenderer = new WebGLRenderer({antialias: true, canvas: this.canvas});
		this.renderFrames();
	}

	componentWillUnmount(){
		cancelAnimationFrame(this.frameId);
	}

	render(){
		const {width, height} = this.props
		return <Scene ref={ref => { if(ref){this.scene = ref.obj; this.canvas = ref.refs.canvas;} }} 
			width={width} height={height} style={{display:"block", margin: 0}}>
				<Camera ref={ref => { if(ref){this.camera = ref.obj} }} fov={80} aspect={width/height} near={0.5} far={250} z={150}/>
			</Scene>
	}
}
```

把Renderer当做某种最外层的概念，我们很可能写出类似上面的代码，注意这里，为了是renderer能够获得的scene和camera（当然也包括canvas），我们使用ref把他们从子元素中提取出来，然后在componentDidMount阶段执行渲染操作。显然，ref的存在让代码显得非常啰嗦，封装做得很糟糕，以至于我们还要自己重写 render loop 的常规逻辑（通过 render loop不断重绘canvas，实现动画是threejs中的一个常识）

怎样跳出这个坑？

办法很简单，

（下一篇 [part3](../react3d_3/index.md)）













