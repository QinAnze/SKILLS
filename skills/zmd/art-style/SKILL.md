---
name: industrial-techwear
version: 4.0.0
title: 工业机能风 · 界面设计语言
description: >
  泛化的工业机能风视觉语言：暖中性灰打底、全站只有一个彩色（柠檬黄系）、
  文字层级靠不透明度、外壳圆角配内核切角、几何单色图标、
  出现快停留久消失干脆。含规则、基调、开关、令牌、组件要点与反模式。
tags: [ui-design, art-direction, design-tokens, industrial-techwear]
agent_created: true
---

# 工业机能风 · 界面设计语言

冷硬、高信息密度的通用界面语言。用于监控台、控制面板、仪表盘、设置窗、管理后台；
不用于儿童/消费/医疗类亲和产品与多品牌色营销页。

## 哲学

1. **整体性 > 单体冲击力。** 多一个彩色、多一套圆角，整体就散。
2. **黑白灰关系可推敲。** 暖中性灰（`R ≥ G ≥ B`）久看不发青；层级用可枚举的不透明度，不靠感觉调。
3. **强调色是信息，不是装饰。** 它唯一职责是"这里要看一眼"。
4. **信息密度是美学。** 秩序来自栅格、对齐、等宽数字，不是来自删信息。
5. **装饰 = 被结构切掉的形状。** 画大，让容器裁掉，只露一段弧或一块条纹。

## 规则

**色彩**
- 全站彩色通道 = **1**。
- 灰阶暖中性 `R ≥ G ≥ B`；禁大面积 `#000`/`#FFF`，禁冷蓝灰。
- 文字层级 = 实色 + 不透明度三档（1.0 / .55 / .40），不用灰字。
- 无渐变；强调色面积 ≤5%。
- 高纯度原色（品红 / 黄 / 春绿）**只作装饰带与分类编码**，不作功能色、不铺面。
- 状态换色**整组换**，不许半红半绿。

**形态**
- 外壳圆角（8/10/12）+ 内核方块**切右下角**，两者必须共存。
- 切角 = 边长 × 0.16–0.20（34px→6、38px→7、44px→8）；同视图 ≤2 档、方向统一；标签切左上角以示区分。
- 圆角随高度变化：矮容器取 **高/2**（全圆端），高容器取 **≈高/5**，两者反相关。
- 描边只有 0 / 1px / 2px（2px 仅给图标）；虚线只作表格分隔。
- 暗底零阴影（靠明度差分层）；浅底单层 `0 2px 6px rgba(0,0,0,.07)`。
- 装饰必须被容器裁剪。

**排版 / 图标 / 动效**
- 无衬线 + CJK 回退；字重 400–700，主力 500/600。
- 同视图内**最大字号 ≥ 最小 × 2**；小字号配字距 1–3px。
- 数值一律 `tabular-nums`，与单位**基线对齐**（不垂直居中）。
- 图标 24×24、单色、吃 `currentColor`；线性/实心同组二选一；禁 emoji 与系统图标。
- 复杂物体抽象成 2–3 个几何，左右对称，不画写实细节。
- 出现 ≤12%、停留 ≥50%、消失 ≤3%；hover 用位移或亮度，**不用缩放**；回弹曲线只用于缩放。

## 两种基调

| | **暗色** | **浅色** |
|---|---|---|
| 底 / 面 | `#1A1A1C` / `#202022`·`#262628`·`#2A2A2D` | `#ECECEB` / `#F6F6F4`·`#FFFFFF` |
| 分层 | 明度差 | 描边 + 轻阴影 |
| 强调色 | `#C6CA4C`（降彩度） | `#FFDF00`（提纯度） |
| 文字 | `#FFF` + 不透明度 | `#4A4A46` + 不透明度 |
| 底纹 | 无 | 点阵 18px |

强调色随底色自适应：暗底上高纯度黄刺眼发脏，浅底上降饱和黄糊进灰底。同一色系的两端。

