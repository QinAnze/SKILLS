---
name: avalonia-glass
title: Avalonia 单板毛玻璃 UI 设计系统（极简 · 纯黑白 · 高流畅 · 非线性）
summary: 在 C# Avalonia 11 桌面端实现极简毛玻璃界面——整窗单块玻璃板、真毛玻璃(acrylic)、无衬线高清无描边字体、文字纯黑/纯白、GPU 合成的高帧率非线性(缓动/弹簧)动画、retained-mode 可视化控件(环形仪表/迷你曲线)。融合 Apple 流体界面与动画审计原则。本 skill 提供范式与原理，代码须按需求理解后生成。
description: >
  当用户要求在 C# Avalonia 11（net8.0）做"玻璃风 / 毛玻璃 / glassmorphism"极简界面时使用：
  透明无边框窗口(窗口级 acrylic 一整块玻璃板)、内容直接布局在单板上不堆小组件块、
  无衬线高清无描边字体、文字纯黑或纯白、GPU 合成的高帧率非线性缓动动画(精确曲线 + 时长预算
  + 中断性 + 物理性)、retained-mode 性能隔离可视化控件。音频后端(NAudio)不在本 skill 范围。
agent_created: true
---

# Avalonia 单板毛玻璃 UI 设计系统（极简 · 纯黑白 · 高流畅 · 非线性）

适用栈：`Avalonia 11.2.x` + `net8.0`，implicit usings 开启。观感：macOS/iOS 风极简磨砂玻璃——
克制、单板、纯黑白文字、流畅且物理自然的非线性动画。

> ⚠️ **代码生成纪律（务必遵守）**：本 skill 给的是**范式与原理**，不是可粘贴的模板。
> 先理解下面 §0 的七条铁律，再依据你当前项目的 `AssemblyName`、主题、圆角、动画曲线**重新生成**
> 对应代码。示例里的颜色、不透明度、时长、缓动类型只是示意，必须按设计需求重新定值，**不要逐字照抄**。

## 0. 设计铁律（七条，全部必须遵守）

1. **单块玻璃板**：整个窗口就是一块玻璃——一个根 `GlassPanel` / `GlassSurface`，所有内容（文字、
   控件、可视化）直接布局在它之上。**禁止**在板上再新添 `GlassCard` 或任何独立的玻璃"小组件块"
   来分区。分区感靠排版（间距 / 对齐 / 留白）、分隔线与透明度实现。
   *（依据：Apple 材料原则——"永不把一块轻透表面叠在另一块轻透表面上，可读性会崩塌"。）*
2. **字体**：无衬线（sans-serif）、高清（保持默认高质量抗锯齿渲染，**不**关抗锯齿）、
   **无描边**（文字不加轮廓 Stroke）。优先系统无衬线字体；`LetterSpacing` 随字号（大标题负、正文近 0）。
3. **颜色**：文字与图标只用**纯黑 `#000000`** 或**纯白 `#FFFFFF`**。层级靠 `Opacity` 区分，
   **不**用灰色、彩色文字。玻璃板的"染色(tint)"可与文字色区分——板可带色，板上文字必须纯黑白。
4. **真毛玻璃**：底层一整块玻璃板靠窗口级 `TransparencyLevelHint="AcrylicBlur"` 实现（系统对窗口
   后方做**适中**模糊）；控件只叠**低不透明度中性 tint**，让模糊透出来。**不要**给控件再叠 `BlurEffect`。
5. **高帧率 + 高流畅**：动画只改 `RenderTransform`（平移/缩放/旋转）与 `Opacity`——这些走合成线程/GPU，
   **绝不**在动画里改 `Width/Height/Margin/Canvas.Left` 等触发布局重排（Measure/Arrange）的属性。
   持续动效用独立 `Control` + 节流 `DispatcherTimer`（~30–60fps，按需 `Start`/`Stop`）；静止时定格、不烧 GPU。
