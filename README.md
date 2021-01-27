# 可视化页面编辑器


[Github：项目地址，有兴趣可以看看！](https://github.com/wsydxiangwang/Visualization-Page)

可视化页面编辑器，听起来可望不可即是吧，先来张动图观摩观摩一番！



要实现这种功能，在网上参考了很多资料，貌似一无所获，千篇一律，就是没有适合我方需求的！

这里的亮点在于对象和引用、数据结构的实现、来一一讲解一下其中的细节吧！！

## 拖拽实现

主要拖拽事件

### 


### 边界处理、角度计算

核心变量：

- `this.isPush`：是否把拖拽元素push到页面数据中
- 


`dragover`: 移动事件

```js
// 'view-content': 整体的盒子，直接 push
// 'item': 内部的单个元素，需计算位置，进行变换操作

if (name == 'view-content') {
    if (!this.isPush) {
        this.index = this.view.length
        this.isPush = true
        this.view.push(defaultData)
    }
} else if (name == 'item') {

    let target = e.target
    let [ y, h, curIndex ] = [ e.offsetY, target.offsetHeight, target.dataset.index ]
    let direction = y < (h / 2) // 计算鼠标处于当前元素的位置，来决定拖拽元素的前/后

    if (!this.isPush) {
        // Push to Top or Bottom
        if (direction) {
            if (curIndex == 0) {
                this.view.unshift(defaultData)
            } else {
                this.view.splice(curIndex, 0, defaultData)
            }
        } else {
            curIndex = +curIndex + 1
            this.view.splice(curIndex, 0, defaultData)
        }
    } else {
        // Moving
        if (direction) {
            var i = curIndex == 0 ? 0 : curIndex - 1
            var result = this.view[i]['status'] == 2
        } else {
            var i = +curIndex + 1
            var result = this.view.length > i && this.view[i]['status'] == 2
        }
        
        // 拖拽元素是否需变换位置
        if (result) return

        const temp = this.view.splice(this.index, 1)
        this.view.splice(curIndex, 0, temp[0])
    }
    this.index = curIndex   // 拖拽元素位置
    this.isPush = true      // 进入则push，即true
} else {
    this.index = 0
}
```

- `2`: 拖拽元素移动状态， 

### 内容块拖拽实现

因时间关系，这里偷懒了，使用了一个较为完美的拖拽插件

后面想自己去实现这个功能，研究了一会，还是有一定的复杂程度，因时间关系还是不折腾了

具体实现


## 组件划分

中间视图，右边编辑区，为一套一套的，不愧是一套一套！

```js
.
├── components
|   ├── Edit           ## 右边编辑
|   |    ├── Info       # 基本信息
|   |    ├── Image      # 广告图
|   |    ├── Product    # 商品
|   |    └── Index      # 管理编辑组件的信息
|   └── View           ## 中间视图
|   |    ├── Banner     # 轮播图
|   |    ├── Images     # 广告图
|   |    └── Product    # 产品列表
└── page
    └── index          ## 主页面
```

## 数据结构的定义

实现一个鲜艳且具有扩展性的功能，那么定义一个符合条件的数据结构是必不可少！

同时也能决定你的代码量、可维护性！

当然，还是得由自身的逻辑思维来决定！



##
这里，我觉得最亮眼的处理方法为：组件之间的传值，只需一次！

借用了对象是引用类型，所存空间一致的方法，视图会跟随编辑的变化而变化，无需过多操作！

最后一步

数据的操作处理

## 评价一下

这只能算是一个简陋粗略版，具体核心实现了，剩下的问题都已不再是问题了，有兴趣可以自己去琢磨琢磨，完善起来！
