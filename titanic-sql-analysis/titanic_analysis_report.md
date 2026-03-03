Titanic数据SQL探索分析报告
分析工具：SQLite
数据来源：Kaggle Titanic数据集（train.csv，891条记录）
分析时间：2025年
核心方法：SQL聚合、窗口函数、多表交叉分析

一、项目背景
1.1 数据集介绍
Titanic数据集记录了1912年泰坦尼克号沉船事故中891名乘客的个人信息及生还情况，是数据科学领域最经典的入门数据集之一。
字段	含义	类型	
PassengerId	乘客编号	INTEGER	
Survived	是否生还（0=否，1=是）	INTEGER	
Pclass	舱位等级（1/2/3）	INTEGER	
Name	姓名	TEXT	
Sex	性别（male/female）	TEXT	
Age	年龄	REAL	
SibSp	兄弟姐妹/配偶数量	INTEGER	
Parch	父母/子女数量	INTEGER	
Ticket	船票号	TEXT	
Fare	票价	REAL	
Cabin	舱房号	TEXT	
Embarked	登船港口（C/Q/S）	TEXT	

1.2 分析目标
 
描述性分析：哪些因素单独影响生还率？
 
诊断性分析：因素之间如何交互作用？
 
预测性洞察：能否定义"高生还可能"的乘客画像？

二、数据概况
2.1 数据量与完整性
sql
-- 数据总量与缺失值统计
SELECT 
    COUNT(*) AS 总人数,
    COUNT(Age) AS 有年龄记录,
    COUNT(Cabin) AS 有舱房记录,
    COUNT(Embarked) AS 有港口记录
FROM passengers;

结果：
指标	数值	说明	
总乘客数	891	完整样本	
年龄缺失	177（19.9%）	需处理NULL	
舱房缺失	687（77.1%）	缺失率过高，分析价值低	
港口缺失	2（0.2%）	可忽略	

2.2 基础分布
维度	分布	关键发现	
生还情况	遇难549（61.6%）/ 生还342（38.4%）	死亡率接近2/3	
性别	男性577（64.8%）/ 女性314（35.2%）	男性占多数	
舱位	1等216 / 2等184 / 3等491（55.1%）	3等舱人数过半	
登船港口	瑟堡168 / 皇后镇77 / 南安普顿644（72.3%）	英国主港为主	

三、单变量分析
3.1 性别：最显著的生还因子
sql
SELECT 
    CASE WHEN Sex = 'male' THEN '男性' ELSE '女性' END AS 性别,
    COUNT(*) AS 人数,
    SUM(Survived) AS 生还人数,
    ROUND(AVG(Survived) * 100, 2) AS 生还率
FROM passengers
GROUP BY Sex;

结果：
性别	人数	生还人数	生还率	
女性	314	233	74.20%	
男性	577	109	18.89%	

结论：女性生还率是男性的3.9倍，"妇女儿童优先"的逃生规则得到数据验证。

3.2 舱位等级：财富决定生存机会
sql
SELECT 
    Pclass AS 舱位,
    COUNT(*) AS 人数,
    ROUND(AVG(Survived) * 100, 2) AS 生还率
FROM passengers
GROUP BY Pclass
ORDER BY Pclass;

结果：
舱位	人数	生还率	
1等舱	216	62.96%	
2等舱	184	47.28%	
3等舱	491	24.24%	

结论：1等舱生还率是3等舱的2.6倍。舱位不仅代表舒适，更意味着逃生通道位置、信息获取优先级和救生艇分配权。

3.3 年龄：儿童受保护，老年被放弃
sql
SELECT 
    CASE 
        WHEN Age IS NULL THEN '未知'
        WHEN Age <= 12 THEN '儿童(0-12)'
        WHEN Age <= 18 THEN '青少年(13-18)'
        WHEN Age <= 35 THEN '青年(19-35)'
        WHEN Age <= 55 THEN '中年(36-55)'
        ELSE '老年(55+)'
    END AS 年龄段,
    COUNT(*) AS 人数,
    ROUND(AVG(Survived) * 100, 2) AS 生还率
FROM passengers
GROUP BY 年龄段
ORDER BY MIN(Age);

结果：
年龄段	人数	生还率	
儿童(0-12)	68	58.82%	
青少年(13-18)	70	45.71%	
青年(19-35)	382	33.25%	
中年(36-55)	162	35.80%	
老年(55+)	32	15.63%	
未知	177	45.20%	

结论：儿童生还率最高（59%），体现保护规则；老年最低（16%），体力劣势明显；青年男性（19-35岁）人数最多但生还率仅33%，可能是"让出机会"的主力。

3.4 登船港口：阶层分布的地理标签
sql
SELECT 
    CASE 
        WHEN Embarked = 'C' THEN '瑟堡(C)'
        WHEN Embarked = 'Q' THEN '皇后镇(Q)'
        ELSE '南安普顿(S)'
    END AS 港口,
    COUNT(*) AS 人数,
    ROUND(AVG(Survived) * 100, 2) AS 生还率
FROM passengers
WHERE Embarked IS NOT NULL
GROUP BY Embarked;

结果：
港口	人数	生还率	深层原因	
瑟堡(C)	168	55.36%	法国港口，1等舱乘客比例50%	
皇后镇(Q)	77	38.96%	爱尔兰移民港，93%为3等舱	
南安普顿(S)	644	33.70%	英国主港，各阶层混杂	

