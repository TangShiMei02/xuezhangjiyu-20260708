# 🎓 傥学长的时空胶囊 — 学长寄语系统

一个NFC触发的"时空胶囊"寄语网页。当学弟用手机扫描贴在书桌下的NFC标签时，打开 `jiyu.ilikeyou.ccwu.cc`，体验一场充满仪式感和科技感的"学长寄语"之旅。

## ✨ 功能流程

1. **加载页** — 旋转进度条 + 粒子动画，营造仪式感
2. **时空胶囊解锁** — 3D CSS 胶囊，点击后发光裂开
3. **学长现身** — 全息投影风格头像 + 打字机效果寄语
4. **记忆碎片** — 横向滑动的卡片（座位档案/桌面印记/音乐/照片/生存指南）
5. **传承仪式** — 烟花庆祝 + 生成可保存的座位传承证书
6. **跨届寄语墙** — Twikoo 评论系统 + 时空信箱
7. **彩蛋** — 时段彩蛋/隐藏基地/摇一摇毒鸡汤

## 🚀 快速开始

```bash
# 安装依赖
npm install

# 开发模式运行（热更新）
npm run dev

# 构建生产版本
npm run build
```

构建后的文件在 `dist/` 目录，可直接部署到任何静态托管服务。

## ⚙️ 配置修改

所有可配置内容集中在 `src/config.js` 中，直接修改即可：

### 学长信息
```js
senior: {
  name: '傥学长',       // 你的名字
  major: '交通工程',    // 专业
  grade: '2025级',      // 年级
  avatar: '/assets/avatar.jpg',  // 头像路径
  signature: '...',     // 个性签名
  message: ['...'],     // 寄语内容（每行一个数组元素）
}
```

### 座位信息
```js
seat: {
  campus: '东丽校区',
  building: '南X院',
  dormitory: 'X号楼',
  room: 'XXX',
  position: '靠窗第二个座位',
  period: '2025.09 — 2026.07',
}
```

### 记忆碎片
```js
memories: {
  deskMark: '...',           // 桌面印记描述
  musicId: '123456789',      // 网易云音乐歌曲ID（留空不显示）
  photos: ['photo1.jpg'],    // 照片文件名（放到 public/assets/photos/）
  tips: ['tip1', 'tip2'],    // 生存指南
}
```

### Twikoo 评论系统
```js
twikoo: {
  envId: 'https://your-twikoo-api.vercel.app',  // 替换为你的 Twikoo 后端地址
  path: window.location.pathname,
}
```

## 🎨 自定义素材

- **头像**: 替换 `public/assets/avatar.jpg`
- **照片**: 放到 `public/assets/photos/` 目录，并在 `config.js` 中引用
- **音效**: 放到 `src/assets/sounds/`（可选，当前版本未启用）

## ☁️ 部署

将 `dist/` 目录的所有文件上传到你的服务器即可。

### 推荐部署方式

- **Vercel**: 连接 GitHub 仓库，自动部署
- **Cloudflare Pages**: 拖拽上传 dist 目录
- **自己的服务器**: 用 Nginx/Caddy 托管 dist 目录

NFC 标签的 URL 设置为 `https://jiyu.ilikeyou.ccwu.cc`（用你自己的域名）。

## 📱 彩蛋

| 触发方式 | 效果 |
|---------|------|
| 晚上11点后访问 | 提示早点休息 |
| 凌晨1点后访问 | 催睡觉 😏 |
| 考试周期间 | 考试加油鼓励 |
| 任意位置连点5次 | 打开"学长の秘密基地" |
| 摇一摇手机 | 随机显示毒鸡汤/暖心话 |

## 📄 技术栈

- Vue 3 + Vite
- Tailwind CSS
- CSS 3D 动画
- Canvas 粒子特效
- Twikoo 评论系统
- html2canvas（证书保存）

## 📝 License

MIT