6. **非线性动画**：所有过渡一律用**缓动（Easing / KeySpline，精确曲线）或弹簧**，绝不线性匀速。
   数值变化用 `CubicEaseOut` 补间；手势/可中断动效用可重定向的 `Transition`/弹簧，从中断值继续而非跳变。
7. **目的性动画**：动画必须服务功能（反馈 / 状态指示 / 空间一致 / 防止跳变），不为装饰。
   **高频动作（键盘快捷键、命令面板）干脆不动画**；越常用的动效越短越微妙。

## 1. 工程初始化

`csproj`（`OutputType` 用 `WinExe`；`AssemblyName` 按实际命名，**`avares://` 引用依赖它**）：
```xml
<PropertyGroup>
  <OutputType>WinExe</OutputType>
  <TargetFramework>net8.0</TargetFramework>
  <Nullable>enable</Nullable>
  <ImplicitUsings>enable</ImplicitUsings>
  <BuiltInComInteropSupport>true</BuiltInComInteropSupport>
  <AssemblyName>你的项目名</AssemblyName>
  <RootNamespace>你的项目名</RootNamespace>
</PropertyGroup>
<ItemGroup>
  <AvaloniaResource Include="Assets\**" />
  <PackageReference Include="Avalonia" Version="11.2.1" />
  <PackageReference Include="Avalonia.Desktop" Version="11.2.1" />
  <PackageReference Include="Avalonia.Themes.Fluent" Version="11.2.1" />
  <PackageReference Include="Avalonia.Fonts.Inter" Version="11.2.1" />
</ItemGroup>
```

`App.axaml`：深色主题 + 引入玻璃主题（主题里只放资源与控件模板，不放 `FluentTheme` 之外的全局覆盖）：
```xml
<App xmlns="https://github.com/avaloniaui" RequestedThemeVariant="Dark">
  <Application.Styles>
    <FluentTheme />
    <StyleInclude Source="avares://你的项目名/Styles/LiquidGlassTheme.axaml" />
  </Application.Styles>
</App>
```

## 2. 透明窗口 = 一整块玻璃板（毛玻璃的真正来源）

MainWindow 顶层（桌面端模糊靠系统合成器）：
```xml
<Window ... TransparencyLevelHint="AcrylicBlur"
        ExtendClientAreaToDecorationsHint="True"
        ExtendClientAreaChromeHints="NoChrome"
        ExtendClientAreaTitleBarHeightHint="-1"
        Background="Transparent" Foreground="#FFFFFF">
```
- `TransparencyLevelHint="AcrylicBlur"` → 操作系统对窗口后方做亚克力模糊，即"底层一整块玻璃板"，
  模糊程度适中、由系统决定。这正是毛玻璃质感的唯一真实来源。
- 三个 `ExtendClientArea*` 去掉系统标题栏，交给自绘（含苹果风交通灯）。
- `Background="Transparent"` 让桌面透出；`Foreground="#FFFFFF"` 纯白文字。
- **不要**给单个玻璃控件叠 `BlurEffect`：会"过度模糊"且每个玻璃面吃 GPU。

## 3. 单板结构范式（不堆小组件块）

根容器 = 一个 `GlassPanel`（整窗玻璃板），内部直接排版；**不**用 `GlassCard` 再包一块玻璃分区。
"分区感"用留白、对齐、分隔线（纯白低透明度）、不同内容层实现。

```xml
<local:GlassPanel x:Name="Board" CornerRadius="16" TintOpacity="0.4" Padding="0">
  <Grid RowDefinitions="Auto,*,Auto">
    <!-- 第 0 行：标题栏 + 苹果交通灯 -->
    <!-- 第 1 行：主内容区——文字/可视化直接布局，不另起玻璃块 -->
    <!-- 第 2 行：底部控制栏 -->
  </Grid>
</local:GlassPanel>
```

> 需要视觉分隔，用 `<Separator Background="#20FFFFFF" />`（纯白 12.5% 透明度）或留白与对齐，
> **不要**放第二块 `GlassCard`。材料分层靠"同一块板上的透明度/留白"，不是叠第二块玻璃。

## 4. 玻璃面实现范式（理解后生成，勿照抄）

