---
name: math-solve
description: "高中数学题目分析器。当用户发送数学题目（通常是拍照图片）时使用此skill。为每道题生成包含7个分析维度的白色打印友好HTML文件（解题思路、解题过程、知识点拆解、思维方式、易错点、变式题、难度定位），同时维护一个汇总索引页面用于学习诊断。触发词：'做这道题'、'分析这道题'、'这题怎么做'、发送数学题目图片。"
---

# 高中数学题目分析器

为高中数学题目（人教A版·浙江）生成详细的7维分析HTML文件，并维护学习诊断索引。

## 工作流程

### Step 0: 初始化

工作目录：`~/Desktop/shuxue/`

1. 检查 `images/` 子目录是否存在，不存在则 `mkdir -p ~/Desktop/shuxue/images`
2. 检查 `index.html` 是否存在，不存在则使用本文件末尾的 **INDEX_TEMPLATE** 生成
3. 统计已有 `problem_*.html` 文件数量，下一个编号 = 已有数量 + 1（三位数补零：001、002...）

### Step 1: 读取题目

**图片输入（常见）：**
1. 将用户提供的图片复制到 `~/Desktop/shuxue/images/problem_NNN.jpg`
2. 如果原图 >2MB，使用 `sips --resampleWidth 1200 <src> --out <dst>` 压缩
3. 从图片中仔细读取题目内容，完整理解题意

**文字输入：**
- 跳过图片处理，HTML中省略图片区域

### Step 2: 分析题目（7个维度）

#### 维度一：解题思路
- 放在题目图片之后、正式解法之前
- 展示做题前的思考过程：不是标准答案，而是"怎么想到的"
- 关键要素：
  - 审题：提取了哪些关键条件，第一反应是什么
  - 尝试：最初想用什么方法，走到哪一步发现困难或不够优雅
  - 突破：怎么想到了能解决问题的方法，关键洞察是什么
  - 路径选择：如果有多条路，为什么选了这条
- 控制篇幅：3-5个思考节点，每个2-4句话
- 语气：像一个有经验的同学在旁边说"我是这么想的"，不是教科书式叙述
- 目标：激发学生的思维模式，而非给出标准路径
- HTML使用时间轴样式（`.thought-flow` + `.thought-step`），关键洞察节点加 `.ts-insight` 类

#### 维度二：解题过程
- 至少提供2种解法（仅1种时说明原因）
- 每种解法完整步骤，公式用 LaTeX（`$...$` 行内，`$$...$$` 独立行）
- 标注解法特点：通法/巧解/计算量对比

#### 维度三：知识点拆解
- 格式：`大模块 > 中模块 > 具体知识点`
- 列出所有涉及知识点，包括隐含的
- 对应人教A版章节

**主模块分类（用于索引统计）：**
集合与逻辑、函数、三角函数、数列、不等式、向量、立体几何、解析几何、概率统计、导数、复数、计数原理

#### 维度四：思维方式
- 逐步标注每个关键节点所用思维
- 格式：`【步骤描述】→ 思维名称：为什么在这用`
- 常见：数形结合、分类讨论、化归转化、函数与方程、整体思想、构造法、反证法、特殊值法、逆向思维、设而不求、参数法、等价转换、递推思想、对称性、极端原理

#### 维度五：易错点与陷阱
- 2-4个常见错误
- 每个：❌ 错误做法 → ✅ 正确做法 → 原因分析

#### 维度六：变式题方向
- 2-3个变式
- 每个：改什么条件 → 变成什么题 → 核心区别

#### 维度七：难度定位
- 等级：基础(1) / 中档(2) / 中高档(3) / 压轴(4)
- 高考/模考定位、建议用时

### Step 3: 生成题目HTML文件

参照 **PROBLEM_TEMPLATE** 的结构和样式生成 `problem_NNN.html`，保存到 `~/Desktop/shuxue/`。

**要求：**
- 数学公式用 LaTeX，KaTeX 自动渲染
- 七个section用HTML结构化，不堆纯文本
- 每个section的div加 `id` 属性用于锚点跳转（思路/解法/知识点/思维/易错/变式/难度）
- 白色简洁界面，适合打印
- 如有图片，用 `<img src="images/problem_NNN.jpg">`

### Step 4: 更新索引

读取 `~/Desktop/shuxue/index.html`，在 `var problems = [...]` 数组末尾、`];` 之前追加：

```javascript
{
  id: NNN,
  date: 'YYYY-MM-DD',
  file: 'problem_NNN.html',
  title: '题目简短描述',
  difficulty: N,
  diffLabel: '中档',
  modules: ['解析几何'],
  moduleDetails: ['解析几何 > 椭圆 > 焦点弦长公式'],
  thinking: ['数形结合', '设而不求'],
  source: '作业'
}
```

### Step 5: 输出确认

简要告知：编号、文件路径、主要知识点、难度。

---

## PROBLEM_TEMPLATE

