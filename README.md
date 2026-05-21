# 洛克王国通行证拼团结算工具

一个用于计算《洛克王国》通行证「传火」拼团方案的网页工具。自动生成最优传火链条、计算每人应付金额与转账指令，并支持一键导出图片分享。

> 在线体验：将本仓库部署到 GitHub Pages 后访问 `https://<用户名>.github.io/<仓库名>/`

## 功能特性

- 🔥 **传火链条自动编排**：根据每人需要的精灵（精灵1 / 精灵2 / 都行），通过 DFS + 回溯算法生成满足「精灵交替」规则的最优链条。
- 👥 **车头 / 车尾指定**：支持手动锁定链条的起点和终点。
- 🤝 **好友关系优先**：录入好友关系后，链条会优先把好友排在相邻位置（副券需互为好友才能赠送），并对非好友相邻段给出提示。
- 💰 **费用自动结算**：根据档次（普通 68 元 / 高级 128 元）自动计算总价、人均费用、差额平摊和每人的转账指令（向谁转多少、原因）。
- 📊 **方案卡片可视化**：每位成员一张卡片，包含角色（源头 / 中间人 / 车尾）、所购通行证、应收应付明细、好友状态。
- 🖼️ **一键导出图片**：基于 `html2canvas` 将完整方案导出为 PNG 图片，方便群内分享。
- 💾 **配置导入 / 导出**：将当前人员、好友关系、档次等配置保存为 JSON 文件，下次直接导入复用，跨设备 / 长期方案管理无忧。
- 📱 **响应式适配**：手机端紧凑单列、PC 端宽屏双列布局（左侧表单常驻，右侧结果卡片），自动跟随系统深色模式。

## 玩法背景

洛克王国通行证支持「副券」机制：购买通行证后可向好友赠送副券激活其通行证，副券价格远低于通行证原价。多人组成「传火链条」交替购买可让每人均摊到更低的价格。

价格表（默认）：

| 档次 | 通行证 | 副券 |
| --- | --- | --- |
| 普通 | 68 元 | 40 元 |
| 高级 | 128 元 | 80 元 |

链条规则：相邻两人必须购买不同的精灵通行证（即「精灵交替」），且需互为好友才能赠送副券。

## 技术栈

- [Vue 3](https://vuejs.org/) + [Vite 6](https://vitejs.dev/)
- [Element Plus](https://element-plus.org/) UI 组件库
- [html2canvas](https://html2canvas.hertzen.com/) 导出图片

## 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 本地预览构建产物
npm run preview
```

## 部署

仓库已配置 GitHub Actions（`.github/workflows/deploy.yml`），推送到 `main` 分支后自动构建并发布到 GitHub Pages。构建时通过 `BASE_URL` 环境变量注入仓库名作为子路径。

启用步骤：

1. 在 GitHub 仓库 **Settings → Pages** 中将 Source 设为 **GitHub Actions**。
2. 推送至 `main` 分支即可触发部署。

## 项目结构

```
.
├── index.html                # 入口 HTML
├── vite.config.js            # Vite 配置（含 BASE_URL）
├── src/
│   ├── main.js               # Vue 应用入口
│   ├── App.vue               # 主界面（人物录入、好友关系、结果展示）
│   └── utils/
│       └── calculator.js     # 链条搜索 & 费用结算核心算法
├── public/                   # 静态资源
└── .github/workflows/        # GitHub Pages 自动部署
```

## 核心算法

`src/utils/calculator.js` 中的 `generatePlan()` 是核心入口：

1. **校验输入**：人数 ≥ 2，车头 / 车尾各最多一人且不能相同。
2. **构建好友邻接表**：基于双向好友对。
3. **DFS 搜索链条**：从车尾向车头回溯，每步选择满足「精灵交替」的候选人；以「好友相邻段数量」为评分函数贪心剪枝，保留最高分链条。
4. **结算费用**：源头垫付通行证 + 一张副券，其余人各付一张副券给上家，再按差额向源头平摊。

## 配置文件格式

点击页面顶部「导出」按钮可下载当前配置 JSON。文件结构如下：

```json
{
  "version": 1,
  "exportedAt": "2026-05-21T12:00:00.000Z",
  "tier": "normal",
  "elfName1": "迪莫",
  "elfName2": "亚比",
  "people": [
    { "id": 1, "name": "张三", "needElf": "elf1", "isHead": true,  "isTail": false },
    { "id": 2, "name": "李四", "needElf": "elf2", "isHead": false, "isTail": false },
    { "id": 3, "name": "王五", "needElf": "any",  "isHead": false, "isTail": true  }
  ],
  "friendships": [[1, 2], [2, 3]]
}
```

- `tier`：档次，可选 `normal` / `premium`
- `needElf`：精灵需求，可选 `elf1` / `elf2` / `any`
- `friendships`：双向好友对，元素为 `[idA, idB]`，使用 `people` 中的 `id` 进行关联

导入时会进行结构校验与 id 合法性检查，非法字段会被忽略而非中断流程。

## License

仅供学习交流使用。
