# view-dxf

这是一个在线打开dxf图纸的插件，欢迎使用，如有不足，欢迎指正。
持续更新中...
* 注：支持所有的框架以及原生的引用方式

## install
安装方式：`npm install view-dxf --save`
引用方式：`import Viewer from 'view-dxf'`

### 需要依赖的第三方库
需要ThreeJS的支持：`npm install three --save`
需要DxfParser的支持: `npm install dxf-parser --save`

## 使用例子

```javascript
let parser = new DxfParser()
let dxfData = parser.parseSync('拿到的dxf文件数据')

// helvetiker_regular.typeface.json 代表的是json格式的字体库数据
// cad-view代表的是你插入dxf的节点id，width代表显示时的宽，height代表显示时的高

let loader = new THREE.FontLoader()
loader.load('/static/lib/fonts/helvetiker_regular.typeface.json', (response) => {
	let cadCanvas = new Viewer(dxfData, document.getElementById('cad-view'), width, height, response, (dxfCallback) => {
		console.log(dxfCallback, '用户操作之后的回调函数')
	})
})
```

### 批注模块

###### 1，按钮功能对应的节点ID:
绘制矩形框：`drawRectId`
绘制云线框：`drawCloudId`
绘制平面框：`drawPlaneId`
绘制箭头：`drawArrowId`
* eg: `<i id="drawRectId" class="iconfont off dxf-el-button icon-xiankuang1"></i>`
* 注：必不可少的有两部分：id="drawRectId" class="off"
* 说明：off为关闭状态时的类名，on为打开状态时的类名

----

###### 2，查询到的批注列表，在画布上显示所需要的数据格式，以及所需要调用的接口为
* 数据格式以及最少需要含有的字段
```javascript
let data = [{annotationId: 320,	// 批注的唯一标识
	content: "批注内容详情",		// ---如果只是使用批注模块，这个参数不要传
	coordinate: {
		drawRectScreenCoord: {
			startX: 38,
			startY: 210,
			endX: 238,
			endY: 402
		},
		drawRectWorldCoord: {
			startX: -24.88748981868235,
			startY: 26.64271682686174,
			endX: 25.490601775931857,
			endY: 48.2797488960321
		},
		type: "drawCloudType"
	},	// 批注操作完成之后callback出来的数据，不用处理同步保存就好
	type: 1,    // 问题类型（1:一般问题，2：严重问题）---如果只是使用批注模块，这个参数不要传
	toRole: 1	// 角色，用来区分绘制时的颜色（1:校对人，2:负责人，3:审核人，4:审定人，5:设计人）
}]
```
* 注：角色所对应的颜色值
	{label: "校对人", value: 1, color: '#0A86FF'},
    {label: "负责人", value: 2, color: '#FF8100'},
    {label: "审核人", value: 3, color: '#0AC99A'},
    {label: "审定人", value: 4, color: '#FC0261'},
    {label: "设计人", value: 5, color: '#A327FF'},
* 调用接口
```javascript
cadCanvas.dxfAnnotationListDrawCtrl(data)
```

----

###### 3，删除单条批注需要调用的接口
* 调用接口
```javascript
cadCanvas.deleteDxfAnnotationCtrl(annotationId)
```

----

###### 4，高亮当前选中的批注需要调用的接口
* 数据格式以及最少需要含有的字段
```javascript
let data = {
	annotationId: 320,	// 批注的唯一标识
	toRole: 1,	// 角色，用来区分绘制时的颜色（1:校对人，2:负责人，3:审核人，4:审定人，5:设计人）
}
```
* 调用接口
```javascript
cadCanvas.selectedDxfAnnotationCtrl(data)
```

----

###### 5，清空画布所需要调用的接口
* 调用接口
```javascript
cadCanvas.sceneRemoveViewerCtrl()
```

----

###### 6，初始化完成的返回数据
* 注：只有初始化完成之后才可以进行批注的增删改查等操作，其中的100.00代表的是初始化的进度到了100%
* 数据格式
```javascript
let data = {
	data: 100.00,
	type: "sceneAddFinishDxf"
}
```

----

