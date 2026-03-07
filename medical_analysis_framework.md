# 从Titanic到医药临床试验的分析框架迁移指南

> **适用场景**：II期临床试验数据分析  
> **核心目标**：识别药物响应驱动因素，定义高响应患者画像  
> **工具**：SQL + Pandas（复用Titanic项目方法论）

---

## 一、数据集映射对照

| Titanic字段 | 临床试验字段 | 业务含义 | 分析角色 |
|------------|-------------|---------|---------|
| Survived | Response | 0=无响应/死亡，1=有效响应/生还 | **目标变量(Y)** |
| Pclass | Treatment_Group | 1=用药组，2/3=对照组/不同剂量 | **核心分组(X1)** |
| Sex | Gender | male/female | 人口学特征(X2) |
| Age | Age | 连续变量 | 人口学特征(X3) |
| Fare | Dose | 支付能力→药物剂量 | 治疗强度(X4) |
| Embarked | Site | 登船港口→试验中心 | 中心效应(X5) |
| FamilySize | Adverse_Event | 家庭规模→安全性指标 | 风险指标(X6) |
| SibSp+Parch | Age+Gender | 组合特征 | 交互特征(X7) |

---

## 二、分析框架：三阶段九步骤

### 阶段一：数据基础（SQL主导）

#### 步骤1：数据质量评估

```sql
-- SQL：缺失值与数据完整性检查
SELECT 
    COUNT(*) AS 总患者数,
    COUNT(Response) AS 响应记录,
    COUNT(Dose) AS 剂量记录,
    COUNT(Adverse_Event) AS 不良反应记录,
    ROUND(AVG(Response)*100, 2) AS 总体响应率
FROM clinical_trial;

```python
# Pandas等效
import pandas as pd

df = pd.read_csv('clinical_trial.csv')
print(f"总患者数: {len(df)}")
print(f"总体响应率: {df['Response'].mean()*100:.2f}%")
print(df.isnull().sum())  # 缺失值统计

#### 步骤2：中心效应检查（类似港口分析）
```sql
-- SQL：各试验中心响应率（Site效应）
SELECT 
    Site,
    COUNT(*) AS 入组人数,
    SUM(Response) AS 响应人数,
    ROUND(AVG(Response)*100, 2) AS 响应率,
    ROUND(AVG(Adverse_Event)*100, 2) AS 不良率
FROM clinical_trial
GROUP BY Site
ORDER BY 响应率 DESC;

关键洞察：某些中心响应率异常高/低，可能存在：
 
患者选择偏倚（入选标准执行不一致）
 
数据质量问题
 
地域/人种差异（需进一步分析）

### 单变量分析（SQL+Pandas并行）
#### 步骤3：治疗组效应（核心分析，类似舱位分析）
```sql
-- SQL：用药组 vs 对照组
SELECT 
    Treatment_Group,
    COUNT(*) AS 人数,
    SUM(Response) AS 响应数,
    ROUND(AVG(Response)*100, 2) AS 响应率,
    ROUND(AVG(Dose), 2) AS 平均剂量,
    ROUND(AVG(Adverse_Event)*100, 2) AS 不良率
FROM clinical_trial
GROUP BY Treatment_Group;

