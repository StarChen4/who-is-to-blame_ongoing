# 分析逻辑详解

本文档详细说明畜度计算和畜生判定的完整逻辑。

## 核心理念

**客观性**: 基于数据,而非主观印象
**公平性**: 考虑位置差异、克制关系、经济背景
**合理性**: 避免过度惩罚
**可解释性**: 每个罪状都有明确的数据支撑

## 分析流程

### 第一步: 位置识别

位置识别是分析的基础,因为不同位置的评判标准不同。

#### 识别规则

```
基于分路数据:
1. 中路玩家 → 2号位
2. 优势路且前10分正补>30 → 1号位
3. 劣势路且前10分正补>20 → 3号位
4. 优势路且前10分正补<15 → 5号位
5. 劣势路且前10分正补<10 → 4号位

特殊情况处理:
- 如果分路数据不清晰,按GPM降序排列:
  - 最高GPM → 1号位
  - 次高GPM → 2号位
  - 第三GPM → 3号位
  - 第四GPM → 4号位
  - 最低GPM → 5号位
```

#### 位置责任

| 位置 | 主要责任 | 评判重点 |
|-----|---------|---------|
| 1号位 | 后期输出 | 发育效率、团战输出、推进能力 |
| 2号位 | 节奏控制 | 对线优势、游走支援、团战贡献 |
| 3号位 | 前排抗压 | 对线能力、团战承伤、控制输出 |
| 4号位 | 节奏辅助 | 游走节奏、团战控制、**视野布置** |
| 5号位 | 保障辅助 | 保护核心、**视野布置(主要责任)**、团队牺牲 |

**视野责任说明**:
- 视野责任仅由4/5号位承担,1/2/3号位不评估视野相关失误
- 5号位承担主要视野责任(系数1.0)
- 4号位承担次要视野责任(系数0.8)

### 第二步: 失误识别

按照SSS到F级分类识别失误,每个失误贡献基础畜度。

#### SSS级失误 (基础畜度+100)

**1. 核心位对线大崩**

条件:
- 位置为1/2/3号位
- 前10分钟正补 < 对手同路玩家的50%
- 对线胜率 < 30%
- 对线未被克制的英雄

示例:
```
中单戴泽前10分52补刀 vs 对手30补刀
戴泽对线胜率54.71% → 不触发此失误

如果戴泽只有15补刀,对手30补刀
且对线胜率25% → 触发SSS级失误
```

**2. 核心位经济转化极差**

条件:
- 位置为1/2号位
- GPM排名队内前2
- 但英雄伤害 < 队内平均值的60%

计算:
```
经济转化率 = 英雄伤害 / GPM
如果核心位的转化率远低于队友 → SSS级失误
```

示例:
```
1号位剃刀: GPM 741, 伤害 47.4k, 转化率 = 63.97
2号位中单: GPM 523, 伤害 26.3k, 转化率 = 50.29

如果1号位转化率<30 → SSS级失误
```

**3. 辅助完全不参与**

条件:
- 位置为4/5号位
- 参团率 < 20%
- 视野得分(如果有) = 0

**4. 辅助视野完全缺失**

条件:
- 位置为4/5号位
- 侦查守卫购买 = 0 且 岗哨守卫购买 = 0
- 比赛时长 > 15分钟

示例:
```
5号位巫医全场0个守卫购买
比赛时长32分钟
→ 触发SSS级失误: 辅助视野完全缺失
```

#### SS级失误 (基础畜度+80)

**1. 对线严重劣势**

条件:
- 任何位置
- 对线胜率 < 30%
- 前10分补刀 < 对手70%

**2. 团战贡献极低**

条件:
- 核心位(1/2/3号位)
- 参团率 < 40%
- 团战平均伤害 < 队内平均值的50%

计算参团率:
```
参团率 = (总击杀 + 总助攻) / 全队总击杀
```

**3. 推进能力缺失**

条件:
- 1/2号位
- 防御塔伤害 < 1000
- 比赛时长 > 30分钟

**4. 辅助视野严重不足**

条件:
- 位置为5号位
- 侦查守卫使用 < 3 且 比赛时长 > 25分钟
- 或: 购买了守卫但使用率 < 30% (即: 使用数/购买数 < 0.3)

示例:
```
5号位全场只插了2个侦查守卫
比赛时长35分钟
→ 触发SS级失误: 辅助视野严重不足

或者:
5号位购买了10个守卫但只插了2个
使用率 = 2/10 = 20% < 30%
→ 触发SS级失误: 买眼不插
```

