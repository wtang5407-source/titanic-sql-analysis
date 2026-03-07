# SQL vs Pandas 工具对比指南

> **版本**：v1.0  
> **基于实践**：Titanic数据分析项目（891行数据）  
> **制作者**：AI副官 × 用户协同学习产出  
> **日期**：2026年3月

---

## 一、适用场景对比

### 1.1 选择SQL的场景

| 场景特征 | 原因说明 | 药企市场部的实际案例 |
|---------|---------|-------------------|
| 数据量 > 100万行 | 数据库索引优化，内存占用低 | 全国医院处方数据查询 |
| 生产环境自动化 | 定时任务、ETL管道标准化 | 每月自动导出销售报表 |
| 多用户协作 | 权限管理、版本控制成熟 | 团队共享市场洞察数据库 |
| 数据在远程服务器 | 网络传输原始数据成本高 | 查询总部中央数据仓库 |
| 简单聚合报表 | 单次查询即可完成的KPI | Q季度各区域销售额汇总 |
| 数据清洗标准化 | 确保所有人使用同一逻辑 | 统一客户分级标准 |

### 1.2 选择Pandas的场景

| 场景特征 | 原因说明 | 药企市场部的实际案例 |
|---------|---------|-------------------|
| 探索性分析(EDA) | 交互式、即时反馈、可视化 | 新上市药品的处方趋势探索 |
| 复杂特征工程 | 窗口函数、移动统计、排名 | 计算医生的"活跃度评分" |
| 数据透视与交叉 | pivot_table一键多维度 | 产品×区域×时间的销售矩阵 |
| 机器学习预处理 | 与sklearn无缝衔接 | 构建医生潜力预测模型 |
| 数据量 < 10万行 | 内存充足时速度更快 | 单产品的医院调研数据 |
| 快速原型验证 | 试错成本低，迭代快 | 验证一个新的市场细分假设 |

---

## 二、操作对比表（12组核心操作）

| 序号 | 分析任务 | SQL实现 | Pandas实现 | 推荐工具 | Titanic案例 |
|:---:|---------|---------|-----------|:-------:|-------------|
| 1 | 基础过滤 | `SELECT * FROM titanic WHERE Age > 18` | `df[df['Age'] > 18]` | 平手 | 成年人筛选 |
| 2 | 分组聚合 | `SELECT Sex, AVG(Survived) FROM titanic GROUP BY Sex` | `df.groupby('Sex')['Survived'].mean()` | 平手 | 性别生还率 |
| 3 | 全局占比 | `COUNT(*) * 100.0 / SUM(COUNT(*)) OVER()` | `df['Survived'].value_counts() / len(df)` | Pandas | 生还分布38.4% |
| 4 | 分组占比 | `COUNT(*) / SUM(COUNT(*)) OVER(PARTITION BY Embarked)` | `.groupby().transform('sum')` | Pandas | 港口舱位结构 |
| 5 | 条件分组 | `CASE WHEN Age <= 12 THEN '儿童'... END` | `.apply(lambda x: ...)` 或 `pd.cut()` | SQL | 年龄段划分 |
| 6 | 排名 | `ROW_NUMBER() OVER(ORDER BY Fare DESC)` | `df['Fare'].rank(method='min')` | Pandas | 票价TOP10 |
| 7 | 滞后/领先 | `LAG(Fare, 1) OVER(PARTITION BY Pclass)` | `.groupby('Pclass')['Fare'].shift(1)` | Pandas | 舱内票价环比 |
| 8 | 移动平均 | `AVG(Fare) OVER(ROWS 2 PRECEDING)` | `.rolling(window=3).mean()` | Pandas | 票价趋势 |
| 9 | 数据透视 | 复杂`CASE WHEN`+`GROUP BY`+`UNION` | `pd.pivot_table()` | Pandas | 性别×舱位热力图 |
| 10 | 缺失值处理 | `COALESCE(Age, AVG(Age) OVER())` | `df['Age'].fillna(df['Age'].mean())` | Pandas | 年龄填充 |
| 11 | 多表连接 | `SELECT * FROM a JOIN b ON a.id = b.id` | `pd.merge(a, b, on='id')` | SQL | 多数据源整合 |
| 12 | 字符串处理 | `SUBSTRING(Name, 1, INSTR(Name, ','))` | `df['Name'].str.split(',').str[0]` | Pandas | 姓名提取头衔 |

---

## 三、药企市场部分析岗位学习建议

### 3.1 技能需求分析

| 工作模块 | 核心任务 | 主要工具 | 优先级 |
|---------|---------|---------|:------:|
| 销售数据分析 | 月度报表、区域对比、趋势追踪 | SQL + Excel | 5星 |
| 医生画像分析 | 处方行为、潜力评分、细分聚类 | Pandas + Python | 5星 |
| 竞品监测 | 市场份额、竞争格局、价格分析 | SQL + Pandas | 4星 |
| 预测模型 | 销量预测、医生转化概率 | Pandas + sklearn | 3星 |
| 可视化报告 | 管理层汇报、区域经理看板 | Pandas + 可视化库 | 4星 |

### 3.2 学习路径建议（3个月计划）

**第1个月：SQL基础（40%精力）**

- Week 1: 基础查询（SELECT, WHERE, ORDER BY, 聚合函数）
- Week 2: 多表操作（JOIN家族，子查询与CTE）
- Week 3: 窗口函数入门（OVER(), OVER(PARTITION BY), ROW_NUMBER）
- Week 4: 实战项目（Titanic基础分析，药企销售数据库查询）

**第2个月：Pandas核心（50%精力）**