结论：港口生还率差异本质是阶层分布差异——瑟堡吸引富人，皇后镇聚集穷移民。

四、多变量分析
4.1 性别 × 舱位：双重优势的叠加效应
sql
SELECT 
    Pclass AS 舱位,
    CASE WHEN Sex = 'male' THEN '男性' ELSE '女性' END AS 性别,
    COUNT(*) AS 人数,
    ROUND(AVG(Survived) * 100, 2) AS 生还率
FROM passengers
GROUP BY Pclass, Sex
ORDER BY Pclass, 性别 DESC;

结果：
舱位	性别	人数	生还率	
1等舱	女性	94	96.81%	
1等舱	男性	122	36.89%	
2等舱	女性	76	92.11%	
2等舱	男性	108	15.74%	
3等舱	女性	144	50.00%	
3等舱	男性	347	13.54%	

关键发现：
 
1等舱女性几乎全活（96.8%），是最大赢家
 
1等舱男性（37%）< 3等舱女性（50%）：性别优先级高于经济阶层
 
3等舱男性生还率仅13.5%，人数却最多（347人），是最大受害群体

4.2 年龄 × 舱位：儿童保护的阶层分化
sql
SELECT 
    Pclass AS 舱位,
    CASE WHEN Age <= 12 THEN '儿童' ELSE '成年人' END AS 人群,
    COUNT(*) AS 人数,
    ROUND(AVG(Survived) * 100, 2) AS 生还率
FROM passengers
WHERE Age IS NOT NULL
GROUP BY Pclass, 人群;

结果：
舱位	人群	生还率	与成人差距	
1等舱	儿童	100%	+38%	
1等舱	成年人	62%	—	
2等舱	儿童	82%	+38%	
2等舱	成年人	44%	—	
3等舱	儿童	46%	+13%	
3等舱	成年人	33%	—	

关键发现：儿童保护存在阶层天花板——1-2等舱儿童生还率80-100%，但3等舱儿童骤降至46%，"优先规则"在底层失效。

4.3 家庭规模：适度最优，两端危险
sql
SELECT 
    SibSp + Parch + 1 AS FamilySize,
    COUNT(*) AS 人数,
    ROUND(AVG(Survived) * 100, 2) AS 生还率
FROM passengers
GROUP BY FamilySize
ORDER BY FamilySize;

结果：
家庭规模	人数	生还率	解读	
1（单人）	537	30.35%	孤立无援，最大群体	
2	161	55.28%	夫妻互助	
3	102	57.84%	小家庭最优	
4	29	72.41%	中等家庭最佳	
5+	62	<25%	疏散混乱，组织困难	

关键发现：生还率呈倒U型曲线——单人太孤独（30%），5人+太混乱（<25%），2-4人刚刚好（55-72%）。

五、生还者画像验证
画像定义与结果
画像	定义条件	人数	生还率	vs全体	
黄金群体	1等舱+女性+20-40岁+有家庭	45	96%	+58%	
精英儿童	1等舱+儿童+有家庭	5	100%	+62%	
中产女性	2等舱+女性+家庭2-4人	35	91%	+53%	
独行青年男	3等舱+男性+19-35岁+单人	150	17%	-21%	

sql
-- 黄金群体SQL
SELECT 
    COUNT(*) AS 人数,
    ROUND(AVG(Survived) * 100, 2) AS 生还率
FROM passengers
WHERE Sex = 'female'
  AND Pclass = 1
  AND Age BETWEEN 20 AND 40
  AND (SibSp + Parch + 1) > 1;

六、最终结论
6.1 数据支撑的完整故事
"1912年4月15日凌晨，泰坦尼克号的891名乘客面临生死抉择。数据揭示了一个残酷的生存法则：你的舱位等级、性别、年龄和家庭结构，共同编织了一张'生还概率网'。
最幸运的人是'1等舱中青年女性且有家庭陪同'——她们占据性别红利（'妇女儿童优先'）、财富特权（甲板位置、信息优先）和互助网络三重优势，生还率高达96%。而最不幸的人是'3等舱独行青年男性'——他们被排除在保护规则之外，困于底层舱房的物理隔离，生还率仅13%，却占全船死亡人数的55%。
'妇女儿童优先'的道德规则确实存在，但它被阶层结构严重扭曲：3等舱儿童的生还率（46%）仍低于1等舱成年男性（37%），'优先'变成了'富裕女性优先于贫穷儿童'。灾难从不随机，它放大社会的不平等。"

6.2 核心洞察矩阵
因素	独立效应	交互效应	
性别	女性3.9倍优势	与舱位叠加产生96%生还率	
舱位	1等舱2.6倍优势	决定儿童保护是否有效	
年龄	儿童2倍优势	在3等舱被压缩至1.4倍	
家庭	2-4人最优	单人孤立，大家庭混乱	

6.3 方法论启示
SQL技术：聚合函数、CASE条件、窗口函数（LAG）、多字段GROUP BY
 
分析思维：从单变量到多变量交叉，从描述到诊断



分析工具：SQLite | 数据来源：Kaggle Titanic | 记录数：891 
 
数据伦理：数据揭示结构性不平等，但需结合历史语境解读