### 高纯度原色族（点缀的来源）

低饱和是底，高纯度是点——对比就从这里来。原色饱和度接近 100%，**只在极小面积出现**。

| 名称 | 原色 | 降饱和后 → 功能位 | 装饰带配比 |
|---|---|---|---|
| 品红 | `#FF00F0` | 无功能对应，**只作装饰与分类编码** | 20% |
| 黄 | `#FFFA00` | → 强调色 `#FFDF00`（浅底）/ `#C6CA4C`（暗底） | 68% |
| 春绿 | `#00FFA2` | → 在线色 `#3BB36B` | 12% |

规律：**原色是"未经调和的信号"，功能色是它在当前底色上的可用版本。**
凡是要承载语义（按钮、状态、进度）的位置，用降饱和版；原色只留给装饰带与分类编码。

## 可替换开关

| | |
|---|---|
| 基调：暗 / 浅 | 灰阶色温：暖中性 / 冷灰 |
| 强调色：`#C6CA4C` / `#FFDF00` / `#FFC300` | 强调色配额：点缀级 ≤5% / 区块级 ≤12% |
| 角处理：纯圆角 / 圆角+切角 / 纯切角 | 切角方向：右下（默认）等四角 |
| 底纹：无 / 点阵 / 等高线 / 原色装饰带 | 阴影：无 / 控件级 / 卡片级 |
| 图标：线性 / 实心 | 全局缩放：1.0 / 0.85（紧凑档） |
| 动效档：界面级 ≤360ms / 叙事级 3–10s | 字体族：Inter + CJK 回退 / 任意无衬线 |
| 风格校准：工业科幻 / 战术军事 / 极简工业 / 医疗洁净 | |

## 令牌

| 语义 | 暗 | 浅 | | 语义 | 暗 | 浅 |
|---|---|---|---|---|---|---|
| bg | `#1A1A1C` | `#ECECEB` | | accent | `#C6CA4C` | `#FFDF00` |
| surface | `#262628` | `#F6F6F4` | | on-accent | `#1A1A1C` | `#1A1A1C` |
| raised | `#2A2A2D` | `#FFFFFF` | | info | `#3BA3DC` | `#3BA3DC` |
| line | `#2A2A2C` | `#C9C9C4` | | warn | `#DFA32A` | `#DFA32A` |
| plate（浅块） | `#E9E7E4` | `#DDDDD8` | | danger | `#FF4D4F` | `#E04A3A` |
| ink（深块） | `#141313` | `#3A3A38` | | online | `#3BB36B` | `#3BB36B` |
| text / muted | `#FFF` / `#888` | `#4A4A46` / `#8A8A84` | | decor | `#656363` | `#A9A9A5` |

**尺度**——间距 2/4/6/8/10/12/16/20/28/40（主栅格 4px）· 圆角 0/2/4/6/8/10/12/18/pill · 切角 6/7/8 · 字号 52/26/24/17/16/15/14/13/12/11/9 · 时长 120/150/220/320ms。

**缓动**——位移 `(.65,0,.35,1)` · 标准 `(.22,.61,.36,1)` · 离场 `(.4,0,.6,1)` · 出现 `(0,0,.58,1)` · 消失 `(.42,0,1,1)` · 弹入 `(.175,.885,.32,1.275)`（仅缩放）。

## 组件

