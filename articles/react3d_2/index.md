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

首先是整个3d场景，Threejs 中称为 Scene。 由于 Scene 是整个场景的根元素，同时为了保持简洁性，我们希望 `react3d.Scene` 同时承载创建 canvas 的作用。由于这些特殊性，我们把 Scene 当做独立于 `react3d.Object3D` 以外的另一个基础类：

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

这步是比较容易的。


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





（下一篇 [part3](../react3d_3/index.md)）