#### S级失误 (基础畜度+60)

**1. 对线失误**

条件:
- 对线胜率 < 45%
- 或前10分补刀明显低于预期

预期补刀:
```
1号位: >40
2号位: >35
3号位: >25
4号位: >10
5号位: >5
```

**2. 资源浪费**

条件:
- 经济占比 > 队内平均
- 但团战贡献 < 队内平均

示例:
```
某玩家NET 22.3k,队内最高
但团战平均伤害只有300,队内倒数
→ 资源浪费
```

**3. 团战严重失误**

条件:
- 多次团战(>3次)提前阵亡
- 或多次团战(>3次)未参与
- 核心位在关键团战中0输出

**4. 视野质量极差**

条件:
- 位置为4/5号位
- 守卫平均存活时长 < 30秒 (正常应>60秒)
- 且使用守卫数量 >= 3 (有一定样本量)

示例:
```
4号位插了5个侦查守卫
平均存活时长只有25秒
→ 触发S级失误: 视野质量极差(眼位选择有严重问题)
```

**注意**: 如果守卫数量<3个,样本量过小,不评估质量

#### A级失误 (基础畜度+40)

**1. 发育效率低**

条件:
- 核心位正补 < 预期的80%
- 或GPM明显低于同位置平均

**2. 团战站位失误**

条件:
- 控制时间过低(如果是控制英雄)
- 或过早死亡(死亡时团战才开始)

**3. 买活失误**

条件:
- 买活但未贡献(买活后立即死亡)
- 或关键时刻不买活

**4. 视野物品使用不积极**

条件:
- 位置为4/5号位
- 诡计之雾购买 = 0 且 比赛时长 > 30分钟
- 或: 对方有隐身英雄但显影之尘购买 = 0

示例:
```
对面有赏金猎人(隐身英雄)
4号位和5号位都没买过显影之尘
→ 两个辅助都触发A级失误: 缺乏反隐意识

或者:
比赛打了40分钟
4号位和5号位都没买过诡计之雾
→ 触发A级失误: 缺乏进攻性视野物品
```

#### B-F级失误 (基础畜度+10-30)

- 死亡过多(死亡>队内平均×1.5)
- 反补效率低(核心位<5个)
- 拉野次数低(辅助0次)
- 其他小失误

### 第三步: 系数调整

基础畜度需要根据实际情况调整。

#### 克制系数

**被克制情况**:

```python
countered_count = 0
total_counter_disadvantage = 0

for enemy_hero in enemy_team:
    if hero in enemy_hero.counters:
        win_rate = get_matchup_winrate(hero, enemy_hero)
        if win_rate < 43:
            countered_count += 1
            total_counter_disadvantage += (43 - win_rate)

if countered_count >= 3:
    克制系数 = 0.7
elif countered_count == 2:
    克制系数 = 0.8
elif countered_count == 1:
    克制系数 = 0.9
else:
    克制系数 = 1.0

# 严重克制额外降低
if total_counter_disadvantage > 30:
    克制系数 *= 0.9
```

示例:
```
戴泽被3个英雄克制,平均胜率40%
→ countered_count = 3
→ 克制系数 = 0.7
```

**克制对手情况**:

```python
counter_count = 0

for enemy_hero in enemy_team:
    if enemy_hero in hero.counters:
        win_rate = get_matchup_winrate(hero, enemy_hero)
        if win_rate > 57:
            counter_count += 1

if counter_count >= 3:
    克制系数 = 1.3
elif counter_count == 2:
    克制系数 = 1.2
elif counter_count == 1:
    克制系数 = 1.1
else:
    克制系数 = 1.0
```

#### 位置系数

```python
位置系数 = {
    1: 1.2,  # 大哥责任最大
    2: 1.2,  # 中单同样重要
    3: 1.1,  # 三号位次之
    4: 0.9,  # 四号位责任较轻
    5: 0.8   # 五号位责任最轻
}
```

**理由**: 核心位拥有更多资源,承担更大责任。

#### 经济背景系数