| | |
|---|---|
| **按钮** | 高 34 · 圆角 8 · 内距 0/20 · 13px/600。主 = 强调色底 + 深色字（每视图 ≤1 个）；次 = 面上浮 + 1px 描边；**图标方块钮 = 44×44 直角 + 切右下 8px**（最具辨识度）；禁用 `.4`。 |
| **卡片** | 暗：`#262628` + 10px 圆角 + 内距 16/14 + 无阴影。浅：`#FFF` + 1px 描边 + 轻阴影 + 内距 18/20/22。标题下边框 3px 实线。**同容器内不嵌套卡片**，分区靠间距 + 1px 分隔线。 |
| **输入框** | 高 32 · 圆角 6 · 内距 0/10 · 13px。焦点只换边框为强调色 + `0 0 0 2px` 22% 光晕，不放大不变圆角。滑块轨 4px、开关 22×12。 |
| **导航** | 顶栏高 56 · 内距 14/28 · gap 18 · 半透明面 + `blur(6px)` + 下边框。Tab 钮 38×38 圆角 8，激活 = 底色反转 + 强调色图标。Tab 组水平居中。 |
| **表格** | 表头深色条 + 白字 13px/字距 1px，列名前配 16px 圆形计数点。行高 36–44、行距 9、条块 + 1px 描边；**hover 只横移 3px**。数值列右对齐 + 等宽。 |
| **状态色** | 正常 = 强调色 · 告警 = 危险色（整组换）· 无数据 = `--` + 0 进度，不编数字 · 在线/离线 = 绿点带光晕 / 灰点无光。 |
| **标签 / 芯片 / 标识件** | 切角标签：低饱和块 + 12px + 内距 4/16 + 切左上 6px。芯片：高 28 + pill。标识件：圆盘底盘 + 环绕进度弧（**12 点起顺时针**，圆头，线宽 ≈ 直径 ×10%）+ 中心几何图形。 |
| **双色块拼贴** | 浅标签块 + 深数值块零间隙等高。暗底放浅块、浅底放深块——**永远明度反转**。轻量数据块的通用解，替代材质。 |
| **进度** | 环：衬环 → 淡色轨道 → 强调色弧（圆头 + 微光）。线：轨 3px + 强调色填充 + `.45s`。柱状走势：顶部留 16% 保留带，柱宽 ≥1.5px，透明度 ±5% 抖动免死板。 |
| **原色装饰带** | 高 3–4px 满宽实心条，三段配比 ≈ **20 : 68 : 12**（品红 `#FF00F0` / 黄 `#FFFA00` / 春绿 `#00FFA2`），无圆角、无渐变、无间距。只出现在页眉下沿、加载页、区块分隔处；**每视图 ≤1 条**，且不与强调色控件同屏相邻。 |

## 反模式

| ❌ | ✅ |
|---|---|
| 出现第二个彩色 | 彩色通道 = 1 |
| `#000`/`#FFF` 铺底、冷蓝灰当工业灰 | 暖中性灰 `R ≥ G ≥ B` |
| 浅灰字做次级 | 实色 + 不透明度三档 |
| 强调色铺底铺标题 | 只给标识件/进度/激活/状态点 |
| 把品红/春绿用于按钮或状态 | 原色只作装饰带与分类编码 |
| 装饰带与强调色控件贴在一起 | 两者拉开距离，避免两种"黄"打架 |
| 告警只改一个环 | 整组同步换色 |
| 全圆角或全切角 | 外壳圆角 + 内核切角 |
| 圆角与高度无关地乱设 | 矮取 高/2、高取 ≈高/5 |
| 阴影叠三层 | 暗底零阴影、浅底单层 |
| 卡片里套卡片 | 间距 + 1px 分隔线 |
| 14/15/16 挤在一起 | 最大 ≥ 最小 × 2 |
| 全站 Bold / 实时数值不等宽 | 主力 500/600 + `tabular-nums` |
| emoji、系统图标、写实细节 | 24×24 单色几何、描边框 + 填充条 |
| 位移用回弹曲线、hover 缩放 | 位移用 `(.65,0,.35,1)`、hover 横移 3px |
| 容器没就位内容先出现 | 容器动画完成后再淡入 |
| 用材质贴图找质感 | 明度层次 + 裁剪装饰 + 1px 精确描边 |

## 最小落地