`GlassSurface` 是纯视觉层（`Control`，`IsHitTestVisible=false`），用 retained-mode `DrawingContext`
画玻璃；画刷按 (tint,opacity,gloss) 缓存，属性变才 `InvalidateVisual`。下列为**范式示意**，
请按项目需求（颜色、圆角、不透明度档位）重新生成：

```csharp
using System;
using Avalonia;
using Avalonia.Controls;
using Avalonia.Media;

namespace 你的项目.Controls;

public class GlassSurface : Control
{
    // 默认 tint 中性灰（不偏蓝）；默认不透明度低，保持半透明质感，让 acrylic 模糊透出
    public static readonly StyledProperty<Color> TintColorProperty =
        AvaloniaProperty.Register<GlassSurface, Color>(nameof(TintColor), Color.FromRgb(40, 40, 44));
    public static readonly StyledProperty<double> TintOpacityProperty =
        AvaloniaProperty.Register<GlassSurface, double>(nameof(TintOpacity), 0.4);
    public static readonly StyledProperty<double> GlossStrengthProperty =
        AvaloniaProperty.Register<GlassSurface, double>(nameof(GlossStrength), 1.0);
    public static readonly StyledProperty<CornerRadius> CornerRadiusProperty =
        AvaloniaProperty.Register<GlassSurface, CornerRadius>(nameof(CornerRadius), new CornerRadius(12));

    public Color TintColor { get => GetValue(TintColorProperty); set => SetValue(TintColorProperty, value); }
    public double TintOpacity { get => GetValue(TintOpacityProperty); set => SetValue(TintOpacityProperty, value); }
    public double GlossStrength { get => GetValue(GlossStrengthProperty); set => SetValue(GlossStrengthProperty, value); }
    public CornerRadius CornerRadius { get => GetValue(CornerRadiusProperty); set => SetValue(CornerRadiusProperty, value); }

    public GlassSurface() { IsHitTestVisible = false; }

    protected override void OnPropertyChanged(AvaloniaPropertyChangedEventArgs change)
    {
        base.OnPropertyChanged(change);
        if (change.Property == TintColorProperty || change.Property == TintOpacityProperty ||
            change.Property == GlossStrengthProperty || change.Property == CornerRadiusProperty)
            InvalidateVisual();
    }

    private Color _cachedTint; private byte _cachedA; private double _cachedGloss = -1;
    private LinearGradientBrush? _bodyBrush; private LinearGradientBrush? _sheenBrush;

    private void EnsureBrushes(Color tint, byte baseA, double gloss)
    {
        if (_bodyBrush != null && _cachedTint == tint && _cachedA == baseA &&
            Math.Abs(_cachedGloss - gloss) < 0.01) return;
        _cachedTint = tint; _cachedA = baseA; _cachedGloss = gloss;

        var top = Lighten(tint, 0.08);
        _bodyBrush = new LinearGradientBrush
        {
            StartPoint = new RelativePoint(0, 0, RelativeUnit.Relative),
            EndPoint = new RelativePoint(0, 1, RelativeUnit.Relative),
            GradientStops = new GradientStops
            {
                new GradientStop(Color.FromArgb(baseA, top.R, top.G, top.B), 0),
                new GradientStop(Color.FromArgb(baseA, tint.R, tint.G, tint.B), 1),
            }
        };
        _sheenBrush = new LinearGradientBrush
        {
            StartPoint = new RelativePoint(0, 0, RelativeUnit.Relative),
            EndPoint = new RelativePoint(0, 0.22, RelativeUnit.Relative),
            GradientStops = new GradientStops
            {
                new GradientStop(Color.FromArgb((byte)(40 * gloss), 255, 255, 255), 0),
                new GradientStop(Color.FromArgb(0, 255, 255, 255), 1),
            }
        };
    }

    public override void Render(DrawingContext context)
    {
        var rect = new Rect(0, 0, Bounds.Width, Bounds.Height);
        if (rect.Width <= 1 || rect.Height <= 1) return;
        var r = CornerRadius.TopLeft;
        var gloss = GlossStrength;
        var baseA = (byte)Math.Clamp(TintOpacity * 255, 0, 255);
        EnsureBrushes(TintColor, baseA, gloss);
        context.DrawRectangle(_bodyBrush, null, rect, r, r);                                  // 磨砂体（透出模糊）
        context.DrawRectangle(null,
            new Pen(new SolidColorBrush(Color.FromArgb((byte)Math.Clamp(34 * gloss, 0, 255), 30, 30, 34)), 1),
            rect, r, r);                                                                       // 发丝描边（中性、不发光）
        context.FillRectangle(_sheenBrush!, rect, (float)r);                                   // 顶部柔光（静态）
    }

    private static Color Lighten(Color c, double amt)
    {
        byte Mix(double v) => (byte)Math.Clamp(v + (255 - v) * amt, 0, 255);
        return Color.FromArgb(c.A, Mix(c.R), Mix(c.G), Mix(c.B));
    }
}
```

