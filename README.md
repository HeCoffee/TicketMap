# 类似淘票票 选座功能（svg）

> 最近项目需要在移动端做一个可以选座订票的功能。上网搜了一下没找到一个现成的库可以解决需求，所以就自己写了一个粗糙的版本。

### 需求
- 前端展示座位分布
- 可拖动 放大缩小 点击选座

### 技术栈
- svg
- [hammerjs](http://hammerjs.github.io/)

## 效果图

![捏放](https://user-gold-cdn.xitu.io/2018/11/15/16716a6e9ff860b0?w=261&h=279&f=gif&s=3473571)

![拖动](https://user-gold-cdn.xitu.io/2018/11/15/16716a607cf8620f?w=263&h=278&f=gif&s=2077812)

详细效果可以去[demo](https://hecoffee.github.io/TicketMap/)那体验一下

## svg
座位分布图的SVG是UI画好后导出来，并且通过后端接口返回整个SVG标签以及里面的内容。
前端只要发请求获得SVG并且插入就好了。插入后需要将已订购的座位置黑，无法选择。

## hammerjs
[hammerjs](http://hammerjs.github.io/)是一个手势库，提供了tap, doubletap, press, pan, swipe, pinch以及 rotate等多种手势事件，同时也提供丰富的自定义配置可以让你完成产品各(nao)种(dong)各(da)样(kai)的需求。

### 用法

``` javaScript
let hammertime = new Hammer(myElement, myOptions);
hammertime.on('pan', function(ev) {
	console.log(ev)
})
```

## html 结构
箱子svg-box用以捏放（缩放）

svg用以偏移
```` html
<div class="ticket-map">
    <div class=svg-box>
        <svg>.....</svg>
    </div>
</div>
````

## 初始化
设置一个变量用以记录手势操作后的属性变化

``` javascript
// 记录位移变量
let transform = {
    svgScale: 0.5, // svg 默认缩放
    scale: 1, // svg-box 缩放
    maxScale: 7, // svg-box 最大缩放
    minScale: 1,  // svg-box 最小缩放
    translateX: 0, // svg X轴偏移
    translateY: 0, // svg Y轴偏移
    minX: 0, // svg 最小X轴偏移
    maxX: 0, // svg 最大X轴偏移
    minY: 0, // svg 最小Y轴偏移
    maxY: 0  // svg 最大Y轴偏移
}
```
因为UI提供的SVG是1000*715 略大，为了适应屏幕 作了svgScale: 0.5的缩放。

获取到SVG后需要进行居中 并且 计算出拖动边界（minX/Y maxX/Y）


```
    let svgTarget = document.querySelector('svg')
    let svgBox = document.querySelector('.svg-box')
    transform.translateX = Math.round((svgBox.clientWidth - svgTarget.clientWidth) / 2) // 垂直居中时的X偏移
    transform.translateY = Math.round((svgBox.clientHeight - svgTarget.clientHeight) / 2) // 垂直居中时的Y偏移

    transform.minX = transform.translateX - svgBox.clientWidth / 4
    transform.maxX = transform.translateX + svgBox.clientWidth / 4
    transform.minY = transform.translateY - svgBox.clientHeight / 2.5
    transform.maxY = transform.translateY + svgBox.clientHeight / 2.5
    svgTarget.style.transform = `translate(${transform.translateX}px, ${transform.translateY}px) scale(${transform.svgScale})`

```
以svg-box的width的4分之一，以及height的2.5分之一作为svg的X Y轴偏移量的极限，以免svg被拖动出屏幕之外。可以根据实际SVG调整分数。

### 手势配置

```` javascript
  //  初始化 hammer对象
  var svgHam = new Hammer(svgBox)
  svgHam.get('pinch').set({ enable: true })  // 返回pinch识别器 设置 可捏放 （放大缩小手势） 默认不监听
  svgHam.get('pan').set({ direction: Hammer.DIRECTION_ALL }) // 返回pan识别器 设置拖动方向为 所有方向
````
监听svg-box，以免svg移动后手指点击不到触发不了事件


### 捏放事件

``` javaScript
svgHam.on('pinchstart pinchmove', (e) => {
        let { scale, maxScale, minScale } = transform
        scale *= e.scale
        scale = scale >= maxScale ? maxScale : scale
        scale = scale <= minScale ? minScale : scale
        transform.scale = scale
        svgBox.style.transform = `scale(${scale})`
      })
```
设置maxScale，minScale以免无限大或者无限小

**· 注意这里缩放的是box而不是svg本身，否则拖动后再缩放就会发现整个SVG都跑偏了**

### 拖动事件


```
function checkXY(x, y) {
    let { minX, minY, maxX, maxY } = transform
    x = x > maxX ? maxX : x
    x = x < minX ? minX : x
    y = y > maxY ? maxY : y
    y = y < minY ? minY : y
    return {
        x,
        y
    }
}

svgHam.on('panstart panmove', (e) => {
    let { scale, translateX, translateY, svgScale } = transform
    let y = translateY + e.deltaY / scale
    let x = translateX + e.deltaX / scale
    let validXY = checkXY(x, y)
    svgTarget.style.transform = `translate(${validXY.x}px, ${validXY.y}px) scale(${svgScale})`
})

svgHam.on('panend', (e) => {
    let { scale, translateX, translateY} = transform
    let y = translateY + e.deltaY / scale
    let x = translateX + e.deltaX / scale
    let validXY = checkXY(x, y)
    transform.translateY = validXY.y
    transform.translateX = validXY.x
})
```
偏移结束后更新偏移值，因为hammer提供的event事件的偏差值deltaY/deltaX是你手指点击初始位置以及移动后的差值。

偏移量 / scale 可以有效的控制放大后的拖动速度，否则放大后一拖动，整个SVG就跑走了。

### 选座
做好了上述效果，选座就简单了 直接一个点击事件就好了


``` javascript
document.querySelector('.svg-box').addEventListener('click', selectSeat)
// svg 点击事件 选座
function selectSeat (e) {
  if (e.target.tagName !== 'circle') return false
  e.target.style.fill = e.target.style.fill === 'red' ? '#ccc' : 'red' // 选中的座位变成红色
  // do something...
}
```

### [完整代码](https://github.com/Hecoffee/TicketMap)
------------

> 写在最后

不是十分完美，不过也满足现在的需求，希望能给大家带来一点启发。不足的地方，也望各位大神指点迷津。
