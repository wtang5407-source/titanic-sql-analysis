# titanic-sql-analysis
SQL + Pandas exploratory data analysis of Titanic dataset using SQLite and Python

# Titanic 数据集SQL+Pandas探索性数据分析

## 📌 项目简介

本项目使用**SQL**和**Pandas**对Titanic数据集进行探索性数据分析，旨在探究"什么样的人更可能在泰坦尼克号沉船事故中生还"。

项目分为两个阶段：
- **阶段一（SQL）**：使用SQL进行单变量分析、多变量交叉分析和生还者画像构建，验证"妇女儿童优先"的逃生规则
- **阶段二（Pandas）**：复现SQL分析结果，进行可视化探索，并完成SQL难以实现的高级分析（窗口函数对比、数据透视、特征工程）

通过两种工具的对比实践，总结了SQL与Pandas的适用场景和协同工作流。

**🚀 扩展应用**：本项目的方法论已迁移至医药临床试验分析场景，详见[从Titanic到医药临床试验的分析框架迁移指南](./medical_analysis_framework.md)。

## 📊 数据集
- **来源**：[Kaggle Titanic竞赛](https://www.kaggle.com/c/titanic)
- **记录数**：891名乘客
- **关键字段**：
  - `Survived`：是否生还（0=遇难，1=生还）
  - `Pclass`：舱位等级（1=一等舱，2=二等舱，3=三等舱）
  - `Sex`：性别
  - `Age`：年龄
  - `SibSp`：兄弟姐妹/配偶数量
  - `Parch`：父母/子女数量
  - `Fare`：票价
  - `Embarked`：登船港口（C=Cherbourg，Q=Queenstown，S=Southampton）

## 🔍 核心发现

| 发现维度 | 关键结论 | 数据支撑 |
|---------|---------|---------|
| **性别** | 女性生还率是男性的3.9倍，"妇女儿童优先"规则得到验证 | 女性74.2% vs 男性18.9% |
| **舱位** | 财富决定生存机会，1等舱生还率是3等舱的2.6倍 | 1等舱63% > 2等舱47% > 3等舱24% |
| **年龄** | 儿童受保护（59%），老年被放弃（16%），青年男性让出机会 | 儿童59% > 全体38% > 青年33% > 老年16% |
| **港口** | 港口差异本质是阶层分布，瑟堡富人多，皇后镇穷人多 | 瑟堡55%（50%为1等舱）> 皇后镇39%（93%为3等舱） |
| **性别×舱位** | 1等舱女性几乎全活，性别优先级高于经济阶层 | 1等舱女性96.8% > 3等舱女性50% > 1等舱男性37% |
| **年龄×舱位** | 儿童保护存在阶层天花板，3等舱儿童保护失效 | 1等舱儿童100% > 2等舱82% > 3等舱46% |
| **家庭规模** | 2-4人最优，单人孤立，大家庭混乱 | 4人家庭72% > 3人58% > 2人55% > 单人30% > 5人+25% |
| **最优画像** | "1等舱+女性+20-40岁+有家庭"最接近免死金牌 | 黄金群体96%生还率，vs全体38% |
| **最差画像** | "3等舱+男性+青年+单人"是最大受害群体 | 独行青年男17%生还率，占死亡55% |
| **规则悖论** | "妇女儿童优先"被阶层扭曲，富人中的穷人不如穷人中的富人 | 1等舱男性37% < 3等舱女性50% |

## 🐍 第二阶段：Pandas分析与可视化（新增）

在完成SQL分析后，使用Pandas进行深度分析和可视化探索，重点包括：

### 2.1 SQL结果复现与对比
- 使用Pandas复现全部SQL单变量分析
- 对比两种工具的结果一致性（验证通过）
- 分析代码可读性和执行效率差异

### 2.2 可视化探索（SQL无法完成）
生成6组可视化图表，直观呈现数据规律：
1. **生还总览**：饼图+柱状图展示38.4%生还率
2. **性别分析**：分组柱状图揭示74.2% vs 18.9%的悬殊差距
3. **舱位分析**：金银铜色梯度展示阶级差异
4. **年龄多维度**：直方图+箱线图+小提琴图+年龄段对比
5. **性别×舱位热力图**：96.8%深绿色块一眼定位最优群体
6. **家庭规模趋势**：倒U型曲线识别2-4人最优区间

### 2.3 窗口函数对比分析
| 分析场景 | SQL实现 | Pandas实现 | 关键发现 |
|---------|---------|-----------|---------|
| 全局占比 | `SUM(COUNT(*)) OVER()` | `len(df)` 标量除法 | Pandas更简洁 |
| 分组占比 | `OVER(PARTITION BY)` | `groupby().transform()` | 皇后镇55%为3等舱 |
| 排名分析 | `ROW_NUMBER()`（复杂） | `rank()`（1行） | 一等舱票价TOP10 |
| 环比分析 | `LAG()/LEAD()`（复杂） | `shift(1)/shift(-1)` | 舱内票价差异 |
| 移动平均 | `ROWS BETWEEN`（极复杂） | `rolling()`（1行） | 票价趋势平滑 |
| 数据透视 | `CASE WHEN`+`GROUP BY`+`UNION`（繁琐） | `pivot_table()`（1行） | 三维交叉分析 |

### 2.4 核心洞察（可视化发现）
- **热力图**：性别×舱位组合中，1等舱女性96.8%生还率形成"深绿色孤岛"
- **家庭规模曲线**：2-4人区间形成72%的"生存甜蜜点"，超出后断崖式下跌
- **年龄分布双峰**：遇难者在20岁和30岁有两个峰值（家庭顶梁柱牺牲）

📁 **相关文件**：
- [Jupyter Notebook: titanic_pandas_analysis.ipynb](./titanic_pandas_analysis.ipynb) - 完整Pandas分析代码
- [可视化图表集: titanic_visualizations/](./titanic_visualizations/) - 6组高清图表（PNG格式）
- [工具对比指南: sql_vs_pandas_comparison.md](./sql_vs_pandas_comparison.md) - SQL vs Pandas详细对比

## 🏥 扩展应用：医药临床试验分析框架（新增）

本项目建立的分析方法论已成功迁移至**医药临床试验**场景，形成完整的II期试验数据分析框架：

## 🛠️ 技术栈

### 第一阶段：SQL分析
- **数据库**：SQLite
- **查询语言**：SQL（聚合函数、窗口函数、子查询、CTE）
- **分析工具**：DB Browser for SQLite

### 第二阶段：Pandas分析（新增）
- **编程语言**：Python 3.x
- **核心库**：
  - **Pandas**：数据清洗、分组聚合、窗口函数、数据透视
  - **NumPy**：数值计算支持
  - **Matplotlib**：基础可视化（饼图、柱状图、折线图）
  - **Seaborn**：高级统计图表（热力图、小提琴图、箱线图）
- **开发环境**：Jupyter Notebook
- **可视化输出**：6组分析图表（300 DPI高清）

### 协作与版本控制
- **AI协同**：与AI副官完成需求拆解、代码生成、结果解读
- **版本控制**：Git + GitHub

## 📁 文件结构
titanic-sql-analysis/ 
├── README.md # 项目主页（本文件） │ 
├── SQL分析阶段/ │ 
├── titanic_queries.sql # 所有SQL分析脚本（按分析顺序排列，含注释） │ 
├── titanic_analysis_report.md # SQL完整分析报告（含详细结论和SQL代码） │
└── summary_findings.md # SQL核心发现摘要（3-5分钟快速阅读版） │ 
├── Pandas分析阶段/（新增） │
├── titanic_pandas_analysis.ipynb # Jupyter Notebook完整分析代码 │ 
├── titanic_visualizations/ # 可视化图表文件夹 │ │ ├── 
01_survival_overview.png # 生还总览（饼图+柱状图） │ │ ├── 
02_gender_survival.png # 性别分析（分组柱状图） │ │ ├── 
03_pclass_survival.png # 舱位分析（金银铜色梯度） │ │ ├── 
04_age_survival.png # 年龄多维度（4合1图表） │ │ ├── 
05_gender_pclass_heatmap.png # 性别×舱位热力图 │ │ └── 
06_family_size.png # 家庭规模趋势（倒U型曲线） │ └── 
sql_vs_pandas_comparison.md # 《SQL vs Pandas 工具对比指南》 │
├── 扩展应用/（新增） │ 
└── medical_analysis_framework.md # 《从Titanic到医药临床试验的分析框架迁移指南》 │
└── data/  └── train.csv # Titanic数据集（Kaggle来源）


## 🚀 快速开始

### 查看SQL分析
```bash
# 使用DB Browser for SQLite打开
# 执行 titanic_queries.sql 中的查询
```
### 查看Pandas分析
# 安装依赖
pip install pandas numpy matplotlib seaborn jupyter

# 启动Jupyter Notebook
jupyter notebook titanic_pandas_analysis.ipynb

查看可视化图表
直接打开  titanic_visualizations/  文件夹查看6组高清图表，或在内嵌显示。

## 📝 学习路径建议

本项目适合作为 SQL + Pandas协同分析 的入门实践：

1. **先执行SQL分析**：理解数据库查询思维和基础窗口函数
2. **再运行Pandas复现**：对比两种工具的语法差异和适用场景
3. **重点研究可视化**：这是SQL无法完成的环节，也是数据洞察的关键
4. **阅读对比指南**：建立工具选择直觉，形成协同工作流
5. **扩展至专业领域**：参考医药临床试验框架，迁移至你的业务场

## 🤝 协作记录

本项目采用**人类分析师 + AI副官**的协同模式完成：

| 阶段 | 人类任务 | AI任务 | 产出 |
|:---|:---|:---|:---|
| 需求拆解 | 提出分析问题 | 拆解为可执行步骤 | 分析路线图 |
| SQL分析 | 验证结果逻辑 | 生成SQL代码 | titanic_queries.sql |
| Pandas复现 | 运行代码反馈结果 | 生成Jupyter代码 | titanic_pandas_analysis.ipynb |
| 可视化 | 选择图表类型 | 生成Matplotlib/Seaborn代码 | 6组高清图表 |
| 工具对比 | 提供实践反馈 | 整理对比指南 | sql_vs_pandas_comparison.md |
| 框架迁移 | 提供业务场景 | 设计映射方案| medical_analysis_framework.md|


项目意义：通过Titanic这个经典数据集，完整实践了从"数据提取(SQL)"到"深度分析(Pandas)"再到"洞察可视化"的现代数据分析流程。更重要的是，建立了一套可迁移的分析方法论，成功应用于医药临床试验等真实业务场景，为数据分析师的职业能力构建提供了从"学习"到"应用"的完整路径。