```python
def get_economic_context_factor(time, team_gold_diff):
    """
    time: 失误发生时间
    team_gold_diff: 当时全队经济差(正值表示领先)
    """
    if team_gold_diff < -5000:
        # 经济劣势期,失误可理解
        return 0.8
    elif team_gold_diff < -2000:
        return 0.9
    elif team_gold_diff > 5000:
        # 经济优势期还犯错,更严重
        return 1.2
    elif team_gold_diff > 2000:
        return 1.1
    else:
        return 1.0
```

示例:
```
某玩家在全队落后8000经济时对线劣势
→ 经济背景系数 = 0.8 (情有可原)

某玩家在全队领先10000经济时被单杀
→ 经济背景系数 = 1.2 (更不应该)
```

#### 视野责任系数(仅4/5号位)

```python
视野责任系数 = {
    5: 1.0,   # 5号位视野责任最重
    4: 0.8,   # 4号位视野责任次之
    # 1/2/3号位不评估视野责任
}
```

**视野畜度计算**:

```python
def calculate_vision_score(player, match_duration_min, enemy_supports, is_disadvantaged):
    """
    仅计算4/5号位的视野畜度
    1/2/3号位返回0

    参数:
    - enemy_supports: 对方4/5号位列表,用于态度对比
    - is_disadvantaged: 是否处于劣势(输方通常为True)
    """
    if player.position not in [4, 5]:
        return 0  # 核心位不评估视野责任

    vision_base_score = 0

    # 维度1: 积极性评估(购买和使用) - 权重1.0
    # 重点: 与对方辅助对比态度
    积极性得分 = evaluate_ward_activity_with_comparison(player, match_duration_min, enemy_supports)

    # 维度2: 能力评估(守卫存活时长) - 权重0.4(降低权重,因为劣势方被压制是正常的)
    能力得分 = evaluate_ward_quality(player.avg_ward_lifetime, player.ward_count, is_disadvantaged)

    vision_base_score = 积极性得分 + 能力得分 * 0.4

    # 应用视野责任系数
    视野畜度 = vision_base_score * 视野责任系数[player.position]

    return 视野畜度
```

**维度1: 积极性评估(与对方辅助对比)**

```python
def evaluate_ward_activity_with_comparison(player, match_duration_min, enemy_supports):
    """
    积极性评估 - 重点对比双方辅助的态度
    态度比能力更重要,因为态度是可控的
    """
    score = 0

    # 获取对方辅助的视野数据用于对比
    enemy_obs_total = sum(s.obs_used for s in enemy_supports)
    enemy_sentry_total = sum(s.sentry_used for s in enemy_supports)

    # === 绝对失误(不需要对比) ===

    # SSS级: 买了眼但一个不用(态度极差)
    if player.obs_purchased > 0 and player.obs_used == 0:
        score += 100

    # SSS级: 完全不买眼(15分钟以上)
    elif player.obs_purchased == 0 and player.sentry_purchased == 0 and match_duration_min > 15:
        score += 100

    # SS级: 使用率<30%(买了不插)
    elif player.obs_purchased > 0:
        usage_rate = player.obs_used / player.obs_purchased
        if usage_rate < 0.3:
            score += 80

    # === 相对失误(与对方辅助对比) ===

    # SS级: 对方辅助积极插眼,我方辅助摆烂
    # 条件: 对方辅助插眼数 >= 我方辅助的3倍
    if enemy_obs_total >= player.obs_used * 3 and enemy_obs_total >= 6:
        score += 80
        # 记录罪状: "对方辅助插了{X}个眼,你只插了{Y}个"

    # S级: 对方辅助积极插眼,我方辅助明显落后
    # 条件: 对方辅助插眼数 >= 我方辅助的2倍
    elif enemy_obs_total >= player.obs_used * 2 and enemy_obs_total >= 4:
        score += 60

    # A级: 对方辅助插眼数明显多于我方
    # 条件: 对方辅助插眼数 >= 我方辅助的1.5倍
    elif enemy_obs_total >= player.obs_used * 1.5 and enemy_obs_total >= 3:
        score += 40

    # === 视野物品使用 ===

    # A级: 30分钟以上0诡计之雾
    if player.smoke_purchased == 0 and match_duration_min > 30:
        score += 40

    # A级: 对方有隐身但0显影之尘
    if has_invisible_enemy() and player.dust_purchased == 0:
        score += 40

    return score
```

**维度2: 能力评估(考虑劣势减免)**