> 想更"湿"：把 `TintOpacity` 降到 0.3，而不是去叠 GPU 模糊。记住——模糊来自窗口级 acrylic，
> 控件只负责低不透明度 tint 与描边。

## 5. 字体规范（无衬线 · 高清 · 无描边）

- **无衬线 + 系统优先**：优先系统无衬线（`"Segoe UI, Inter, sans-serif"` 或 `avares` 引入的无衬线
  TTF）；不混用衬线体。系统字体已带光学尺寸与字距微调，除非有理由否则不覆盖。
- **高清**：Avalonia 文本默认高质量抗锯齿（子像素/灰度），**不要**设
  `TextOptions.TextRenderingMode="Aliased"`；高 DPI 自动清晰。
- **无描边**：文字不加 `Stroke` / `Outline`；强调与发光用「透明度」或「纯白多层偏移辉光」（非轮廓描边）实现。
- **字距随字号（LetterSpacing 不是定值）**：大标题用**负**字距（字越大越显散），正文近 `0`；
  用 `FontWeight` 建立层级（字重比单纯加大字号更省空间）。
- 层级分级用 **FontWeight + LetterSpacing + Opacity**，不靠颜色。

```xml
<!-- 大标题：负字距、紧行高 -->
<TextBlock FontSize="28" FontWeight="SemiBold" LetterSpacing="-0.5" .../>
<!-- 正文：近 0 字距 -->
<TextBlock FontSize="13" LetterSpacing="0" .../>
```

## 6. 颜色规范（纯黑 / 纯白）

- 文字：纯白 `#FFFFFF`（深板）/ 纯黑 `#000000`（浅板）；本项目深色玻璃用纯白。
- 次要文字：纯白 + `Opacity="0.6"`（**不是**灰 `#9A9AA2` 之类的中间色）。
- **禁用**：灰色文字、金棕/蓝/青等彩色文字、彩色 accent 文字。
- 玻璃 tint（板染色）可保留动态取色，但**板上文字恒为纯黑白**。

设计令牌（去色，纯黑白）：
```xml
<Color x:Key="GlassText">#FFFFFF</Color>           <!-- 主文字：纯白 -->
<!-- 次要文字：直接用 GlassText + Opacity 降级，不要定义灰色 -->
<SolidColorBrush x:Key="GlassForeground" Color="{StaticResource GlassText}" />
<SolidColorBrush x:Key="HairlineBrush" Color="#20FFFFFF" />   <!-- 分隔线：纯白 12.5% -->
```

## 7. 运动系统：高帧率 + 非线性（核心章节）