###### 7，批注绘制完成的返回数据
绘制矩形框：`drawRectType`
绘制云线框：`drawCloudType`
绘制平面框：`drawPlaneType`
绘制箭头：`drawArrowType`
* 数据格式
```javascript
let data = {
	data: {},	// 和查询批注列表里面的coordinate字段的数据要保持同步
	type: "selectedComponentDxf"
}
```

----

###### 8，移动缩放之后的返回数据
* 数据格式
```javascript
let data = {
	data: {
		moveAndZoom: {
			offsetX: 0,		// X轴偏移量
			offsetY: 0,		// Y轴偏移量
			zoom: 1			// 画布缩放大小，1为不放大也没缩小
		}
	},
	type: "updateScreenPositionDxf"
}
```
* 注：zoom的值是相对于左上角(0,0)为基点进行缩放的

----

###### 9，重新计算当前批注的屏幕位置
* 数据格式
```javascript
let data = [{
	coordinate: {
		drawRectScreenCoord: {
			startX: 38,
			startY: 210,
			endX: 238,
			endY: 402
		},
		drawRectWorldCoord: {
			startX: -24.88748981868235,
			startY: 26.64271682686174,
			endX: 25.490601775931857,
			endY: 48.2797488960321
		},
		type: "drawCloudType"
	},	// 还是之前第一次保存之后的callback数据
}]
```
* 或者直接传入data[0]也可以，只不过传进去什么，返回什么
* 调用接口
```javascript
cadCanvas.pointToScreenPositionCtrl(data, (callback) => {
	console.log(callback)
})
```

----

###### 10,新绘制的批注框与原来绘制的批注框的type都是selectedComponentDxf
* 新绘制的批注框的callback数据格式
```javascript
let data = {
	type: "selectedComponentDxf",
	data: {
		drawRectScreenCoord: {
			startX: 38,
			startY: 210,
			endX: 238,
			endY: 402
		},
		drawRectWorldCoord: {
			startX: -24.88748981868235,
			startY: 26.64271682686174,
			endX: 25.490601775931857,
			endY: 48.2797488960321
		},
		type: "drawCloudType"	// 根据这个type判断是新增批注框，以及批注框的形状
	}
}
```
* 点击之前绘制的批注框的callback数据格式
```javascript
let data = {
	type: "selectedComponentDxf"
	data: {
		name: 341,	// 返回点选的批注框的唯一标识
		type: "Line",
		userData: {
			drawRectScreenCoord: {
				startX: 38,
				startY: 210,
				endX: 238,
				endY: 402
			},
			drawRectWorldCoord: {
				startX: -24.88748981868235,
				startY: 26.64271682686174,
				endX: 25.490601775931857,
				endY: 48.2797488960321
			},
			type: "drawCloudType"	// 根据这个type判断是点选的批注框，以及选中的批注框的形状
		}
	}
}
```

* eg: data.data.type === 'drawCloudType'  // 如果取到对应值，则代表新增
* eg: data.data.userData.type === 'drawCloudType'  // 如果取到对应值，则代表点选

----

###### 11,每次新增之后操作
* 第一步：保存数据库
* 第二步：保存成功之后，再按照数据规则调用第二个接口：cadCanvas.dxfAnnotationListDrawCtrl(data)





### 移动端只使用批注这部分的功能的使用例子

```javascript
// cad-view代表的是你插入dxf的节点id，width代表显示时的宽，height代表显示时的高

let cadCanvas = new Viewer({}, document.getElementById('cad-view'), width, height, {}, (dxfCallback) => {
	console.log(dxfCallback, '用户操作之后的回调函数')
})
```

### 移动端操作方式说明
1，如若在绘制状态，单指即为绘制，否则就是平移
2，双指表示缩放
3，三指表示平移


###### 特殊情况说明：
* 引用方式：`<script src="https://cdn.jsdelivr.net/npm/view-dxf@1.2.6/dist/index.js" type="text/javascript" charset="utf-8"></script>`
* 打包方式：webpack --config webpack.config.js
* 发布方式：npm publish
* 更新方式：npm update view-dxf --save