```python
# Pandas：更详细的统计
treatment_analysis = df.groupby('Treatment_Group').agg({
    'Response': ['count', 'sum', 'mean'],
    'Dose': 'mean',
    'Adverse_Event': 'mean',
    'Age': 'mean'
}).round(4)

treatment_analysis.columns = ['人数', '响应数', '响应率', '平均剂量', '不良率', '平均年龄']
treatment_analysis['响应率(%)'] = (treatment_analysis['响应率'] * 100).round(2)

# 关键指标：相对风险(RR)和需治疗人数(NNT)
control_rate = df[df['Treatment_Group']=='对照组']['Response'].mean()
treatment_rate = df[df['Treatment_Group']=='用药组']['Response'].mean()
rr = treatment_rate / control_rate  # 相对风险
nnt = 1 / (treatment_rate - control_rate)  # 需治疗人数

print(f"相对风险(RR): {rr:.2f}")
print(f"需治疗人数(NNT): {nnt:.1f} (每治疗{nnt:.0f}人多1人响应)")

#### 步骤4：剂量-响应关系（类似票价分析，连续变量分箱）
```sql
-- SQL：剂量分箱（类似年龄段分组）
SELECT 
    CASE 
        WHEN Dose < 50 THEN '低剂量(<50mg)'
        WHEN Dose < 100 THEN '中剂量(50-100mg)'
        ELSE '高剂量(≥100mg)'
    END AS 剂量组,
    COUNT(*) AS 人数,
    ROUND(AVG(Response)*100, 2) AS 响应率,
    ROUND(AVG(Adverse_Event)*100, 2) AS 不良率
FROM clinical_trial
WHERE Treatment_Group = '用药组'  -- 仅分析用药组
GROUP BY 剂量组
ORDER BY MIN(Dose);

```python
# Pandas：更精细的剂量-响应曲线
import matplotlib.pyplot as plt

# 创建剂量分箱
df['Dose_Group'] = pd.cut(df['Dose'], 
                          bins=[0, 50, 100, 150, 200], 
                          labels=['低', '中', '高', '超高'])

# 剂量-响应关系
dose_response = df[df['Treatment_Group']=='用药组'].groupby('Dose_Group')['Response'].mean() * 100

# 可视化（类似Titanic舱位分析）
dose_response.plot(kind='bar', color=['#90EE90', '#FFD700', '#FF6B6B', '#8B0000'])
plt.title('剂量-响应关系（用药组）')
plt.ylabel('响应率(%)')
plt.axhline(y=control_rate*100, color='gray', linestyle='--', label='对照组基线')
plt.legend()

关键洞察：
 
是否存在剂量依赖性响应？
 
最佳剂量窗口在哪里？
 
高剂量是否带来更高不良率（安全性-有效性权衡）？

#### 步骤5：人口学特征（性别×年龄，复用Titanic分析）
```python
# Pandas：性别×年龄交叉分析（完全复用Titanic代码结构）

# 年龄分组（复用Titanic的age_group函数）
def age_group(age):
    if pd.isna(age): return '未知'
    elif age <= 18: return '青少年(≤18)'
    elif age <= 35: return '青年(19-35)'
    elif age <= 55: return '中年(36-55)'
    elif age <= 70: return '老年(56-70)'
    else: return '高龄(>70)'

df['AgeGroup'] = df['Age'].apply(age_group)

# 性别×年龄 响应率热力图（复用Titanic的pivot_table）
pivot_demo = pd.pivot_table(df[df['Treatment_Group']=='用药组'],
                            values='Response',
                            index='Gender',
                            columns='AgeGroup',
                            aggfunc='mean') * 100

print("用药组：性别×年龄 响应率(%)")
print(pivot_demo.round(1))

# 可视化（复用Titanic的热力图代码）
import seaborn as sns
sns.heatmap(pivot_demo, annot=True, fmt='.1f', cmap='RdYlGn', vmin=0, vmax=100)
plt.title('用药组：性别×年龄 响应率热力图')


### 阶段三：多变量分析与画像构建（Pandas主导）
####步骤6：不良反应影响分析（Titanic无直接对应，新增）
```python

# 安全性-有效性权衡分析
safety_efficacy = pd.crosstab([df['Treatment_Group'], df['Adverse_Event']], 
                               df['Response'], 
                               normalize='index') * 100

print("安全性-有效性矩阵（行百分比）：")
print(safety_efficacy.round(1))

# 关键问题：发生不良反应的患者是否响应更好？
ae_patients = df[df['Adverse_Event']==1]
no_ae_patients = df[df['Adverse_Event']==0]

print(f"\n发生不良反应患者响应率: {ae_patients['Response'].mean()*100:.1f}%")
print(f"未发生不良反应患者响应率: {no_ae_patients['Response'].mean()*100:.1f}%")

# 假设检验
from scipy import stats
chi2, p_value = stats.chi2_contingency(pd.crosstab(df['Adverse_Event'], df['Response']))[:2]
print(f"\n卡方检验p值: {p_value:.4f} ({'显著' if p_value < 0.05 else '不显著'})")

#### 步骤7：中心效应校正（类似港口×舱位交互）
```python
# 各中心的治疗效应是否一致？（中心×治疗组交互）
site_treatment = pd.pivot_table(df,
                                values='Response',
                                index='Site',
                                columns='Treatment_Group',
                                aggfunc='mean') * 100