### 7.1 缓动决策表（选对曲线，再映射 Avalonia）
| 场景 | 缓动 | 精确曲线（cubic-bezier） | Avalonia 写法 |
| --- | --- | --- | --- |
| 进入 / 退出（entrance/exit） | ease-out | `0.23, 1, 0.32, 1` | `Easing="CubicEaseOut"` 或 `KeySpline="0.23,1,0.32,1"` |
| 屏内移动 / 形变（A→B） | ease-in-out | `0.77, 0, 0.175, 1` | `Easing="CubicEaseInOut"` 或 `KeySpline="0.77,0,0.175,1"` |
| 抽屉 / 底部 sheet | ease-drawer | `0.32, 0.72, 0, 1` | `KeySpline="0.32,0.72,0,1"` |
| 悬停 / 颜色变化 | ease | `0.4, 0, 0.2, 1`（或 `CubicEaseOut`） | `Easing="CubicEaseOut"` |
| 匀速持续（跑马灯 / 进度条） | linear | — | `Easing="LinearEase"`（仅此类用） |

> **铁律**：UI 上**绝不用 `ease-in`**（起步慢、延迟用户注视的那一刻）；默认就是 `ease-out`。
> 内置弱曲线不够"有劲"，用上表强曲线（存成共享 token，别散落五个近似 cubic-bezier）。

### 7.2 时长预算（UI 动画一律 < 300ms）
| 元素 | 时长 |
| --- | --- |
| 按钮按压反馈 | 100–160ms |
| Tooltip / 小 popover | 125–200ms |
| 下拉 / 选择 | 150–250ms |
| 模态 / 抽屉 | 200–500ms |

> 频率越高，动效越短；命令快捷键、命令面板**不动画**（Raycast 风格——正确做法是零动画）。

### 7.3 物理性与起源
- **绝不用 `scale(0)`**（现实中没有东西凭空出现）：用 `scale(0.9–0.97)` + `opacity:0` 进入。
- **从触发源缩放**：popover/菜单从触发它的按钮长出（`RenderTransformOrigin` 设到触发器），
  不是从自身中心——否则空间关系断裂。
- **按压反馈**：`:pressed` 时 `Scale(0.97)`、~160ms `ease-out`，且**在按下瞬间**给反馈（不是释放时）。
- 列表项按压、卡片等非交互元素不要乱加 hover 动效（高频命中区要克制）。

### 7.4 中断性与速度（流体界面的关键）
- **动画从中断值继续，不从目标值重启**：重定向时读元素当前在屏值（presentation value）作为起点，
  否则可见跳变。Avalonia 的 `Transitions`（声明式）天然从中断态重定目标——优先用它，而非手写
  keyframe 每帧重绘（keyframe 重定向会从头重启）。
- **手势 / 可中断动效用弹簧**：弹簧携带速度，反向时平滑（无"砖墙"感）。Avalonia 用
  `ElasticEase`/`BackEase` 近似反弹，或用 `Transitions` + `Easing` 达到可中断重定向。
- 2D 运动拆成独立 X/Y 弹簧/过渡（X、Y 速度不同，单弹簧会失同步）。

### 7.5 性能铁律（高帧率的根）
- **只动 `RenderTransform` 与 `Opacity`**——走合成线程/GPU，不触发布局重排。
- **禁止**动画 `Width/Height/Margin/Padding/Top/Left/Canvas.Left`（每帧触发 Layout+Paint+Composite，必卡）。
- **禁止 `Transition`/动画 `All` 属性**（等价 `transition: all`，会动到非 GPU 属性）。
- 过渡期 `Blur` 模糊半径 **< 20px**（重模糊昂贵）。
- 用 Avalonia 声明式 `Transitions`（预知运动）做状态动画；用 `DispatcherTimer` + 手写 `Render`
  做动态/手势/持续动效。

### 7.6 频率原则
- 100+ 次/天（键盘快捷键、命令面板开关）：**不动画**。
- 数十次/天（hover、列表导航）：移除或大幅削减。
- 偶尔（模态、抽屉、提示）：标准动画。
- 罕见/首次（引导、成功庆祝）：可加 delight。
> 最强修复往往不是"加动画"，而是**删动画**。

### 7.7 无障碍（Reduced Motion / Transparency）
- **`prefers-reduced-motion`**：用**短 cross-fade（透明度/颜色）替代位移与弹簧**，去掉弹性/overshoot；
  不是"全关反馈"。在 Avalonia 中检测系统偏好后，对位移动画降级为纯 `Opacity` 过渡（~200ms ease）。
