# 类似淘票票 选座功能（svg）
最近项目需要做一个可以选座订票的功能。上网搜了一下没找到一个现成的库所以就自己写了一个粗糙的版本，日后有机会再完善。

### 需求
- 根据数据显示座位分布
- 可拖动 放大缩小 点击选座

### 技术栈
- [hammerjs](http://hammerjs.github.io/)
- svg

### TODO
- 拖动边界
- 对焦捏放。

## 效果

![捏放](https://raw.githubusercontent.com/Hecoffee/TicketMap/master/image/pinch.gif)

![拖动](https://raw.githubusercontent.com/Hecoffee/TicketMap/master/image/panmove.gif)

## 实现
手势库hammer 提供了多种手势以及详细的配置，可以查看它的文档进行相关设置。

核心代码如下，详细代码看index
#### html
```
<div>
    <svg>....</svg>
</div>
```

#### js
```
  var svgTarget = document.querySelector('svg')
  var svgHtml = document.querySelector('.svg-html')

  //  计算 自适应 居中
  transform.translateX = Math.round((svgHtml.clientWidth - svgTarget.clientWidth) / 2)
  transform.translateY = Math.round((svgHtml.clientHeight - svgTarget.clientHeight) / 2)
  var defaultTransform = `translate(${transform.translateX}px, ${transform.translateY}px) scale(${transform.scale})`
  svgTarget.style.transform = defaultTransform

  //  初始化 hammer对象
  var svgHam = new Hammer(svgHtml)
  svgHam.get('pinch').set({ enable: true })  // 返回pinch识别器 设置 可捏放 （放大缩小手势） 默认不监听
  // svgHam.add(new Hammer.Pinch({ threshold: 0 }))
  // svgHam.add(new Hammer.Pan({ threshold: 0, pointers: 0 }))
  svgHam.get('pan').set({ direction: Hammer.DIRECTION_ALL }) // 返回pan识别器 设置拖动方向为 所有方向

  // 监听 捏放 手势
  svgHam.on('pinchstart pinchmove', (e) => {
    var { scale, translateX, translateY } = transform
    scale *= e.scale
    scale = scale >= 4.5 ? 4.5 : scale
    scale = scale <= 0.5 ? 0.5 : scale
    transform.scale = scale
    svgTarget.style.transform = `translate(${translateX}px, ${translateY}px) scale(${scale})`
  })
  svgHam.on('pinchend', (e) => {
    if (transform.scale > 1.5) return false
    transform.scale = 0.5 // 捏放小于 1.5倍时 重置图像
    transform.translateX = Math.round((svgHtml.clientWidth - svgTarget.clientWidth) / 2)
    transform.translateY = Math.round((svgHtml.clientHeight - svgTarget.clientHeight) / 2)
    svgTarget.style.transform = defaultTransform
  })

  //  PC端无法模拟 捏放手势 用 双击代替
  svgHam.on('doubletap', (e) => {
    var { scale, translateX, translateY } = transform
    if (scale < 1.5) {
      scale = 1.5
      transform.scale = scale
      svgTarget.style.transform = `translate(${translateX}px, ${translateY}px) scale(${scale})`
    } else {
      transform.scale = 0.5
      transform.translateX = Math.round((svgHtml.clientWidth - svgTarget.clientWidth) / 2)
      transform.translateY = Math.round((svgHtml.clientHeight - svgTarget.clientHeight) / 2)
      svgTarget.style.transform = defaultTransform
    }
  })

  // 监听 拖动 手势
  svgHam.on('panstart panmove', (e) => {
    var { scale, translateX, translateY } = transform
    if (scale < 1) return false // 必须先捏放到1倍才可 进行拖动
    var y = translateY + e.deltaY
    var x = translateX + e.deltaX
    svgTarget.style.transform = `translate(${x}px, ${y}px) scale(${scale})`
  })
  svgHam.on('panend', (e) => {
    if (transform.scale < 1) return false
    transform.translateY += e.deltaY
    transform.translateX += e.deltaX
    console.log(e.deltaY, e.deltaX, transform.translateX, transform.translateY)
  })
```
