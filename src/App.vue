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

// 检查两个圆是否重叠
const isCirclesOverlap = (x1, y1, x2, y2, radius) => {
  const distance = Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
  // 如果两个圆心的距离小于两倍半径，说明重叠
  return distance < (radius * 2);
};

// 获取一个不重叠的随机位置
const getNoOverlapPosition = (existingPositions, radius, padding) => {
  let attempts = 0;
  const maxAttempts = 100; // 最大尝试次数，防止无限循环
  
  while (attempts < maxAttempts) {
    const posX = Math.random() * (app.screen.width - 2 * radius - padding * 2) + (padding + radius);
    const posY = Math.random() * (app.screen.height - 2 * radius - padding * 2) + (padding + radius);
    
    // 检查是否与现有的圆重叠
    let hasOverlap = false;
    for (const pos of existingPositions) {
      if (isCirclesOverlap(posX, posY, pos.x, pos.y, radius)) {
        hasOverlap = true;
        break;
      }
    }
    
    if (!hasOverlap) {
      return { x: posX, y: posY };
    }
    
    attempts++;
  }
  
  // 如果尝试多次都找不到合适的位置，可能需要调整参数
  console.warn('Could not find non-overlapping position after', maxAttempts, 'attempts');
  return null;
};

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
  const existingPositions = [];
  const padding = 40;

  for (let i = 0; i < count; i++) {
    const circle = new Graphics();
    
    // 获取不重叠的位置
    const position = getNoOverlapPosition(existingPositions, radius, padding);
    
    // 如果找不到合适的位置，跳过这个圆
    if (!position) {
      console.warn('Skipping circle due to no available space');
      continue;
    }
    
    existingPositions.push(position);
    
    circle.circle(position.x, position.y, radius);
    circle.fill(0xaaff00);

    const container = new Container();
    container.addChild(circle);
    container.interactive = true;
    container.hitArea = new Circle(position.x, position.y, radius + 10);
    circleList.push(container);

    container.on('tap', () => {
      // 判断当前点击的是否是闪烁的图形
      if (container.children[0].tint === 0xaaff00) {
        // 创建点击动画效果
        createClickAnimation(position.x, position.y, radius);
        
        // 点击以后清除当前图形
        app.stage.removeChild(container);
        circle.destroy();
        circleCount--;
        // circleList 中找到当前点击的图形
        const index = circleList.indexOf(container);
        circleList.splice(index, 1);
        // 如果所有的图形都隐藏了，则显示庆祝动画后重新生成一批
        if (circleCount === 0) {
          createCelebrationAnimation();
        } else {
          setRandomBlink(circleList);
        }
      }
    })
    app.stage.addChild(container);
  }

  setRandomBlink(circleList);
}

// 创建点击特效动画
const createClickAnimation = (x, y, radius) => {
  // 创建爆炸效果圆圈
  const burst = new Graphics();
  burst.circle(x, y, radius);
  burst.lineStyle(3, 0xffff00);
  burst.fill(0xffff00, 0.5);
  app.stage.addChild(burst);

  // 创建粒子效果
  const particles = [];
  const particleCount = 8;
  for (let i = 0; i < particleCount; i++) {
    const particle = new Graphics();
    particle.circle(x, y, 4);
    particle.fill(0xffff00);
    
    // 设置粒子的初始属性
    const angle = (Math.PI * 2 * i) / particleCount;
    particle.vx = Math.cos(angle) * 5;
    particle.vy = Math.sin(angle) * 5;
    particle.alpha = 1;
    
    app.stage.addChild(particle);
    particles.push(particle);
  }

  // 动画效果
  let scale = 1;
  const animate = () => {
    scale += 0.2;
    burst.scale.set(scale);
    burst.alpha -= 0.1;

    // 更新粒子位置
    particles.forEach(particle => {
      particle.x += particle.vx;
      particle.y += particle.vy;
      particle.alpha -= 0.05;
    });

    if (burst.alpha <= 0) {
      app.stage.removeChild(burst);
      particles.forEach(p => app.stage.removeChild(p));
      app.ticker.remove(animate);
      burst.destroy();
      particles.forEach(p => p.destroy());
    }
  };

  app.ticker.add(animate);
};

// 创建烟花粒子
const createFirework = (x, y, color) => {
  const particles = [];
  const particleCount = 30;
  
  for (let i = 0; i < particleCount; i++) {
    const particle = new Graphics();
    particle.circle(x, y, 2);
    particle.fill(color);
    
    // 随机角度和速度
    const angle = Math.random() * Math.PI * 2;
    const speed = 2 + Math.random() * 3;
    particle.vx = Math.cos(angle) * speed;
    particle.vy = Math.sin(angle) * speed;
    // 添加重力效果
    particle.gravity = 0.1;
    particle.alpha = 1;
    
    app.stage.addChild(particle);
    particles.push(particle);
  }

  return particles;
};

// 创建庆祝动画
const createCelebrationAnimation = () => {
  const colors = [0xff0000, 0x00ff00, 0x0000ff, 0xffff00, 0xff00ff, 0x00ffff];
  let fireworks = [];
  let animationTime = 0;
  const animationDuration = 2000; // 动画持续2秒

  // 创建多个烟花
  const createNewFirework = () => {
    const x = Math.random() * app.screen.width;
    const y = Math.random() * (app.screen.height / 2);
    const color = colors[Math.floor(Math.random() * colors.length)];
    const particles = createFirework(x, y, color);
    fireworks.push(...particles);
  };

  // 初始创建几个烟花
  for (let i = 0; i < 3; i++) {
    createNewFirework();
  }

  const animate = (delta) => {
    animationTime += delta.elapsedMS;

    // 每隔一段时间创建新的烟花
    if (animationTime % 500 < 50) {
      createNewFirework();
    }

    // 更新所有粒子
    fireworks.forEach(particle => {
      particle.x += particle.vx;
      particle.y += particle.vy;
      particle.vy += particle.gravity;
      particle.alpha -= 0.01;
    });

    // 移除消失的粒子
    fireworks = fireworks.filter(particle => {
      if (particle.alpha <= 0) {
        app.stage.removeChild(particle);
        particle.destroy();
        return false;
      }
      return true;
    });

    // 动画结束
    if (animationTime >= animationDuration) {
      app.ticker.remove(animate);
      fireworks.forEach(particle => {
        app.stage.removeChild(particle);
        particle.destroy();
      });
      // 动画结束后重新开始游戏
      initTapCircle(15, 50);
    }
  };

  app.ticker.add(animate);
};

try {
  // await app.init({ width: 640, height: 360 })

  document.body.appendChild(app.canvas);

  // 获取显示的宽度，高度
  console.log(app.screen.width, app.screen.height);

  initBg(40, true);
  initTapCircle(15, 5)
} catch (error) {
  console.log(error);
}

// 当页面大小变化时，重新初始化背景
window.addEventListener('resize', () => {
  initBg(40, true)
  initTapCircle(15, 50)
})
// 页面翻转（平板浏览器上），重新初始化背景
window.addEventListener('orientationchange', () => {
  initBg(40, true)
  initTapCircle(15, 50)
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