每道题目的HTML页面。白色主题、简洁、适合打印。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>题目 #NNN — 简短描述</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css">
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{font-family:-apple-system,'PingFang SC','Microsoft YaHei','Noto Sans SC',sans-serif;
  max-width:800px;margin:0 auto;padding:32px 24px;color:#1a1a1a;background:#fff;line-height:1.8;font-size:15px}

.nav{font-size:12px;color:#9ca3af;margin-bottom:20px}
.nav a{color:#2563eb;text-decoration:none}
.nav a:hover{text-decoration:underline}

.header{display:flex;align-items:center;justify-content:space-between;
  padding-bottom:14px;border-bottom:2px solid #1a1a1a;margin-bottom:24px}
.problem-id{font-size:24px;font-weight:800;color:#2563eb}
.header-right{display:flex;gap:12px;align-items:center;font-size:13px;color:#6b7280}
.difficulty-badge{padding:2px 12px;border-radius:12px;font-weight:600;font-size:12px}
.diff-1{background:#dcfce7;color:#166534}
.diff-2{background:#fef3c7;color:#92400e}
.diff-3{background:#fed7aa;color:#9a3412}
.diff-4{background:#fecaca;color:#991b1b}

.problem-image-section{margin-bottom:28px;text-align:center}
.problem-image{max-width:100%;max-height:500px;border:1px solid #e5e7eb;border-radius:4px}

.section{margin-bottom:36px}
.section-title{font-size:17px;font-weight:700;padding:8px 0 8px 14px;margin-bottom:16px;background:#f9fafb}
.st-1{border-left:4px solid #2563eb}
.st-2{border-left:4px solid #7c3aed}
.st-3{border-left:4px solid #0891b2}
.st-4{border-left:4px solid #dc2626}
.st-5{border-left:4px solid #ea580c}
.st-6{border-left:4px solid #16a34a}
.st-0{border-left:4px solid #d97706}
.thought-flow{position:relative}
.thought-step{position:relative;padding:10px 0 10px 28px;margin-bottom:4px;
  border-left:2px solid #e5e7eb;margin-left:6px}
.thought-step:last-child{border-left-color:transparent}
.thought-step::before{content:'';position:absolute;left:-7px;top:15px;
  width:12px;height:12px;border-radius:50%;border:2px solid #d97706;background:#fff}
.thought-step.ts-insight::before{background:#d97706}
.thought-label{font-size:13px;font-weight:700;margin-bottom:4px}
.thought-text{font-size:14px;color:#374151;line-height:1.8}

.method{margin-bottom:20px;padding:14px 16px;border:1px solid #e5e7eb;border-radius:6px}
.method-title{font-size:15px;font-weight:700;color:#2563eb;margin-bottom:10px}
.method-tag{display:inline-block;font-size:11px;padding:1px 8px;border-radius:4px;
  background:#eff6ff;color:#2563eb;margin-left:8px;font-weight:400}
.step{margin-bottom:6px;padding-left:4px}

.knowledge-item{padding:8px 12px;margin-bottom:6px;border-left:3px solid #e5e7eb;
  margin-left:8px;font-size:14px;background:#fafafa;border-radius:0 4px 4px 0}
.knowledge-item .module-tag{display:inline-block;font-size:11px;padding:1px 8px;
  border-radius:10px;background:#f3f4f6;color:#6b7280;margin-right:6px}
.knowledge-item .arrow{color:#9ca3af;margin:0 4px}

.thinking-item{display:flex;gap:12px;padding:10px 0;border-bottom:1px solid #f3f4f6;font-size:14px}
.thinking-step{flex:1;color:#374151}
.thinking-method{flex-shrink:0;padding:2px 10px;border-radius:10px;font-size:12px;
  font-weight:600;background:#f0f4ff;color:#2563eb;white-space:nowrap;align-self:flex-start}
.thinking-why{font-size:12px;color:#6b7280;margin-top:2px}

.pitfall{margin-bottom:14px;padding:12px 16px;border-radius:6px;border:1px solid #fecaca;background:#fef2f2}
.pitfall-wrong{color:#dc2626;font-size:14px;margin-bottom:4px}
.pitfall-wrong::before{content:'❌ '}
.pitfall-right{color:#16a34a;font-size:14px;margin-bottom:4px}
.pitfall-right::before{content:'✅ '}
.pitfall-why{font-size:13px;color:#6b7280;padding-left:20px}

.variation{margin-bottom:12px;padding:12px 16px;border:1px solid #e5e7eb;border-radius:6px;background:#f9fafb}
.variation-title{font-weight:600;font-size:14px;margin-bottom:4px;color:#ea580c}
.variation-desc{font-size:13px;color:#374151}

.difficulty-box{padding:16px;border:1px solid #e5e7eb;border-radius:6px;background:#f9fafb}
.diff-row{display:flex;gap:24px;margin-bottom:6px;font-size:14px}
.diff-label{color:#6b7280;min-width:80px}
.diff-value{font-weight:600}

.print-btn{position:fixed;top:16px;right:16px;width:36px;height:36px;border-radius:50%;
  border:1px solid #e5e7eb;background:#fff;font-size:16px;cursor:pointer;
  display:flex;align-items:center;justify-content:center;box-shadow:0 1px 3px rgba(0,0,0,.1)}
.print-btn:hover{background:#f3f4f6}

@media print{
  body{padding:16px;font-size:11pt}
  .section{page-break-inside:avoid}
  .print-btn,.nav{display:none}
  .problem-image{max-height:300px}
  .header{border-bottom-width:1px}
}
</style>
</head>
<body>
<button class="print-btn no-print" onclick="window.print()" title="打印">&#128424;</button>
<div class="nav"><a href="index.html">&larr; 返回索引</a></div>

<div class="header">
  <span class="problem-id"># NNN</span>
  <div class="header-right">
    <span>YYYY-MM-DD</span>
    <span>来源：XXX</span>
    <span class="difficulty-badge diff-N">难度标签</span>
  </div>
</div>

<div class="problem-image-section">
  <img src="images/problem_NNN.jpg" alt="题目" class="problem-image">
</div>

<div class="section" id="思路">
  <h2 class="section-title st-0">一、解题思路</h2>
  <div class="thought-flow">
    <div class="thought-step">
      <div class="thought-label" style="color:#2563eb">审题 — 第一反应</div>
      <div class="thought-text">提取关键条件，初步判断题型和方向</div>
    </div>
    <div class="thought-step">
      <div class="thought-label" style="color:#f59e0b">尝试 — 走过的弯路</div>
      <div class="thought-text">最初想用什么方法，遇到了什么困难</div>
    </div>
    <div class="thought-step ts-insight">
      <div class="thought-label" style="color:#10b981">突破 — 关键洞察</div>
      <div class="thought-text">怎么想到了正确的方法，核心转折点是什么</div>
    </div>
  </div>
</div>

<div class="section" id="解法">
  <h2 class="section-title st-1">二、解题过程</h2>
  <div class="method">
    <div class="method-title">解法一 <span class="method-tag">通法</span></div>
    <div class="step">步骤内容，公式用 $...$ 或 $$...$$ </div>
  </div>
</div>

<div class="section" id="知识点">
  <h2 class="section-title st-2">三、知识点拆解</h2>
  <div class="knowledge-item">
    <span class="module-tag">解析几何</span>
    解析几何 <span class="arrow">&gt;</span> 椭圆 <span class="arrow">&gt;</span> 焦点弦长公式
  </div>
</div>

<div class="section" id="思维">
  <h2 class="section-title st-3">四、思维方式</h2>
  <div class="thinking-item">
    <div>
      <div class="thinking-step">【设椭圆标准方程】</div>
      <div class="thinking-why">将问题纳入标准框架，便于使用公式</div>
    </div>
    <span class="thinking-method">化归转化</span>
  </div>
</div>

<div class="section" id="易错">
  <h2 class="section-title st-4">五、易错点与陷阱</h2>
  <div class="pitfall">
    <div class="pitfall-wrong">错误做法描述</div>
    <div class="pitfall-right">正确做法描述</div>
    <div class="pitfall-why">原因分析</div>
  </div>
</div>

<div class="section" id="变式">
  <h2 class="section-title st-5">六、变式题方向</h2>
  <div class="variation">
    <div class="variation-title">变式一：改变条件描述</div>
    <div class="variation-desc">变成什么新题，核心区别是什么</div>
  </div>
</div>

<div class="section" id="难度">
  <h2 class="section-title st-6">七、难度定位</h2>
  <div class="difficulty-box">
    <div class="diff-row"><span class="diff-label">难度等级</span><span class="diff-value">中档</span></div>
    <div class="diff-row"><span class="diff-label">考试定位</span><span class="diff-value">高考选填第X题水平</span></div>
    <div class="diff-row"><span class="diff-label">建议用时</span><span class="diff-value">5-8分钟</span></div>
    <div class="diff-row"><span class="diff-label">适合阶段</span><span class="diff-value">基础巩固后的提升训练</span></div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js"></script>
<script>
document.addEventListener("DOMContentLoaded",function(){
  renderMathInElement(document.body,{
    delimiters:[
      {left:'$$',right:'$$',display:true},
      {left:'$',right:'$',display:false},
      {left:'\\(',right:'\\)',display:false},
      {left:'\\[',right:'\\]',display:true}
    ],
    throwOnError:false
  });
});
</script>
</body>
</html>
```

---

## INDEX_TEMPLATE

汇总索引页面。深色主题，视觉丰富，用于学习诊断，通常不打印。

当 `~/Desktop/shuxue/index.html` 不存在时生成。`var problems = [];` 初始为空，每次分析题目后追加。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>高中数学 · 学习诊断</title>
<style>
:root{--bg:#0f172a;--card:#1e293b;--text:#e2e8f0;--text2:#94a3b8;
  --blue:#38bdf8;--gold:#fbbf24;--pink:#f472b6;--green:#34d399;
  --purple:#a78bfa;--orange:#fb923c;--fuchsia:#e879f9;--cyan:#22d3ee;
  --red:#f87171;--lime:#4ade80;--lpurple:#c084fc;--yellow:#fcd34d}
*{margin:0;padding:0;box-sizing:border-box}
html{scroll-behavior:smooth}
body{font-family:-apple-system,'PingFang SC','Microsoft YaHei',sans-serif;
  background:var(--bg);color:var(--text);line-height:1.7}

.header{padding:48px 40px 32px;text-align:center;
  background:linear-gradient(135deg,#0f172a,#1e3a5f,#0f172a);
  border-bottom:1px solid rgba(56,189,248,.1)}
.header h1{font-size:28px;font-weight:800}
.header h1 span{color:var(--blue)}
.header .sub{color:var(--text2);font-size:13px;margin-top:6px}

.tab-bar{display:flex;justify-content:center;gap:4px;margin:32px auto 0;
  background:var(--card);border-radius:12px;padding:4px;max-width:460px}
.tab{flex:1;padding:8px 16px;text-align:center;border-radius:10px;
  font-size:13px;font-weight:600;cursor:pointer;color:var(--text2);transition:all .2s}
.tab.on{background:var(--blue);color:#0f172a}
.tab:hover:not(.on){color:var(--text)}

.container{max-width:1000px;margin:0 auto;padding:32px 24px}
.view{display:none}.view.active{display:block}

.stats-bar{display:flex;gap:24px;justify-content:center;margin:24px 0 32px;flex-wrap:wrap}
.sb{text-align:center}
.sb .n{font-size:28px;font-weight:800;color:var(--blue)}
.sb .l{font-size:11px;color:var(--text2);margin-top:2px}

.problem-card{background:var(--card);border-radius:14px;margin-bottom:16px;overflow:hidden;
  border:1px solid rgba(255,255,255,.05);transition:border-color .2s}
.problem-card:hover{border-color:rgba(56,189,248,.2)}
.problem-head{padding:16px 20px;display:flex;align-items:center;justify-content:space-between;
  cursor:pointer;user-select:none}
.problem-head .left{display:flex;align-items:center;gap:14px}
.p-num{font-size:20px;font-weight:800;color:var(--blue);min-width:40px}
.p-title{font-size:15px;font-weight:700}
.p-date{font-size:12px;color:var(--text2);margin-top:2px}
.problem-head .right{display:flex;align-items:center;gap:8px}
.p-diff{font-size:11px;font-weight:600;padding:2px 10px;border-radius:8px}
.pd-1{background:rgba(34,197,94,.15);color:var(--lime)}
.pd-2{background:rgba(234,179,8,.15);color:var(--gold)}
.pd-3{background:rgba(249,115,22,.15);color:var(--orange)}
.pd-4{background:rgba(239,68,68,.15);color:var(--red)}
.p-modules{display:flex;gap:4px;flex-wrap:wrap}
.p-module-tag{font-size:10px;padding:1px 7px;border-radius:6px;
  background:rgba(255,255,255,.06);color:var(--text2)}
.open-btn{font-size:12px;color:var(--blue);text-decoration:none;padding:4px 12px;
  border:1px solid rgba(56,189,248,.3);border-radius:8px;transition:all .2s;white-space:nowrap}
.open-btn:hover{background:rgba(56,189,248,.1)}
.arrow{color:var(--text2);font-size:14px;transition:transform .2s;margin-left:8px}
.problem-card.expanded .arrow{transform:rotate(90deg)}
.p-detail{max-height:0;overflow:hidden;transition:max-height .3s ease;
  border-top:0 solid rgba(255,255,255,.04)}
.problem-card.expanded .p-detail{max-height:600px;border-top-width:1px}
.p-detail-inner{padding:12px 20px;font-size:13px;color:var(--text2)}
.p-detail-row{margin-bottom:6px}
.p-detail-label{color:var(--text2);margin-right:8px}
.p-detail-tags{display:flex;gap:4px;flex-wrap:wrap;margin-top:4px}
.p-thinking-tag{font-size:11px;padding:1px 8px;border-radius:8px;
  background:rgba(8,145,178,.12);color:var(--cyan)}

.module-group{margin-bottom:24px}
.module-group-head{display:flex;align-items:center;gap:10px;padding:12px 0;
  border-bottom:1px solid rgba(255,255,255,.06);margin-bottom:8px}
.module-group-head .m-bar{width:4px;height:22px;border-radius:2px;flex-shrink:0}
.module-group-head h3{font-size:15px;font-weight:700;flex:1}
.m-count{font-size:11px;font-weight:700;padding:2px 10px;border-radius:8px;
  background:rgba(255,255,255,.06);color:var(--text2)}
.m-problem{display:flex;align-items:center;gap:10px;padding:6px 0 6px 14px;font-size:13px}
.m-problem .m-link{color:var(--blue);text-decoration:none;font-size:12px}
.m-problem .m-link:hover{text-decoration:underline}
.m-problem .m-context{color:var(--text2);font-size:12px;flex:1}
.m-problem .m-diff{font-size:10px;padding:1px 6px;border-radius:4px}

.chart-row{display:flex;gap:24px;margin-bottom:24px;flex-wrap:wrap}
.chart-card{background:var(--card);border-radius:14px;padding:24px;flex:1;min-width:300px;
  border:1px solid rgba(255,255,255,.05)}
.chart-card.full{flex-basis:100%;min-width:100%}
.chart-title{font-size:14px;font-weight:700;margin-bottom:16px;display:flex;align-items:center;gap:8px}
.chart-subtitle{font-size:11px;color:var(--text2);margin-top:-10px;margin-bottom:16px}

.radar-wrap{display:flex;justify-content:center}
.radar-wrap svg{width:100%;max-width:420px;height:auto}

.cat-stat-row{display:flex;align-items:center;gap:10px;margin-bottom:10px}
.cat-stat-row .cs-dot{width:10px;height:10px;border-radius:50%;flex-shrink:0}
.cat-stat-row .cs-name{font-size:13px;width:64px}
.cat-stat-row .cs-bar-bg{flex:1;height:8px;border-radius:4px;background:rgba(255,255,255,.06);overflow:hidden}
.cat-stat-row .cs-bar{height:100%;border-radius:4px;transition:width .5s ease}
.cat-stat-row .cs-count{font-size:12px;color:var(--text2);min-width:28px;text-align:right}

.insight-row{display:flex;gap:16px;flex-wrap:wrap;margin-bottom:24px}
.ins-card{background:var(--card);border-radius:12px;padding:18px 20px;flex:1;min-width:140px;
  border:1px solid rgba(255,255,255,.05);text-align:center}
.ins-card .ins-val{font-size:24px;font-weight:800}
.ins-card .ins-label{font-size:11px;color:var(--text2);margin-top:4px}

.weakness-list{list-style:none;padding:0}
.weakness-item{display:flex;align-items:center;gap:10px;padding:8px 0;
  border-bottom:1px solid rgba(255,255,255,.04);font-size:13px}
.weakness-dot{width:8px;height:8px;border-radius:50%}
.weakness-name{flex:1}
.weakness-status{font-size:11px;padding:2px 8px;border-radius:6px}
.ws-none{background:rgba(239,68,68,.12);color:var(--red)}
.ws-weak{background:rgba(249,115,22,.12);color:var(--orange)}
.ws-ok{background:rgba(234,179,8,.12);color:var(--gold)}
.ws-good{background:rgba(34,197,94,.12);color:var(--lime)}

.thinking-cloud{display:flex;flex-wrap:wrap;gap:8px;justify-content:center;
  padding:16px;margin-bottom:16px}
.t-tag{padding:4px 14px;border-radius:16px;cursor:default;transition:all .2s;
  border:1px solid transparent;background:rgba(255,255,255,.04)}
.t-tag:hover{transform:scale(1.05);filter:brightness(1.2)}

.diff-bar-wrap{display:flex;align-items:flex-end;gap:16px;justify-content:center;
  height:120px;padding:0 20px}
.diff-col{display:flex;flex-direction:column;align-items:center;gap:4px}
.diff-bar{width:48px;border-radius:6px 6px 0 0;transition:height .5s ease;min-height:4px}
.diff-bar-label{font-size:11px;color:var(--text2)}
.diff-bar-count{font-size:14px;font-weight:700}

.heatmap-wrap{overflow-x:auto;padding-bottom:4px}
.heatmap-wrap svg{height:auto}
.hm-legend{display:flex;align-items:center;gap:6px;margin-top:12px;font-size:10px;
  color:var(--text2);justify-content:flex-end}
.hm-legend-cell{width:12px;height:12px;border-radius:2px}
.hm-tip{font-size:11px;color:var(--text2);margin-top:8px;text-align:center;font-style:italic}

.empty-state{text-align:center;color:var(--text2);padding:60px 20px;font-size:14px}
.footer{text-align:center;padding:32px;color:var(--text2);font-size:11px;
  border-top:1px solid rgba(255,255,255,.04);margin-top:40px}

@media(max-width:600px){
  .header{padding:36px 16px 24px}
  .header h1{font-size:22px}
  .container{padding:20px 12px}
  .problem-head{padding:14px 16px;flex-direction:column;align-items:flex-start;gap:8px}
  .problem-head .right{align-self:flex-end}
  .chart-row{flex-direction:column}
  .chart-card{min-width:auto}
  .insight-row{flex-direction:column}
  .ins-card{min-width:auto}
}
</style>
</head>
<body>

<div class="header">
  <h1>&#128209; 高中数学 · <span>学习诊断</span></h1>
  <div class="sub">题目分析汇总 &middot; 知识模块覆盖 &middot; 薄弱环节诊断</div>
</div>

<div class="tab-bar">
  <div class="tab on" onclick="switchView('list',this)">&#128197; 题目列表</div>
  <div class="tab" onclick="switchView('module',this)">&#128218; 知识模块</div>
  <div class="tab" onclick="switchView('diag',this)">&#128202; 学习诊断</div>
</div>

<div class="container">

<div class="stats-bar">
  <div class="sb"><div class="n" id="st-total">0</div><div class="l">题目总数</div></div>
  <div class="sb"><div class="n" id="st-modules">0</div><div class="l">涉及模块</div></div>
  <div class="sb"><div class="n" id="st-thinking">0</div><div class="l">思维方式</div></div>
  <div class="sb"><div class="n" id="st-weak">0</div><div class="l">待加强模块</div></div>
</div>

<div class="view active" id="view-list"></div>

<div class="view" id="view-module">
  <div id="module-groups"></div>
</div>

<div class="view" id="view-diag">
  <div class="insight-row" id="insight-row"></div>
  <div class="chart-row">
    <div class="chart-card">
      <div class="chart-title">&#128171; 知识模块雷达</div>
      <div class="chart-subtitle">各模块练习量 — 形状越饱满覆盖越均衡</div>
      <div class="radar-wrap" id="radar-chart"></div>
    </div>
    <div class="chart-card">
      <div class="chart-title">&#128202; 难度分布</div>
      <div class="chart-subtitle">各难度等级题目数量</div>
      <div class="diff-bar-wrap" id="diff-chart"></div>
      <div style="margin-top:24px">
        <div class="chart-title" style="margin-bottom:12px">&#128161; 薄弱模块诊断</div>
        <ul class="weakness-list" id="weakness-list"></ul>
      </div>
    </div>
  </div>
  <div class="chart-row">
    <div class="chart-card full">
      <div class="chart-title">&#129504; 思维方式频率</div>
      <div class="chart-subtitle">出现次数越多说明越常用 — 关注低频思维，它们可能是你的盲区</div>
      <div class="thinking-cloud" id="thinking-cloud"></div>
      <div id="thinking-bars"></div>
    </div>
  </div>
  <div class="chart-row">
    <div class="chart-card full">
      <div class="chart-title">&#128293; 学习热力图</div>
      <div class="chart-subtitle">每个格子代表一天 — 颜色越深当天练习越多</div>
      <div class="heatmap-wrap" id="heatmap-chart"></div>
      <div class="hm-legend">
        <span>少</span>
        <div class="hm-legend-cell" style="background:rgba(255,255,255,.04)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.2)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.45)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.7)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.95)"></div>
        <span>多</span>
      </div>
      <div class="hm-tip">坚持每天练习，让热力图亮起来</div>
    </div>
  </div>
</div>

</div>

<div class="footer">高中数学 · 学习诊断 &middot; 由 /math-solve 自动维护</div>

<script>
var problems = [];

var moduleNames=['集合与逻辑','函数','三角函数','数列','不等式','向量','立体几何','解析几何','概率统计','导数','复数','计数原理'];
var moduleColors={'集合与逻辑':'#38bdf8','函数':'#fbbf24','三角函数':'#f472b6','数列':'#34d399',
  '不等式':'#a78bfa','向量':'#fb923c','立体几何':'#e879f9','解析几何':'#22d3ee',
  '概率统计':'#f87171','导数':'#4ade80','复数':'#c084fc','计数原理':'#fcd34d'};
var diffLabels={1:'基础',2:'中档',3:'中高档',4:'压轴'};
var diffColors={1:'#22c55e',2:'#eab308',3:'#f97316',4:'#ef4444'};

var moduleCounts={},thinkingCounts={},diffCounts={1:0,2:0,3:0,4:0},allThinking=[];

function computeData(){
  moduleCounts={};thinkingCounts={};diffCounts={1:0,2:0,3:0,4:0};allThinking=[];
  moduleNames.forEach(function(m){moduleCounts[m]=0});
  problems.forEach(function(p){
    p.modules.forEach(function(m){moduleCounts[m]=(moduleCounts[m]||0)+1});
    p.thinking.forEach(function(t){
      thinkingCounts[t]=(thinkingCounts[t]||0)+1;
      if(allThinking.indexOf(t)===-1) allThinking.push(t);
    });
    diffCounts[p.difficulty]=(diffCounts[p.difficulty]||0)+1;
  });
}

function render(){
  computeData();
  var coveredModules=0,weakModules=0;
  moduleNames.forEach(function(m){
    if(moduleCounts[m]>0) coveredModules++;
    if(moduleCounts[m]<3) weakModules++;
  });
  document.getElementById('st-total').textContent=problems.length;
  document.getElementById('st-modules').textContent=coveredModules+'/'+moduleNames.length;
  document.getElementById('st-thinking').textContent=allThinking.length;
  document.getElementById('st-weak').textContent=weakModules;
  renderListView();
  renderModuleView();
  renderDiagView();
}

function renderListView(){
  var html='';
  problems.slice().reverse().forEach(function(p){
    html+='<div class="problem-card" onclick="toggleCard(this)">';
    html+='<div class="problem-head"><div class="left">';
    html+='<div class="p-num">#'+String(p.id).padStart(3,'0')+'</div>';
    html+='<div><div class="p-title">'+p.title+'</div>';
    html+='<div class="p-date">'+p.date+' &middot; '+p.source+'</div></div>';
    html+='</div><div class="right">';
    html+='<span class="p-diff pd-'+p.difficulty+'">'+p.diffLabel+'</span>';
    html+='<div class="p-modules">';
    p.modules.forEach(function(m){
      html+='<span class="p-module-tag" style="color:'+(moduleColors[m]||'#94a3b8')+'">'+m+'</span>';
    });
    html+='</div>';
    html+='<a class="open-btn" href="'+p.file+'" target="_blank" onclick="event.stopPropagation()">查看分析 &rarr;</a>';
    html+='<span class="arrow">&#9654;</span>';
    html+='</div></div>';
    html+='<div class="p-detail"><div class="p-detail-inner">';
    html+='<div class="p-detail-row" style="margin-bottom:10px"><span class="p-detail-label">跳转：</span>';
    ['思路','解法','知识点','思维','易错','变式','难度'].forEach(function(s){
      html+='<a href="'+p.file+'#'+s+'" target="_blank" style="font-size:11px;color:var(--blue);text-decoration:none;padding:2px 8px;border:1px solid rgba(56,189,248,.2);border-radius:6px;margin-right:4px;display:inline-block;margin-bottom:4px" onclick="event.stopPropagation()">'+s+'</a>';
    });
    html+='</div>';
    html+='<div class="p-detail-row"><span class="p-detail-label">知识点：</span>';
    (p.moduleDetails||[]).forEach(function(d){html+='<div style="margin-left:16px;font-size:12px">'+d+'</div>';});
    html+='</div>';
    html+='<div class="p-detail-row"><span class="p-detail-label">思维方式：</span>';
    html+='<div class="p-detail-tags">';
    p.thinking.forEach(function(t){html+='<span class="p-thinking-tag">'+t+'</span>';});
    html+='</div></div>';
    html+='</div></div></div>';
  });
  document.getElementById('view-list').innerHTML=html||'<div class="empty-state">暂无题目，发送数学题目开始分析</div>';
}

function renderModuleView(){
  var grouped={};
  moduleNames.forEach(function(m){grouped[m]=[]});
  problems.forEach(function(p){
    p.modules.forEach(function(m){
      if(!grouped[m]) grouped[m]=[];
      grouped[m].push(p);
    });
  });
  var sorted=moduleNames.slice().sort(function(a,b){return(grouped[b]||[]).length-(grouped[a]||[]).length});
  var html='';
  sorted.forEach(function(m){
    var ps=grouped[m]||[];
    var col=moduleColors[m]||'#94a3b8';
    html+='<div class="module-group">';
    html+='<div class="module-group-head">';
    html+='<div class="m-bar" style="background:'+col+'"></div>';
    html+='<h3 style="color:'+col+'">'+m+'</h3>';
    html+='<span class="m-count">'+ps.length+' 题</span>';
    html+='</div>';
    if(ps.length===0){
      html+='<div style="padding:8px 14px;font-size:12px;color:#64748b">暂无题目</div>';
    }
    ps.forEach(function(p){
      html+='<div class="m-problem">';
      html+='<a class="m-link" href="'+p.file+'#知识点" target="_blank">#'+String(p.id).padStart(3,'0')+' '+p.date+' &rarr;</a>';
      html+='<span class="m-context">'+p.title+'</span>';
      html+='<span class="m-diff p-diff pd-'+p.difficulty+'" style="font-size:10px;padding:1px 6px">'+p.diffLabel+'</span>';
      html+='</div>';
    });
    html+='</div>';
  });
  document.getElementById('module-groups').innerHTML=html;
}

function renderDiagView(){
  renderInsights();
  renderRadar();
  renderDiffChart();
  renderWeakness();
  renderThinkingCloud();
  renderHeatmap();
}

function renderInsights(){
  var topModule='',topVal=0;
  moduleNames.forEach(function(m){if(moduleCounts[m]>topVal){topVal=moduleCounts[m];topModule=m;}});
  var avgDiff=0;
  if(problems.length>0){
    var sum=0;problems.forEach(function(p){sum+=p.difficulty});
    avgDiff=(sum/problems.length).toFixed(1);
  }
  var dates=problems.map(function(p){return p.date}).sort();
  var span=0;
  if(dates.length>=2){
    var d0=new Date(dates[0]),d1=new Date(dates[dates.length-1]);
    span=Math.round((d1-d0)/86400000);
  }
  var html='';
  html+='<div class="ins-card"><div class="ins-val" style="color:'+(moduleColors[topModule]||'var(--blue)')+'">'+
    (topModule||'-')+'</div><div class="ins-label">练习最多模块</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--gold)">'+avgDiff+'</div><div class="ins-label">平均难度 (1-4)</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--green)">'+span+'</div><div class="ins-label">学习跨度 (天)</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--blue)">'+allThinking.length+'</div><div class="ins-label">思维方式种类</div></div>';
  document.getElementById('insight-row').innerHTML=html;
}

function renderRadar(){
  var names=moduleNames,n=names.length;
  var cx=190,cy=190,r=130;
  var step=2*Math.PI/n,start=-Math.PI/2;
  var maxVal=1;
  names.forEach(function(m){if(moduleCounts[m]>maxVal) maxVal=moduleCounts[m]});

  var svg='<svg viewBox="0 0 380 400" xmlns="http://www.w3.org/2000/svg">';
  [0.25,0.5,0.75,1].forEach(function(scale){
    var pts=[];
    for(var i=0;i<n;i++){
      var a=start+i*step;
      pts.push((cx+r*scale*Math.cos(a)).toFixed(1)+','+(cy+r*scale*Math.sin(a)).toFixed(1));
    }
    svg+='<polygon points="'+pts.join(' ')+'" fill="none" stroke="rgba(255,255,255,.07)"/>';
  });
  for(var i=0;i<n;i++){
    var a=start+i*step;
    svg+='<line x1="'+cx+'" y1="'+cy+'" x2="'+(cx+r*Math.cos(a)).toFixed(1)+'" y2="'+(cy+r*Math.sin(a)).toFixed(1)+'" stroke="rgba(255,255,255,.06)"/>';
  }
  var dataPts=[];
  for(var i=0;i<n;i++){
    var a=start+i*step;
    var val=Math.max(moduleCounts[names[i]]/maxVal,0.06);
    dataPts.push((cx+r*val*Math.cos(a)).toFixed(1)+','+(cy+r*val*Math.sin(a)).toFixed(1));
  }
  svg+='<polygon points="'+dataPts.join(' ')+'" fill="rgba(56,189,248,.12)" stroke="rgba(56,189,248,.6)" stroke-width="2"/>';
  for(var i=0;i<n;i++){
    var a=start+i*step;
    var val=Math.max(moduleCounts[names[i]]/maxVal,0.06);
    var dx=cx+r*val*Math.cos(a),dy=cy+r*val*Math.sin(a);
    var col=moduleColors[names[i]]||'#94a3b8';
    svg+='<circle cx="'+dx.toFixed(1)+'" cy="'+dy.toFixed(1)+'" r="4" fill="'+col+'" stroke="#0f172a" stroke-width="2"/>';
    var lx=cx+(r+22)*Math.cos(a),ly=cy+(r+22)*Math.sin(a);
    var anchor='middle';
    if(Math.cos(a)>0.3) anchor='start';
    else if(Math.cos(a)<-0.3) anchor='end';
    svg+='<text x="'+lx.toFixed(1)+'" y="'+ly.toFixed(1)+'" text-anchor="'+anchor+'" fill="'+col+'" font-size="11" font-weight="700">'+names[i]+'</text>';
    svg+='<text x="'+lx.toFixed(1)+'" y="'+(ly+14).toFixed(1)+'" text-anchor="'+anchor+'" fill="#94a3b8" font-size="10">'+moduleCounts[names[i]]+' 题</text>';
  }
  svg+='</svg>';
  document.getElementById('radar-chart').innerHTML=svg;
}

function renderDiffChart(){
  var maxVal=1;
  for(var d=1;d<=4;d++){if(diffCounts[d]>maxVal) maxVal=diffCounts[d];}
  var html='';
  for(var d=1;d<=4;d++){
    var h=Math.max(diffCounts[d]/maxVal*100,4);
    html+='<div class="diff-col">';
    html+='<div class="diff-bar-count" style="color:'+diffColors[d]+'">'+diffCounts[d]+'</div>';
    html+='<div class="diff-bar" style="height:'+h+'px;background:'+diffColors[d]+'"></div>';
    html+='<div class="diff-bar-label">'+diffLabels[d]+'</div>';
    html+='</div>';
  }
  document.getElementById('diff-chart').innerHTML=html;
}

function renderWeakness(){
  var sorted=moduleNames.slice().sort(function(a,b){return moduleCounts[a]-moduleCounts[b]});
  var html='';
  sorted.forEach(function(m){
    var c=moduleCounts[m]||0;
    var status,cls;
    if(c===0){status='未涉及';cls='ws-none';}
    else if(c<=2){status='需加强';cls='ws-weak';}
    else if(c<=5){status='初步掌握';cls='ws-ok';}
    else{status='较熟练';cls='ws-good';}
    var col=moduleColors[m]||'#94a3b8';
    html+='<li class="weakness-item">';
    html+='<div class="weakness-dot" style="background:'+col+'"></div>';
    html+='<span class="weakness-name">'+m+'</span>';
    html+='<span style="font-size:12px;color:var(--text2)">'+c+' 题</span>';
    html+='<span class="weakness-status '+cls+'">'+status+'</span>';
    html+='</li>';
  });
  document.getElementById('weakness-list').innerHTML=html;
}

function renderThinkingCloud(){
  var sorted=allThinking.slice().sort(function(a,b){return thinkingCounts[b]-thinkingCounts[a]});
  var maxFreq=1;
  sorted.forEach(function(t){if(thinkingCounts[t]>maxFreq) maxFreq=thinkingCounts[t]});

  var cloudHtml='';
  sorted.forEach(function(t){
    var freq=thinkingCounts[t];
    var size=Math.round(13+12*(freq/maxFreq));
    var opacity=0.5+0.5*(freq/maxFreq);
    cloudHtml+='<span class="t-tag" style="font-size:'+size+'px;color:var(--cyan);opacity:'+opacity.toFixed(2)+
      ';border-color:rgba(34,211,238,'+((freq/maxFreq)*0.3).toFixed(2)+')">'+t+' <sup style="font-size:10px">'+freq+'</sup></span>';
  });
  document.getElementById('thinking-cloud').innerHTML=cloudHtml||'<div class="empty-state" style="padding:20px">暂无数据</div>';

  var barHtml='';
  sorted.slice(0,10).forEach(function(t){
    var pct=Math.round(thinkingCounts[t]/maxFreq*100);
    barHtml+='<div class="cat-stat-row">';
    barHtml+='<div class="cs-dot" style="background:var(--cyan)"></div>';
    barHtml+='<div class="cs-name" style="width:80px">'+t+'</div>';
    barHtml+='<div class="cs-bar-bg"><div class="cs-bar" style="width:'+pct+'%;background:var(--cyan)"></div></div>';
    barHtml+='<div class="cs-count">'+thinkingCounts[t]+'</div>';
    barHtml+='</div>';
  });
  document.getElementById('thinking-bars').innerHTML=barHtml;
}

function renderHeatmap(){
  var today=new Date();today.setHours(0,0,0,0);
  var start52=new Date(today);start52.setDate(start52.getDate()-52*7);
  var earliest=start52;
  problems.forEach(function(p){
    var d=new Date(p.date+'T00:00:00');
    if(d<earliest) earliest=new Date(d);
  });
  var startDate=new Date(earliest);
  startDate.setDate(startDate.getDate()-startDate.getDay());

  var dateMap={},topicMap={};
  problems.forEach(function(p){
    dateMap[p.date]=(dateMap[p.date]||0)+1;
    topicMap[p.date]=(topicMap[p.date]||'')+p.title+' ';
  });

  var cells=[];
  var d=new Date(startDate);
  while(d<=today){
    var key=d.getFullYear()+'-'+pad2(d.getMonth()+1)+'-'+pad2(d.getDate());
    cells.push({date:key,count:dateMap[key]||0,topic:topicMap[key]||'',dow:d.getDay()});
    d.setDate(d.getDate()+1);
  }

  var cellSize=13,gap=3,cellStep=cellSize+gap;
  var weeks=Math.ceil(cells.length/7);
  var labelW=28,labelH=18;
  var W=labelW+weeks*cellStep+10;
  var H=labelH+7*cellStep+4;
  var svg='<svg viewBox="0 0 '+W+' '+H+'" width="'+W+'" xmlns="http://www.w3.org/2000/svg" style="min-width:'+W+'px">';

  var dayNames=['','一','','三','','五',''];
  for(var i=0;i<7;i++){
    if(dayNames[i]) svg+='<text x="'+(labelW-4)+'" y="'+(labelH+i*cellStep+cellSize-2)+'" text-anchor="end" fill="#64748b" font-size="9">'+dayNames[i]+'</text>';
  }

  var lastMonth=-1,cellIdx=0;
  var monthNames=['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月'];
  for(var w=0;w<weeks;w++){
    for(var dow=0;dow<7;dow++){
      if(cellIdx>=cells.length) break;
      var c=cells[cellIdx];
      var cDate=new Date(c.date+'T00:00:00');
      var mon=cDate.getMonth();
      if(mon!==lastMonth&&dow===0){
        svg+='<text x="'+(labelW+w*cellStep)+'" y="'+(labelH-5)+'" fill="#64748b" font-size="9">'+monthNames[mon]+'</text>';
        lastMonth=mon;
      }
      cellIdx++;
    }
  }
  cellIdx=0;
  for(var w=0;w<weeks;w++){
    for(var dow=0;dow<7;dow++){
      if(cellIdx>=cells.length) break;
      var c=cells[cellIdx];
      var x=labelW+w*cellStep,y=labelH+dow*cellStep;
      var fill=hmColor(c.count);
      var title=c.date+(c.count>0?' ('+c.count+'题) '+c.topic.trim():'');
      svg+='<rect x="'+x+'" y="'+y+'" width="'+cellSize+'" height="'+cellSize+'" rx="2" fill="'+fill+'"><title>'+title+'</title></rect>';
      cellIdx++;
    }
  }
  svg+='</svg>';
  document.getElementById('heatmap-chart').innerHTML=svg;
}

function hmColor(c){
  if(c===0) return 'rgba(255,255,255,.04)';
  if(c===1) return 'rgba(56,189,248,.25)';
  if(c<=3) return 'rgba(56,189,248,.5)';
  if(c<=5) return 'rgba(56,189,248,.75)';
  return 'rgba(56,189,248,.95)';
}
function pad2(n){return n<10?'0'+n:''+n}
function switchView(v,el){
  document.querySelectorAll('.tab').forEach(function(t){t.classList.remove('on')});
  el.classList.add('on');
  document.querySelectorAll('.view').forEach(function(vw){vw.classList.remove('active')});
  document.getElementById('view-'+v).classList.add('active');
}
function toggleCard(card){card.classList.toggle('expanded')}

render();
var first=document.querySelector('.problem-card');
if(first) first.classList.add('expanded');
</script>
</body>
</html>
```
