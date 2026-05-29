# 手势特效切换器

操作看不懂的,实操我发在抖音了
https://v.douyin.com/ZRGM4rz-kgM/

基于 **MediaPipe Hands** 的手势识别 Web 应用，通过摄像头实时捕捉手部动作，驱动四种不同的视觉特效模式。

## 技术栈

| 技术 | 用途 |
|------|------|
| [MediaPipe Hands](https://developers.google.com/mediapipe/solutions/vision/hand_landmarker) | 手部 21 个关键点实时识别 |
| WebGL / WASM | GPU 加速推理，SIMD 优化 |
| Canvas 2D | 特效渲染、粒子系统、笔画绘制 |
| getUserMedia | 摄像头采集 |

MediaPipe Hands 会返回每只手的 **21 个三维关键点**（x, y, z），所有手势判断都基于这些关键点之间的相对位置关系。

## 如何使用

1. 用浏览器直接打开 `index.html`（推荐 Chrome / Edge）
2. 允许摄像头权限
3. 屏幕左上角会显示模式切换面板，用食指指向对应行并停留 **0.8 秒** 即可切换
4. 确保手部在光线充足的环境下，手掌朝向摄像头

> **注意**：必须以 HTTP 服务方式运行（`npx serve .` 或 VS Code Live Server），直接用 `file://` 打开可能无法访问摄像头。

## 四种模式

### Mode 1：粒子特效

| 手势 | 效果 |
|------|------|
| 单手伸出四指 | 在手掌位置绽放心形烟花 |
| 双手同时伸出四指 | 屏幕中央汇聚 300 个粒子形成巨大爱心，短暂停留后爆炸 |

### Mode 2：取景器

| 手势 | 效果 |
|------|------|
| 双手拇指和食指捏合 | 四个指尖框出一个矩形区域；区域内恢复彩色，区域外变为黑白 |

### Mode 3：3D 粒子牵引

| 手势 | 效果 |
|------|------|
| 五指张开 | 手掌位置持续生成彩色 3D 粒子（上限 300 个） |
| 仅伸出食指 | 食指尖产生引力场，吸引附近粒子跟随指尖移动 |
| 双手食指尖靠近 + 粒子聚集过半 | 粒子爆破散射 |

粒子带加速度和惯性，甩手时有"飞出去"的手感。

### Mode 4：隔空手写

| 手势 | 效果 |
|------|------|
| 拇指和食指捏合 | 在空中书写/绘画 |
| 食指尖悬停在笔画上 1.5 秒 | 进入移动模式，拖动整条笔画 |
| 双食指缩放（不捏合） | 缩放所有笔画 |
| 伸出拇指 + 食指 + 小指（三指） | 所有笔画消散为粒子 |
| 右上角工具栏悬停 | 换颜色、橡皮擦、调节笔刷大小 |

右侧工具栏从上到下依次为：收起按钮、笔/橡皮切换、6 种颜色、笔刷大小滑块。

## 手势判断原理

代码中通过关键点（landmarks）坐标关系判断手势：

- **手指伸出**：指尖 y 坐标小于该手指第二关节 y 坐标（`hand[8].y < hand[6].y`）
- **捏合**：拇指尖与食指尖的欧几里得距离 < 40px（`Math.hypot(tx - ix, ty - iy) < 40`）
- **三指手势**：拇指外展 + 食指向上 + 小指向上 + 中指/无名指向下

## 文件结构

```
.
├── index.html          # 主程序（手势逻辑 + Canvas 渲染 + UI 控制器）
├── lib/
│   ├── camera_utils.js # 摄像头采集封装（Camera 类）
│   └── hands.js        # MediaPipe Hands 解决方案封装（Hands 类）
└── models/
    ├── hands.binarypb                      # MediaPipe 计算图
    ├── hands_solution_packed_assets_loader.js
    ├── hands_solution_packed_assets.data   # 手掌检测模型（打包）
    ├── hands_solution_wasm_bin.js/.wasm    # WASM 引擎（非 SIMD）
    ├── hands_solution_simd_wasm_bin.js/.wasm # WASM 引擎（SIMD）
    └── hand_landmark_full.tflite           # 手部关键点模型
```

## 浏览器兼容

- Chrome / Edge 87+（完整支持 WebGL + SIMD WASM）
- Firefox 89+
- Safari 15.4+
- 需要 HTTPS 或 localhost 才能访问摄像头
