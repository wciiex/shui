# 喝水提醒项目优化建议

## 1. 代码结构优化

### 问题
- 所有代码混在一个 HTML 文件中，难以维护
- 没有模块化结构

### 建议
```
shui/
├── index.html          (精简版，仅包含 HTML 结构)
├── css/
│   └── style.css       (提取样式)
├── js/
│   ├── app.js          (主应用)
│   ├── state.js        (状态管理)
│   ├── storage.js      (本地存储)
│   ├── audio.js        (声音/振动)
│   └── antiburn.js     (防烧屏)
└── README.md
```

---

## 2. 功能优化

### A. 自定义间隔时间
**现状：** 硬编码 5 分钟
**改进：** 允许用户设置提醒间隔
```javascript
// 添加设置面板
const PRESETS = {
  light: 30 * 60 * 1000,    // 30分钟
  normal: 5 * 60 * 1000,    // 5分钟
  frequent: 2 * 60 * 1000   // 2分钟
};
```

### B. 提醒消息自定义
**现状：** 硬编码中文消息
**改进：** 支持多语言和自定义提醒文本

### C. 统计功能增强
- 添加每日统计
- 记录喝水次数趋势
- 显示今日喝水量估算

### D. 深色/浅色主题切换
**现状：** 仅支持深色
**改进：** 添加主题切换按钮

---

## 3. 性能优化

### A. 减少重排和重绘
```javascript
// 优化前：频繁修改 DOM
$timer.textContent = fmtTime(r);
$count.textContent = state.count;
$elapsed.textContent = fmtActive(elapsedMs());

// 优化后：批量更新
function batchRender() {
  const updates = {
    timer: fmtTime(remainingMs()),
    count: state.count,
    elapsed: fmtActive(elapsedMs())
  };
  requestAnimationFrame(() => {
    Object.assign($timer.textContent = updates.timer, ...);
  });
}
```

### B. 优化定时器
- 使用 `requestAnimationFrame` 替代某些 `setTimeout`
- 避免频繁创建/销毁定时器

### C. 减少存储写入
**现状：** 每 5 秒写一次 localStorage
**改进：** 改为每 30 秒或仅关键操作时写入

---

## 4. 可访问性改进

### A. 添加 ARIA 标签
```html
<div role="status" aria-live="polite" aria-label="剩余时间">
  <div class="metric" id="timer">05:00</div>
</div>
```

### B. 键盘控制
- 支持 Space 键启动/暂停
- 支持 R 键重置
- 支持 Esc 键关闭弹窗

### C. 色彩对比度
- 确保符合 WCAG AA 标准
- 不仅依靠颜色区分状态

---

## 5. 用户体验改进

### A. 设置界面
```html
<!-- 新增设置按钮 -->
<button id="btnSettings" class="btn ghost">⚙️</button>

<!-- 设置面板 -->
<div class="sheet settings-sheet">
  <h3>设置</h3>
  <label>
    提醒间隔（分钟）
    <input type="number" id="intervalInput" min="1" max="120" value="5">
  </label>
  <label>
    <input type="checkbox" id="enableSound"> 启用声音
  </label>
  <label>
    <input type="checkbox" id="enableNotification"> 启用浏览器通知
  </label>
  <label>
    <input type="checkbox" id="enableVibration"> 启用振动
  </label>
</div>
```

### B. 提醒预览
- 在设置时预览声音和振动效果

### C. 一键分享
- 分享喝水提醒链接给朋友

---

## 6. 错误处理改进

### 现状问题
```javascript
try {
  audioCtx = audioCtx || new (window.AudioContext || window.webkitAudioContext)();
} catch (e) {}  // 静默失败
```

### 改进
```javascript
try {
  audioCtx = audioCtx || new (window.AudioContext || window.webkitAudioContext)();
} catch (e) {
  console.warn('AudioContext 初始化失败:', e);
  state.audioAvailable = false;
}
```

---

## 7. 文档和注释

### 添加
- JSDoc 注释
- README.md 使用说明
- 功能说明文档

---

## 8. 测试建议

```javascript
// 单元测试
describe('fmtTime', () => {
  it('应该正确格式化时间', () => {
    expect(fmtTime(300000)).toBe('05:00');
    expect(fmtTime(65000)).toBe('01:05');
  });
});

// 集成测试
describe('提醒系统', () => {
  it('应该在指定时间触发提醒', (done) => {
    // ...
  });
});
```

---

## 9. SEO 和 Meta 优化

```html
<meta name="description" content="一个轻量级的喝水提醒应用，帮助你养成良好的饮水习惯">
<meta name="keywords" content="喝水,提醒,健康,应用">
<meta property="og:title" content="喝水提醒">
<meta property="og:description" content="...">
```

---

## 10. 部署和CI/CD

### 建议
- 添加 GitHub Actions 自动化测试
- 添加 ESLint 代码检查
- 压缩 HTML/CSS/JS
- 生成 PWA 清单

---

## 优先级排序

### 高优先级（立即做）
1. ✅ 代码模块化分离
2. ✅ 自定义提醒间隔
3. ✅ 添加设置界面

### 中优先级（1-2周）
4. 多语言支持
5. 主题切换
6. 性能优化

### 低优先级（持续改进）
7. 统计图表
8. PWA 支持
9. 社交分享