- **`prefers-reduced-transparency`**：抬高玻璃不透明度、降低模糊（更实/更霜）。
- **`@media (hover: hover) and (pointer: fine)` 等价**：触摸设备不触发 hover 动效（tap 会误触 hover）。

```csharp
// 伪代码：读取系统偏好后分支
bool reduce = 读取系统 reduced-motion;          // 平台 API（Win: SystemParametersInfo / 注册表）
var entrance = reduce
    ? new Animation { Duration = TimeSpan.FromMilliseconds(200), Easing = new LinearEase(),
                      /* 仅 Opacity 0→1 */ }
    : new Animation { Duration = TimeSpan.FromMilliseconds(280), Easing = new CubicEaseOut(),
                      /* Opacity + Translate(Y) */ };
```

### 7.8 Avalonia 实现范式（非线性 + 高帧率）

**XAML 声明式过渡（可中断、推荐用于状态动画）**：
```xml
<Border>
  <Border.Transitions>
    <Transitions>
      <TransformTransition Property="RenderTransform" Duration="0:0:0.28">
        <TransformTransition.Easing>
          <CubicEaseOut />
        </TransformTransition.Easing>
      </TransformTransition>
      <DoubleTransition Property="Opacity" Duration="0:0:0.28" Easing="CubicEaseOut" />
    </Transitions>
  </Border.Transitions>
</Border>
<!-- 精确曲线用 KeySpline（等价 cubic-bezier 控制点） -->
<Animation Duration="0:0:0.28" FillMode="Forward">
  <Animation.SpeedRatio>1</Animation.SpeedRatio>
  <KeyFrame Cue="0%"   Setter="{Setter Opacity, 0}">
    <KeyFrame.KeySpline>0.23,1,0.32,1</KeyFrame.KeySpline>
  </KeyFrame>
  <KeyFrame Cue="100%" Setter="{Setter Opacity, 1}">
    <KeyFrame.KeySpline>0.23,1,0.32,1</KeyFrame.KeySpline>
  </KeyFrame>
</Animation>
```

**代码动画（数值补间，理解后改写）**：
```csharp
var anim = new Animation
{
    Duration = TimeSpan.FromMilliseconds(280),
    Easing = new CubicEaseOut(),          // 或 new SplineEase{...} / KeySpline.Parse("0.23,1,0.32,1")
    Children =
    {
        new KeyFrame { Cue = new Cue(0), Setters = { new Setter(OpacityProperty, 0d) } },
        new KeyFrame { Cue = new Cue(1), Setters = { new Setter(OpacityProperty, 1d) } },
    }
};
await anim.RunAsync(control);
```

**持续动效（频谱/旋转/进度）——独立 Control + 节流定时器 + retained-mode**：
```csharp
// 后台建变换，定时器只改 Angle / 平移，走 GPU 合成
RotateTransform _rt = new(); ctrl.RenderTransform = _rt;
var _t = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(33) }; // ~30fps
_t.Tick += (_, _) => { _rt.Angle = (_rt.Angle + step) % 360; };  // 仅改 transform，不触重排
// 仅在需要（播放/特效开）时 Start()，停止即 Stop()；数值变化用 CubicEaseOut 补间而非硬切
```

## 8. 可视化控件范式（retained-mode，吸收 flat-meters）

做实时数据可视化（环形仪表 / 迷你曲线 / 进度）时，按单板范式直接布局在根 `GlassPanel` 上：
- **RingGauge**：`Control`，`Render` 画背景整环 + 按值(0..100)扫过的弧 + 端点高光点；内部
  `DispatcherTimer`(~60fps) 对"显示值"做 `CubicEaseOut` 补间（进入场从 0 缓动到首值）；
  `Stroke` 接负载语义色。
- **Sparkline**：`Control`，`Render` 画历史折线 + 渐变填充；仅 `Values` 变化才重绘；
  `Max` 固定（如 100）或直接取数据上限。