print("各中心响应率(%)：")
print(site_treatment.round(1))

# 计算各中心的效应量（用药组-对照组）
site_treatment['效应量'] = site_treatment['用药组'] - site_treatment['对照组']
print("\n各中心治疗效应量（用药组-对照组）：")
print(site_treatment['效应量'].sort_values(ascending=False))

关键洞察：
 
是否存在"超级响应中心"？（可能入选了更适合的患者）
 
是否存在"无效应中心"？（可能执行偏倚或数据问题）
 
是否需要在III期试验中剔除某些中心？

#### 步骤8：高响应患者画像（复用Titanic"黄金群体"方法）
```python
# 定义多个候选画像（类似Titanic的10.1-10.5查询）

profiles = []

# 画像A：高剂量+女性+中年（假设）
profile_a = df[(df['Treatment_Group']=='用药组') & 
               (df['Dose'] >= 100) & 
               (df['Gender']=='Female') & 
               (df['Age'].between(36, 55))]
profiles.append({
    '画像名称': '高剂量女性中年',
    '条件': '用药组+剂量≥100mg+女性+36-55岁',
    '人数': len(profile_a),
    '响应率': profile_a['Response'].mean() * 100
})

# 画像B：低不良风险+男性+青年
profile_b = df[(df['Treatment_Group']=='用药组') & 
               (df['Adverse_Event']==0) & 
               (df['Gender']=='Male') & 
               (df['Age'].between(19, 35))]
profiles.append({
    '画像名称': '低不良风险男性青年',
    '条件': '用药组+无不良事件+男性+19-35岁',
    '人数': len(profile_b),
    '响应率': profile_b['Response'].mean() * 100
})

# 画像C：特定中心+高剂量（Site C效应）
profile_c = df[(df['Treatment_Group']=='用药组') & 
               (df['Site']=='C') & 
               (df['Dose'] >= 100)]
profiles.append({
    '画像名称': 'C中心高剂量',
    '条件': '用药组+Site C+剂量≥100mg',
    '人数': len(profile_c),
    '响应率': profile_c['Response'].mean() * 100
})

# 对比全体用药组
overall_treatment = df[df['Treatment_Group']=='用药组']['Response'].mean() * 100

profiles_df = pd.DataFrame(profiles)
profiles_df['提升(vs全体用药组)'] = profiles_df['响应率'] - overall_treatment

print(f"全体用药组响应率: {overall_treatment:.1f}%")
print("\n候选高响应画像：")
print(profiles_df.sort_values('响应率', ascending=False))

#### 步骤9：预测模型特征工程（超越Titanic，为III期准备）
```python

# 构建预测特征（类似Titanic的FamilySize特征工程）
df['Dose_per_Age'] = df['Dose'] / df['Age']  # 年龄标准化剂量
df['Risk_Score'] = df['Adverse_Event'] * 2 + (df['Age'] > 65).astype(int)  # 风险评分

# 特征重要性初筛（单变量逻辑回归）
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import LabelEncoder

# 准备特征
features = ['Age', 'Gender', 'Dose', 'Site', 'Dose_per_Age', 'Risk_Score']
X = df[df['Treatment_Group']=='用药组'][features].copy()
X['Gender'] = LabelEncoder().fit_transform(X['Gender'])
X['Site'] = LabelEncoder().fit_transform(X['Site'])
y = df[df['Treatment_Group']=='用药组']['Response']