- Week 1: 数据结构（Series vs DataFrame，索引操作）
- Week 2: 数据清洗（缺失值、重复值、异常值处理）
- Week 3: 数据分析（groupby深入，透视表，合并操作）
- Week 4: 窗口函数与特征工程（rolling, shift, 特征构造）

**第3个月：整合实战（10%精力SQL，90%精力Pandas）**

- Week 1: 协同工作流（SQL提取 → Pandas分析 → 可视化）
- Week 2: 可视化（Matplotlib, Seaborn, Plotly）
- Week 3: 综合项目（复现Titanic全流程，药企场景模拟）
- Week 4: 面试准备（SQL笔试题，Pandas操作题）

### 3.3 精力分配比例

**总体建议：SQL 30% + Pandas 70%**

原因：
1. SQL入门快，掌握基础查询和窗口函数即可应对80%场景
2. Pandas学习曲线陡峭但收益高，是"分析师"与"取数员"的分水岭
3. 药企更重视"洞察产出"而非"取数能力"
4. Pandas掌握的深度直接决定你的分析天花板

---

## 四、个人实践心得（Titanic项目）

### 4.1 核心观察

**观察1：SQL的"隐形天花板"**

> 我可以用SQL算出各舱位的生还率，但当我想要"每个舱位内票价排名前3的乘客"时，SQL突然变得复杂。而在Pandas里，这只是`df.groupby('Pclass').apply(lambda x: x.nlargest(3, 'Fare'))`。

**体会**：SQL擅长"回答已知问题"，Pandas擅长"发现未知问题"。

**观察2：窗口函数的"语法负担"**

| SQL窗口函数 | 记忆难度 | Pandas等效 | 记忆难度 |
|-----------|---------|-----------|---------|
| `SUM() OVER(PARTITION BY x ORDER BY y ROWS UNBOUNDED PRECEDING)` | 5星 | `.groupby(x)[y].cumsum()` | 2星 |
| `LAG(col, 1) OVER(PARTITION BY x ORDER BY y)` | 4星 | `.groupby(x)[col].shift(1)` | 2星 |

**体会**：Pandas的方法命名更符合直觉，`shift`就是"移位"，`cumsum`就是"累积和"。

**观察3：数据透视表的"降维打击"**

> 用SQL做"性别×舱位×年龄段"的三维交叉分析，我需要写嵌套子查询或多次JOIN。Pandas的`pivot_table`让我一行代码就看到了96.8%这个数字——一等舱女性的生还率。

**体会**：多维度交叉分析是Pandas的"杀手级"功能。

**观察4：可视化的"认知闭环"**

> SQL给我数字：一等舱生还率63%，三等舱24%。但当我用Pandas画出那张金色-银色-铜色的柱状图时，"阶级决定生存"这个结论才真正击中我。

**体会**：Pandas + Matplotlib完成了从"数据"到"洞察"再到"说服"的完整链条。

### 4.2 个人工具选择策略

基于实践，形成以下工作流：

1. 数据在数据库 → SQL提取
2. 数据在本地 → Pandas读取
3. 深度分析 → Pandas（清洗、特征、透视、可视化）
4. 洞察输出 → 统计结论 + 可视化图表 + 业务建议

### 4.3 给同期学习者的建议

1. **不要纠结"学哪个"**：两者都要学，但分主次。建议先掌握SQL基础，再深入Pandas。
2. **以项目驱动学习**：Titanic项目让我同时用两种工具实现相同功能，这种对比是最有效的学习方式。
3. **关注"思维转换"**：SQL是声明式（告诉电脑要什么），Pandas是命令式（告诉电脑怎么做）。
4. **建立"工具直觉"**：
   - 需要连接生产数据库？→ SQL
   - 需要做特征工程？→ Pandas
   - 需要出图？→ Pandas
   - 数据量>100万？→ SQL
5. **药企场景的特殊性**：医药行业数据敏感，很多时候你拿到的已经是脱敏后的CSV，这时候Pandas的能力直接决定你的分析质量。

---

## 五、速查表：核心差异

| SQL | Pandas |
|-----|--------|
| 数据库里查数据 | 内存里分析数据 |
| 标准化、稳定 | 灵活、快速迭代 |
| 适合"生产环境" | 适合"探索发现" |
| 大数据的"搬运工" | 小数据的"雕刻家" |

**核心语法对照**：

| SQL | Pandas |
|-----|--------|
| `SELECT *` | `df[['col1', 'col2']]` |
| `WHERE` | `df[df['age'] > 18]` |
| `GROUP BY` | `df.groupby('sex')` |
| `SUM/COUNT/AVG` | `.sum()/.count()/.mean()` |
| `OVER()` | 标量值 / transform |
| `OVER(PARTITION BY)` | `groupby().transform()` |
| `ROW_NUMBER()` | `.rank(method='min')` |
| `LAG/LEAD` | `.shift(1)` / `.shift(-1)` |
| `CASE WHEN` | `.apply()` / `np.where()` |
| `JOIN` | `pd.merge()` |
| （无） | `pd.pivot_table()` |
| （无） | `.rolling()` / `.expanding()` |

---

## 附录：推荐学习资源

| 类型 | 资源 | 用途 |
|-----|------|-----|
| SQL练习 | LeetCode Database | 面试笔试准备 |
| SQL练习 | SQLBolt | 交互式入门 |
| Pandas练习 | Kaggle Learn Pandas | 系统化学习 |
| Pandas速查 | Pandas官方Cheat Sheet | 日常查阅 |
| 综合项目 | Titanic数据集 | 工具对比实践 |

---

> **结语**：SQL和Pandas不是竞争关系，而是数据分析师的左右手。SQL让你拿到数据，Pandas让你理解数据。在药企市场部的岗位上，两者结合才能从"取数"走向"洞察"，从"报表"走向"策略"。