- **状态色**：低绿 / 中琥珀 / 高红，阈值按指标定（如 CPU 60/85、内存 70/88）；
  `StatusColor(pct,warn,hot)` 返回对应 `Color`。**注意**：负载语义色只用于环/曲线的描边与填充，
  **绝不**用于文字（文字恒纯黑白，见 §6）。
- **采集**：`DispatcherTimer` 1s 节流采样；历史用定长 `List` 环形缓冲（容量 ~60）；
  `PerformanceCounter`(CPU/磁盘) + `kernel32.GlobalMemoryStatusEx`(内存) + `NetworkInterface`
  字节差值(网络)，全部 try/catch 兜底。
- **性能隔离**：retained-mode（画刷缓存，仅属性变才 `InvalidateVisual`）；持续动效用独立
  `Control` + 节流定时器，静止定格不烧 GPU；只改 `RenderTransform`/重绘，不触重排。

## 9. 按钮风格
- **窗口控制（关闭 / 最小化 / 最大化）**：固定**苹果红黄绿**三色灯（见下方交通灯样式）。
- **功能按钮**：苹果玻璃风（`GlassButton` 半透明）或 WinUI 风（`Fluent` accent）二选一；
  但按钮上的**文字仍纯黑白**。
- 按压反馈：`RenderTransform` `Scale(0.97)`、~160ms `ease-out`，按下瞬间触发（§7.3）。

macOS 交通灯（固定苹果风）：
```xml
<Style Selector="Button.mac">
  <Setter Property="Width" Value="13" /><Setter Property="Height" Value="13" />
  <Setter Property="CornerRadius" Value="7" /><Setter Property="BorderThickness" Value="0" />
  <Setter Property="Focusable" Value="False" /><Setter Property="Cursor" Value="Hand" />
</Style>
<!-- close #FF5F57 / min #FEBC2E / max #28C840；glyph 默认透明，:pointerover 才显形 -->
```

## 10. 设计令牌（去色，纯黑白）
```xml
<Color x:Key="GlassText">#FFFFFF</Color>
<SolidColorBrush x:Key="GlassForeground" Color="{StaticResource GlassText}" />
<SolidColorBrush x:Key="HairlineBrush" Color="#20FFFFFF" />   <!-- 分隔线：纯白 12.5% -->
<!-- 动画曲线 token（按 §7.1）：在代码里以常量/静态字段复用，避免散落 -->
```
// 代码侧动画 token 示例（复用，不重复手写）：
// static readonly Easing EaseOut = new CubicEaseOut();   // ≈ cubic-bezier(0.23,1,0.32,1)
// static readonly Easing EaseInOut = new CubicEaseInOut();

## 11. 代码生成纪律（重申）
不照抄；理解 §0 七条铁律后，按本项目的 AssemblyName、主题、圆角、动画曲线与阈值**重新生成**代码。

## 12. 优化提示词（针对 C# / Avalonia，可直接复制给 AI 生成）

**中文版**：
```
请用 C# + Avalonia 11（net8.0）实现一个极简毛玻璃桌面界面，严格遵守以下约束：
1. 整窗只有一块玻璃板（一个根 GlassPanel），所有内容直接布局其上，禁止在板上新增任何玻璃
   小组件块（如 GlassCard）来分区；分区用间距、对齐、留白和纯白低透明度(#20FFFFFF)分隔线。
2. 窗口用 TransparencyLevelHint="AcrylicBlur" 实现真正的毛玻璃（系统对窗口后方做适中模糊），
   玻璃控件只叠低不透明度(≈0.4)的中性 tint，让模糊透出；不要给控件加 BlurEffect。
3. 字体用无衬线、高清（保持默认抗锯齿）、无描边的系统字体（Segoe UI/Inter）；
   LetterSpacing 随字号（大标题负、正文近0）；文字与图标颜色只用纯黑 #000000 或纯白 #FFFFFF，
   层级靠 Opacity 区分，禁止灰色或彩色文字。
4. 动画必须高帧率、高流畅、非线性：只改 RenderTransform 与 Opacity（走 GPU 合成，不触布局重排）；
   进入/退出用 ease-out（≈cubic-bezier(0.23,1,0.32,1)），屏内移动用 ease-in-out，UI 时长 <300ms；
   优先用 Avalonia 声明式 Transitions（可中断重定向），持续动效用独立 Control + 节流 DispatcherTimer
   (30–60fps)；数值变化用 CubicEaseOut 补间，手势/可中断动效用弹簧/Transition，禁止线性匀速；
   高频动作（键盘快捷键/命令面板）不动画；尊重 prefers-reduced-motion（降级为短 cross-fade）。
5. 窗口控制按钮（关闭/最小化/最大化）用苹果红黄绿三色灯；功能按钮可用苹果玻璃风或 WinUI 风。
6. 实时可视化（如需）用 retained-mode 独立 Control（环形仪表/迷你曲线），CubicEaseOut 数值补间，
   负载语义色只用于描边/填充、绝不用于文字；采集用节流定时器 + 环形缓冲。
7. 不要直接照抄模板，理解上述原理后按本项目的 AssemblyName、主题色、圆角与动画曲线生成代码。
```