# 逻辑回归
lr = LogisticRegression()
lr.fit(X, y)

# 特征重要性（系数）
feature_importance = pd.DataFrame({
    '特征': features,
    '系数': lr.coef_[0],
    'OR值': np.exp(lr.coef_[0])
}).sort_values('系数', ascending=False)

print("逻辑回归特征重要性（用药组内预测响应）：")
print(feature_importance)

## 三、关键输出物清单

| 输出物 | 工具 | 用途 | 对应Titanic产出 |
|-------|------|------|---------------|
| 数据质量报告 | SQL | 监管申报 | 缺失值统计 |
| 中心效应分析表 | SQL | 试验监查 | 港口分析 |
| 剂量-响应曲线 | Pandas+Matplotlib | 剂量选择 | 舱位分析图 |
| 人口学热力图 | Pandas+Seaborn | 亚组分析 | 性别×舱位热力图 |
| 安全性-有效性矩阵 | Pandas | 风险获益评估 | 新增 |
| 高响应画像排名 | Pandas | 适应症选择 | 黄金群体分析 |
| 特征重要性表 | Pandas+sklearn | III期设计 | 扩展分析 |

---

## 四、Titanic到临床试验的代码复用率

| Titanic分析模块 | 复用度 | 临床试验适配点 |
|--------------|-------|-------------|
| 数据加载与清洗 | 90% | 字段名映射 |
| 单变量分组统计 | 95% | Treatment_Group替代Pclass |
| 窗口函数（占比） | 100% | 直接复用 |
| 数据透视表 | 100% | Site替代Embarked |
| 可视化（柱状图/热力图） | 85% | 颜色主题调整（医学蓝） |
| 特征工程 | 70% | 新增Dose相关特征 |
| 安全性分析 | 0% | 新增模块（Titanic无对应） |
| 预测模型 | 0% | 新增模块（II期探索性） |

**总体复用率：约75%**

---

## 五、临床试验特有的分析要点

### 5.1 监管合规性

- 所有分析需**预定义**在SAP（统计分析计划）中
- 缺失值处理需符合**ITT（意向性治疗）**原则
- 中心效应需进行**敏感性分析**

### 5.2 医学意义优先

- 统计显著 ≠ 临床意义（响应率提升需≥20%才有价值）
- 关注**NNT**（需治疗人数）而非仅p值
- 安全性信号（不良率>10%需警惕）

### 5.3 后续试验设计

II期结果直接决定III期的：

- 目标适应症（高响应画像）
- 对照组选择（标准治疗 vs 安慰剂）
- 样本量计算（效应量估计）
- 主要终点定义（响应率 vs 生存期）

---

## 六、快速启动代码模板

```python
# 临床试验分析启动模板（复用Titanic项目结构）

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats

# 设置（复用Titanic配置）
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# 加载数据
df = pd.read_csv('clinical_trial.csv')

# 快速概览（复用Titanic结构）
print(f"总患者数: {len(df)}")
print(f"总体响应率: {df['Response'].mean()*100:.1f}%")
print(f"用药组响应率: {df[df['Treatment_Group']=='用药组']['Response'].mean()*100:.1f}%")
print(f"对照组响应率: {df[df['Treatment_Group']=='对照组']['Response'].mean()*100:.1f}%")

# 复用Titanic的全部分析函数（只需改字段名）
# 1. 单变量分析 → 改Pclass为Treatment_Group
# 2. 交叉分析 → 改Sex为Gender，Embarked为Site
# 3. 特征工程 → 新增Dose相关特征
# 4. 可视化 → 颜色改为医学蓝绿配色

# 医学配色方案（替代Titanic的红绿）
medical_palette = ['#2E86AB', '#A23B72', '#F18F01', '#C73E1D']  # 蓝紫橙红

结语
Titanic项目建立的分析思维（分组比较、交互效应、画像构建）可直接迁移至临床试验，只需关注医学特异性的安全-有效性权衡和监管合规要求。75%的代码可直接复用，剩余25%是临床试验增值部分。





