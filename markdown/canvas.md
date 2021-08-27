# canvas签名

## 前言

>
 canvas元素在前端日常开发中有着百宝箱般多样的用法，今天这篇文章主要讲解canvas标签的基本使用和如何实现日常开发中常见的canvas签名板，以及进阶轨迹签名轨迹播放的思路，在了解canvas基本用法之余，通过实践有更深的canvas使用了解。
>

## canvas基本用法

canvas标签指定id能让我们通过id找到canvasDom节点,再通过dom.getContext('2d')获取canvas上下文方法和绘画功能。

```javascript
    const canvasDoc = document.getElementById(canvasId)
    const canvasBox = canvasDoc.getContext('2d')

    // 属性定义
    canvasBox.lineWidth = 10         // 直线宽度
    canvasBox.strokeStyle = 'black' // 路径的颜色(gradient-使用渐变填充,pattern-使用纹理填充)
    canvasBox.fillStyle = 'black'  // 填充的颜色(gradient,pattern)
    canvasBox.lineCap = 'round'         // 直线首尾端圆滑
    canvasBox.lineJoin = 'round'     // 当两条线条交汇时，创建圆形边角
    canvasBox.shadowBlur = 1         // 边缘模糊，防止直线边缘出现锯齿
    canvasBox.shadowColor = 'black'  // 边缘颜色
    canvasBox.shadowOffsetX/Y = ''  //阴影
```

### 常用api

* beginPath() - 新路径的开始  
* moveTo(x,y) - 移动画笔到起始点绘制一条线
* lineTo(x,y) - 直线连接到终点
* fill() - 填充
* closePath() - 返回到当前路径的开始
* stroke() - 绘制当前或已经存在的路径
* save() - 保存当前绘制状态

```javascript
// 实现具备填充色的三角形
const canvasDoc = document.getElementById(canvasId)
const canvasBox = canvasDoc.getContext('2d')

canvasBox.beginPath()
canvasBox.moveTo(0,0)
canvasBox.lintTo(20,0)
canvasBox.lintTo(10,10)
canvasBox.closePath()
canvasBox.stroke()
canvasBox.fill()
```

### canvas签名实现

>
canvas签名的实现思路是监听鼠标事件 - onMouseDown，onMouseMove，onMouseUp，通过onMouseDown事件标识绘画开始，onMouseMove事件计算鼠标移动路径，绘制路径。onMouseDown事件标识绘画结束。
>

```javascript
    // 绘画开始
    function beginPath(e, canvasBox) {
        isMove = true
        canvasBox.beginPath()
        canvasBox.save()
    }

    // 绘画过程
    function draw(e, canvasObj) {
        const {canvasBox, canvasDoc} = canvasObj
        if(coordinate && isMove){
            const xy = this.getCoordinate(e, canvasDoc)
            canvasBox.save()
            canvasBox.moveTo(xy?.x,xy?.y)
            canvasBox.lineTo(coordinate.x,coordinate.y)
            canvasBox.stroke()
        }
        coordinate = this.getCoordinate(e, canvasDoc)

    }

    // 绘画结束
    function endDraw(canvasBox) {
        coordinate = null
        isMove = false
    }

    // 计算路径
    function getCoordinate(e, canvasDoc) {
        var x, y
        x = e.clientX - canvasDoc.offsetLeft
        y = e.clientY - canvasDoc.offsetTop // 获取当前x,y坐标
        timer = new Date().getTime() - startTime
        return {
           x,y,timer
        }
    }
```

### 笔画轨迹播放

>
笔画轨迹播放的实现思路大概就是保存所有x轴和y轴的路径数组，再通过setInterval累加索引不断绘制连接x轴和y轴的点。第一次实现的方式通过forEach遍历绘制，结果发现瞬间结果就出来列。
>
```javascript
allCoordinate.forEach(item => {
    canvas2.canvasBox.save()
    canvas2.canvasBox.moveTo(item?.x,item?.y)
    canvas2.canvasBox.lineTo(item?.x,item?.y)
    canvas2.canvasBox.restore()
    canvas2.canvasBox.stroke()
})
```
>
然后改成setInterval方式伪递归实现，不过这种方式存在断点的问题。
>
```javascript
let index = 0
const intervalHandle = window.setInterval(() => {
        canvas2.canvasBox.save()
        canvas2.canvasBox.moveTo(allCoordinate[index]?.x, allCoordinate[index]?.y)
        canvas2.canvasBox.lineTo(allCoordinate[index]?.x,allCoordinate[index]?.y)
        canvas2.canvasBox.restore()
        canvas2.canvasBox.stroke()
        index++
    }
},0)
```

思考会不会是点和点之间相差时间的时间导致的，想要还原原来的笔画速度和轨迹，还需要保存路径点与开始路径点的相差时间,设置setInterval入参时间为路径点与路径点的相差时间能解决这个问题。按照这个思路实现，发现断点的减少了，但是还是存在少量断点。

```javascript
allCoordinate.forEach((item,index) => {
    const intervalHandle = window.setInterval(() => {
        canvas2.canvasBox.save()
        canvas2.canvasBox.moveTo(item?.x, item?.y)
        canvas2.canvasBox.lineTo(item?.x, item?.y)
        canvas2.canvasBox.restore()
        canvas2.canvasBox.stroke()
        clearInterval(intervalHandle)
    },allCoordinate?.[index+1]?.timer - item.timer)
})
```

>
既然问题不是在时间上，那问题应该还是在点数组上，笔画是通过多段路径绘制形成的，保存多段这样的路径数组是不是能解决问题。果然最终通过保存每次onMouseDown鼠标事件开始和onMouseUp鼠标事件结束为一段路径的路径点数组组成的对象，通过每个起点和终点的连续路径绘制不会有断点了。
>

```javascript
let textIndex = 0
let index = 0
const intervalHandle = window.setInterval(() => {
    line = allCoordinate?.[textIndex]?.[index];
    if(line){
        if (index == 0) {
            canvas2.canvasBox.moveTo(line?.x, line?.y)
        }
        canvas2.canvasBox.save()
        canvas2.canvasBox.lineTo(line?.x,line?.y)
        canvas2.canvasBox.restore()
        canvas2.canvasBox.stroke()
        index++
    }else{
        index = 0
        textIndex++
    }
},0)
```