```python
def evaluate_ward_quality(avg_lifetime_sec, ward_count, is_disadvantaged):
    """
    能力评估 - 守卫存活时长

    重要: 劣势方被视野压制是正常的,存活时长较低不应过度惩罚
    """
    if ward_count < 3:
        return 0  # 样本量过小,不评估

    # 劣势方的存活时长阈值降低(被压制正常)
    if is_disadvantaged:
        # 劣势方阈值放宽
        if avg_lifetime_sec < 15:
            return 40  # 眼位极差(即使劣势也不该这么低)
        elif avg_lifetime_sec < 30:
            return 20  # 眼位较差但可理解
        elif avg_lifetime_sec < 45:
            return 10  # 轻微问题
        else:
            return 0   # 合格
    else:
        # 优势方/均势方正常阈值
        if avg_lifetime_sec < 20:
            return 60  # 眼位极差
        elif avg_lifetime_sec < 40:
            return 40  # 眼位较差
        elif avg_lifetime_sec < 60:
            return 20  # 眼位一般
        else:
            return 0   # 眼位合格
```

**特殊情况处理**:

```python
# 如果全队都不买眼,说明是团队问题,不单独惩罚某人
if team_total_obs_purchased == 0:
    # 降低个人视野失误的权重
    for player in supports:
        player.vision_score *= 0.5
    add_note("全队无人购买守卫,视野缺失是团队问题")

# 如果双方辅助插眼数量都很少,说明是比赛风格问题
if our_supports_obs_total < 5 and enemy_supports_obs_total < 5:
    # 不进行对比惩罚
    skip_comparison_penalty = True
    add_note("双方辅助插眼都少,可能是快节奏比赛")
```

#### 最终畜度计算

```python
# 视野畜度作为基础畜度的一部分
基础畜度 = 常规失误畜度 + 视野畜度

最终畜度 = 基础畜度 × 克制系数 × 位置系数 × 经济背景系数

# 确保畜度在合理范围
最终畜度 = max(0, min(200, 最终畜度))
```

### 第四步: 排名与判定

#### 排名规则

```python
# 按最终畜度降序排列
players_sorted = sort_by_final_score(players, descending=True)

# 输出排名
for i, player in enumerate(players_sorted):
    rank = i + 1
    print(f"#{rank} {player.name} ({player.hero}) - 畜度: {player.final_score}")
```

#### 畜生判定

```python
def determine_bastards(players_sorted):
    bastards = []

    # 第一名必定是畜生
    if players_sorted[0].final_score > 50:
        bastards.append(players_sorted[0])

    # 检查第二名
    if len(players_sorted) > 1:
        first_score = players_sorted[0].final_score
        second_score = players_sorted[1].final_score

        # 如果第二名畜度>80,也是畜生
        if second_score > 80:
            bastards.append(players_sorted[1])
        # 或者第二名畜度与第一名相差不大
        elif second_score > first_score * 0.8 and second_score > 60:
            bastards.append(players_sorted[1])

    # 位置平衡检查
    if len(bastards) == 2:
        positions = [b.position for b in bastards]
        # 如果两个畜生都是辅助(4/5号位),检查核心位
        if all(p >= 4 for p in positions):
            # 找出畜度最高的核心位
            for player in players_sorted:
                if player.position <= 3 and player.final_score > 40:
                    # 可能需要重新考虑
                    break

    return bastards[:2]  # 最多2个
```

#### 无畜生情况

如果所有玩家畜度都很低(<30),说明是团队整体问题:

```python
if max(player.final_score for player in players) < 30:
    return "本局无明显畜生,全队均有责任"
```

## 罪状生成

为每个畜生生成详细的罪状列表。

### 罪状模板

```python
def generate_charges(player, failures):
    charges = []

    for failure in failures:
        charge = {
            "level": failure.level,  # SSS, SS, S, A, B, C, D, E, F
            "description": failure.description,
            "data": failure.supporting_data
        }
        charges.append(charge)

    # 按等级排序
    charges.sort(key=lambda x: level_score[x['level']], reverse=True)

    return charges
```

### 罪状示例

```
【SSS级罪状】核心位对线大崩
- 你的前10分补刀: 15 (对手: 52)
- 对线胜率: 25.3% (严重劣势)
- 落后程度: 你只有对手的28.8%
- 数据来源: 分路页

【SS级罪状】团战贡献极低
- 你的参团率: 35% (队内最低)
- 团战平均伤害: 280 (队内平均: 650)
- 多场团战未参与: 8/15场
- 数据来源: 团战页

【S级罪状】资源严重浪费
- 你的经济: 22.3k (队内最高)
- 你的输出: 30k (队内第4)
- 经济转化率: 1.35 (队内最低,正常应>2.0)
- 数据来源: 概览页
```