```html
<style>
  :root{ --bg:#ECECEB; --card:#FFF; --line:#C9C9C4; --ink:#3A3A38;
         --accent:#FFDF00; --text:#4A4A46; --dim:#8A8A84; --ch:7px;
         --clip:polygon(0 0,100% 0,100% calc(100% - var(--ch)),
                        calc(100% - var(--ch)) 100%,0 100%); }
  body{ margin:0; background:var(--bg); color:var(--text); font-variant-numeric:tabular-nums;
        font:500 14px/1.5 Inter,"Microsoft YaHei UI","PingFang SC",sans-serif; }

  .bar{ display:flex; align-items:center; gap:14px; padding:12px 24px;
        background:rgba(246,246,244,.88); border-bottom:1px solid var(--line);
        backdrop-filter:blur(6px); }
  .logo{ background:var(--ink); color:#fff; padding:6px 12px; font-size:15px;
         letter-spacing:1px; clip-path:var(--clip); }        /* 切角 */
  .logo b{ color:var(--accent); font-weight:500; }           /* 唯一彩色点 */
  .tab{ width:38px; height:38px; background:var(--ink); color:#fff; border:none; cursor:pointer;
        display:grid; place-items:center; clip-path:var(--clip);
        transition:filter .15s,transform .15s; }
  .tab:hover{ filter:brightness(1.35); transform:scale(1.06); }

  .card{ max-width:860px; margin:26px auto; padding:18px 20px 22px; background:var(--card);
         border:1px solid var(--line); border-radius:10px;
         box-shadow:0 3px 12px rgba(0,0,0,.07); }
  .card h2{ margin:0 0 14px; padding:0 4px 8px; font-size:17px; font-weight:500;
            letter-spacing:3px; border-bottom:3px solid var(--ink); }
  .row{ display:grid; grid-template-columns:1fr 130px 150px; align-items:center;
        background:var(--card); margin:7px 0; padding:7px 10px; border:1px solid #E2E2DE;
        box-shadow:0 1px 4px rgba(0,0,0,.07); transition:transform .12s; }
  .row:hover{ transform:translateX(3px); }                   /* hover 位移，不缩放 */
  .row .ic{ width:38px; height:38px; background:var(--ink); color:#fff;
            display:grid; place-items:center; clip-path:var(--clip); }
  .row .num{ text-align:right; font-size:15px; }

  .input{ height:32px; padding:0 10px; background:#fff; color:var(--text);
          border:1px solid var(--line); border-radius:6px; outline:none; }
  .input:focus{ border-color:var(--accent); box-shadow:0 0 0 2px rgba(255,223,0,.22); }
  .btn{ height:34px; padding:0 20px; border:none; border-radius:8px; background:var(--accent);
        color:#1A1A1C; font-weight:600; cursor:pointer; }
  .dot{ width:8px; height:8px; border-radius:50%; background:#B9B9B3; }
  .dot.live{ background:#3BB36B; box-shadow:0 0 0 3px rgba(59,179,107,.22); }

  /* 原色装饰带：三段配比 20 : 68 : 12，高 3px，无圆角无渐变 */
  .band{ height:3px; display:flex; }
  .band i:nth-child(1){ flex:20; background:#FF00F0; }
  .band i:nth-child(2){ flex:68; background:#FFFA00; }
  .band i:nth-child(3){ flex:12; background:#00FFA2; }
</style>

<div class="band"><i></i><i></i><i></i></div>
<header class="bar">
  <div class="logo"><b>//</b> 标题</div>
  <button class="tab">◎</button><button class="tab">▤</button>
  <span style="flex:1"></span><span><i class="dot live"></i> 正常</span>
</header>
<main class="card">
  <h2>数据列表</h2>
  <div class="row">
    <div style="display:flex;gap:12px;align-items:center">
      <span class="ic">▣</span>
      <div><div>条目名称</div>
           <div style="font-size:11px;color:var(--dim)">次级说明</div></div>
    </div>
    <div class="num">11.1%</div><div class="num">1 720 MB</div>
  </div>
  <div style="display:flex;gap:8px;margin-top:14px">
    <input class="input" placeholder="筛选…">
    <button class="btn">确认</button>
  </div>
</main>
```