**English version**：
```
Implement a minimal frosted-glass desktop UI in C# + Avalonia 11 (net8.0). Strictly follow:
1. ONE glass slab (a single root GlassPanel); lay out all content directly on it. Do NOT add any
   extra glass widget blocks (e.g. GlassCard) to section it — use spacing, alignment, whitespace,
   and low-opacity pure-white (#20FFFFFF) dividers instead.
2. TransparencyLevelHint="AcrylicBlur" for real frosted glass (OS blurs behind the window, moderate).
   Glass controls only overlay a low-opacity (~0.4) neutral tint; do NOT add BlurEffect to controls.
3. Fonts: sans-serif, high-DPI crisp (keep default antialiasing), NO stroke (system font preferred,
   Segoe UI/Inter). LetterSpacing scales with size (negative on large, ~0 on body). Text/icons use
   ONLY pure black #000000 or pure white #FFFFFF; use Opacity for hierarchy, never grey/colored text.
4. Animations: high-FPS, smooth, NON-LINEAR. Animate ONLY RenderTransform and Opacity (GPU composited,
   no reflow). Entrances/exits ease-out (~cubic-bezier(0.23,1,0.32,1)), on-screen moves ease-in-out,
   UI durations <300ms. Prefer Avalonia declarative Transitions (interruptible re-targeting); persistent
   effects use a dedicated Control + throttled DispatcherTimer (30–60fps). Tween values with CubicEaseOut;
   use springs/Transitions for gesture/interruptible motion — never linear. No animation on high-frequency
   actions (shortcuts, command palette). Respect prefers-reduced-motion (degrade to short cross-fade).
5. Window controls (close/min/max) use macOS red/yellow/green; function buttons may be Apple-glass or WinUI.
6. Realtime visualizations (if any): retained-mode dedicated Controls (ring gauge / sparkline), CubicEaseOut
   value tweening; load-semantic colors ONLY on strokes/fills, never on text; throttle sampling + ring buffer.
7. Do not copy templates verbatim — understand the principles and generate code for this project's
   AssemblyName, theme, corner radius, and easing curves.
```

## 13. 验证

1. `dotnet build -v q` 应 **0 错误 0 警告**（Avalonia 警告当错误看）。
2. 视觉自查靠截图交给用户/多模态模型——**当前模型通常读不了渲染图**，别假装能看图自检。
3. 专项核对清单：
   - 窗口后方桌面是否被**适度模糊**透出（真毛玻璃成立）；
   - 是否**只有一块玻璃板**、无额外玻璃小组件块；
   - 字体是否**无衬线、无描边**；`LetterSpacing` 是否随字号；
   - 文字是否**纯黑/纯白**、无灰色/彩色；
   - 动画是否**流畅且非线性**（有缓动曲线 / 可中断）、帧率高、无重排卡顿；
   - 高频动作是否**没**多余动画；`prefers-reduced-motion` 是否降级。