## 特殊情况处理

### 1. 位置不清晰

如果分路数据不清晰(如游走型英雄):

```python
# 使用GPM作为备选判断
if player.lane == "未知" or player.lane_time_ratio < 0.5:
    # 根据GPM推断位置
    position = estimate_position_by_gpm(player, team)
```

### 2. 数据缺失

如果关键数据缺失:

```python
if missing_data("laning_stats"):
    # 跳过对线分析
    skip_laning_analysis()
else if missing_data("teamfight_stats"):
    # 使用KDA作为替代
    estimate_teamfight_from_kda()
```

### 3. 克制数据异常

如果ProTracker无法访问:

```python
if not available("protracker"):
    # 不使用克制系数调整
    克制系数 = 1.0
    # 在报告中说明
    add_note("克制数据暂时无法获取,未进行克制系数调整")
```

### 4. 全队表现差

如果所有人畜度都很高(>60):

```python
if avg_score > 60:
    # 说明是团队整体问题
    conclusion = "本局失利是团队整体问题,以下玩家表现尤其需要改进:"
    # 只列出前2名
    return top_2_players
```

## 分析示例

### 示例: 比赛8669004890分析

#### 输入数据

天辉(输方):
1. Ander (潮汐) - 3号位
2. ssdsdbcx (戴泽) - 2号位
3. 浅醉晚春 (巫医) - 4号位
4. XinG (赏金) - 3号位/游走
5. tatakai (蓝猫) - 5号位

#### 失误识别

**ssdsdbcx (戴泽)**:
- S级: 对线胜率54.71%,不算劣势,但作为中单对线优势不明显
- S级: 治疗量6.3k偏低(作为戴泽)
- A级: GPM 422,作为中单偏低

**tatakai (蓝猫)**:
- S级: 作为5号位走劣势路,对线胜率39.73%
- A级: 正补115,作为5号位过高(疑似位置错误)
- B级: 死亡11次,偏多

#### 畜度计算

**ssdsdbcx (戴泽)**:
```
基础畜度 = 60 + 60 + 40 = 160 (假设)
克制系数 = 0.9 (被1个英雄轻度克制)
位置系数 = 1.2 (中单)
经济背景系数 = 1.0 (均势)

最终畜度 = 160 × 0.9 × 1.2 × 1.0 = 172.8
```

**tatakai (蓝猫)**:
```
基础畜度 = 60 + 40 + 10 = 110
克制系数 = 1.0
位置系数 = 0.8 (5号位)
经济背景系数 = 1.0

最终畜度 = 110 × 1.0 × 0.8 × 1.0 = 88
```

#### 判定结果

1. 戴泽(畜度172.8) - **畜生**
2. 蓝猫(畜度88) - **畜生**(第二名且>80)
3. 其他玩家 - 正常

#### 报告输出

```markdown
## 🎯 畜生判定

经过科学分析,本场比赛的畜生是:

### 🐷 第一畜: ssdsdbcx (戴泽)
**畜度**: 172.8 / 200

**罪状**:
1. 【S级】中单对线优势不明显,未能建立节奏
2. 【S级】作为戴泽治疗量严重偏低
3. 【A级】发育效率低,GPM仅422

**数据对比**:
| 指标 | 戴泽 | 队内平均 | 对手中单 |
|-----|------|---------|---------|
| GPM | 422 | 369 | 523 |
| 对线胜率 | 54.71% | - | 54.71% |
| 治疗量 | 6.3k | 6.8k | - |

### 🐷 第二畜: tatakai (蓝猫)
**畜度**: 88 / 200

**罪状**:
1. 【S级】劣势路对线失误,胜率仅39.73%
2. 【B级】死亡过多(11次)

...
```

## 总结

完整的分析流程:

1. ✅ 识别位置
2. ✅ 识别所有失误(SSS-F级)
3. ✅ 计算基础畜度
4. ✅ 应用系数调整(克制、位置、经济背景)
5. ✅ 排名并判定畜生(最多2个)
6. ✅ 生成详细罪状和数据对比
7. ✅ 输出幽默的惩罚建议

**核心原则**: 数据驱动、公平客观、可解释性强
