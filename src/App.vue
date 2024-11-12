<script setup>
import {
  Application,
  Graphics,
  Container,
  Circle
} from 'pixi.js';
// Create the application helper and add its render target to the page
const app = new Application();
globalThis.__PIXI_APP__ = app;
// 初始化应用，宽度和高度适配页面大小
await app.init({ antialias: true, resizeTo: window });
const initBg = (rectWidth, isMove = false) => {

  // 计算当前宽度需要多少个方块
  const rectCount = Math.ceil(app.screen.width / rectWidth);
  // 计算当前高度需要多少个方块
  const rectHeight = Math.ceil(app.screen.height / rectWidth);

  const rg = new Container({
    isRenderGroup: true
  });

  const totalCount = isMove ? rectCount * 2 : rectCount;
  // 创建方块
  for (let i = 0; i < totalCount; i++) {
    for (let j = 0; j < rectHeight; j++) {
      const rect = new Graphics();
      rect.rect(i * rectWidth, j * rectWidth, rectWidth, rectWidth);
      // 相邻的方格，颜色额填充不同，分别为 #000 和 #fff
      if ((i + j) % 2 === 0) {
        rect.fill(0x000000);
      } else {
        rect.fill(0xffffff);
      }
      rg.addChild(rect);
    }
  }

  app.stage.addChild(rg);
  if (isMove) {
    app.ticker.add((delta) => {
      const speed = 1
      rg.x -= speed;
      if (rg.x < -rectWidth * rectCount) {
        rg.x = 0;
      }
    })
  }
}

// 随机设置其中一个圆形，背景色闪烁
const setRandomBlink = (circleList) => {
    const randomIndex = Math.floor(Math.random() * circleList.length);
    const blinkCircle = circleList[randomIndex].children[0];
    let blinkColor = 0xaaff00;
    let blinkTime = 0;
    const blinkFrequency = 500;
    app.ticker.add((delta) => {
      blinkTime += delta.elapsedMS;
      // console.log(blinkTime, blinkFrequency);
      if (blinkTime >= blinkFrequency) {
        blinkTime = 0;
        blinkColor = (blinkColor === 0xaaff00) ? 0xff00ff : 0xaaff00;
        blinkCircle.tint = blinkColor;
      }
    })
  }

// 在页面中生成可点击圆形图案
const initTapCircle = (radius, count) => {
  let circleCount = count;
  const circleList = [];
  for (let i = 0; i < count; i++) {
    const circle = new Graphics();
    // x, y 使用随机位置, 边缘留 padding 
    // x 的范围 padding + radius ~ app.screen.width - radius - padding
    // y 的范围 padding + radius ~ app.screen.height - radius - padding
    // radius 是圆形的半径
    const padding = 40;
    let posX = Math.random() * (app.screen.width - 2 * radius - padding) + padding;
    let posY = Math.random() * (app.screen.height - 2 * radius - padding) + padding;
    // console.log(posX, posY);
    circle.circle(posX, posY, radius);
    circle.fill(0xaaff00);
    //希望给图形增加变换颜色的滤镜
    // 创建 Container 来代理点击事件
    const container = new Container();
    container.addChild(circle);
    container.interactive = true;
    // 可点击范围要比 circle 大一点
    container.hitArea = new Circle(posX, posY, radius + 10);
    circleList.push(container);

    container.on('tap', () => {
      // 判断当前点击的是否是闪烁的图形
      if (container.children[0].tint === 0xaaff00) {
        // 点击以后清除当前图形
        app.stage.removeChild(container);
        circle.destroy();
        circleCount--;
        // circleList 中找到当前点击的图形
        const index = circleList.indexOf(container);
        circleList.splice(index, 1);
        // 如果所有的图形都隐藏了，则重新生成一批
        console.log(app.stage.children.length);
        if (circleCount === 0) {
          initTapCircle(radius, count);
        } else {
          setRandomBlink(circleList);
        }
      }
    })
    app.stage.addChild(container);
  }

  setRandomBlink(circleList);
}

try {
  // await app.init({ width: 640, height: 360 })

  document.body.appendChild(app.canvas);

  // 获取显示的宽度，高度
  console.log(app.screen.width, app.screen.height);

  initBg(40, true);
  initTapCircle(15, 10)
} catch (error) {
  console.log(error);
}

// 当页面大小变化时，重新初始化背景
window.addEventListener('resize', () => {
  initBg(40)
  initTapCircle(15, 10)
})
// 页面翻转（平板浏览器上），重新初始化背景
window.addEventListener('orientationchange', () => {
  initBg(40)
  initTapCircle(15, 10)
})

</script>

<template>
  <!-- <div>123123123</div> -->
</template>

<style scoped></style>
<style>
body {
  margin: 0;
}
</style>
