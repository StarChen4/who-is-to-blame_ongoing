# 错误处理指南

本文档说明如何优雅地处理各种异常情况,确保用户体验良好。

## 处理原则

1. **用户友好**: 错误提示清晰易懂,不使用技术术语
2. **提供建议**: 告诉用户如何解决问题
3. **降级处理**: 部分数据缺失时仍能提供分析结果
4. **进度透明**: 让用户知道当前进度和预计等待时间

## 常见错误场景

### 1. 比赛ID无效

**场景**: 用户输入的不是有效的比赛ID

**检测方式**:
```python
def validate_match_id(match_id):
    # 比赛ID应该是8-10位数字
    if not match_id.isdigit():
        return False
    if len(match_id) < 8 or len(match_id) > 10:
        return False
    return True
```

**错误提示**:
```
❌ 比赛ID格式不正确

你输入的"{match_id}"不是有效的DOTA2比赛ID。

有效的比赛ID应该是8-10位数字,例如: 8669004890

💡 如何获取比赛ID:
1. 打开DOTA2游戏客户端
2. 进入"观看"标签页
3. 在"比赛历史"中找到你的比赛
4. 复制比赛ID
```

### 2. 比赛不存在

**场景**: 比赛ID格式正确但OpenDota上找不到

**检测方式**:
```python
def check_match_exists(match_id):
    # 访问OpenDota页面
    # 检查是否有"Match not found"或404错误
    if "not found" in page_content or status_code == 404:
        return False
    return True
```

**错误提示**:
```
❌ 未找到比赛 {match_id}

可能的原因:
1. 比赛太新,OpenDota还未收录(通常需要几分钟)
2. 比赛ID输入错误
3. 这是一场非天梯比赛,OpenDota未记录

💡 建议:
- 等待3-5分钟后重试
- 检查比赛ID是否正确
- 确认这是一场天梯比赛
```

### 3. 比赛未分析

**场景**: 比赛存在但OpenDota未进行详细分析

**检测方式**:
```python
def check_match_parsed(match_id):
    # 查找"重新分析"或"分析"按钮
    # 或检查是否有详细数据(如分路、团战等)
    if has_parse_button and not has_detailed_data:
        return False
    return True
```

**处理方式**:
```python
if not check_match_parsed(match_id):
    print("⏳ 检测到比赛尚未分析,正在触发分析...")

    # 点击"分析"按钮
    click_parse_button()

    print("⏳ 分析已开始,这通常需要1-2分钟,请稍候...")

    # 等待分析完成
    max_wait = 120  # 最多等待2分钟
    wait_time = 0

    while wait_time < max_wait:
        time.sleep(10)
        wait_time += 10

        if check_match_parsed(match_id):
            print("✅ 分析完成!")
            break

        print(f"⏳ 仍在分析中... ({wait_time}/{max_wait}秒)")

    if not check_match_parsed(match_id):
        print("⚠️ 分析超时,将使用可用数据进行分析")
```

**错误提示**(如果分析失败):
```
⚠️ 比赛分析未完成

OpenDota正在分析这场比赛,但尚未完成。

当前状态:
- 基础数据: ✅ 可用
- 详细数据(分路/团战): ❌ 暂时不可用

💡 你可以:
1. 等待几分钟后使用 `/抓畜 {match_id}` 重新分析
2. 继续使用当前可用数据(分析结果可能不完整)

请输入 "继续" 使用当前数据,或 "取消" 结束分析:
```

### 4. 网络错误

**场景**: 无法访问OpenDota或ProTracker

**检测方式**:
```python
def check_network_access(url):
    try:
        response = requests.get(url, timeout=10)
        return response.status_code == 200
    except Exception as e:
        return False
```

**错误提示**:
```
❌ 网络连接失败

无法访问{service_name},请检查网络连接。

错误详情: {error_message}

💡 建议:
- 检查你的网络连接
- 稍后重试
- 如果问题持续,可能是{service_name}暂时不可用
```

### 5. 登录要求

**场景**: OpenDota某些功能需要登录

**检测方式**:
```python
def check_login_required():
    # 查找"登录"按钮或提示
    if "登录" in page_content or "sign in" in page_content.lower():
        return True
    return False
```

**错误提示**:
```
⚠️ 需要登录OpenDota

部分数据需要登录OpenDota账号才能访问。

💡 如何登录:
1. 访问 https://www.opendota.com/
2. 点击右上角"登录"按钮
3. 使用Steam账号登录
4. 登录后重新运行 `/抓畜 {match_id}`

或者,我可以尝试使用可用数据继续分析(可能不完整)。

继续吗? (是/否)
```

### 6. ProTracker访问失败

**场景**: 无法获取克制数据

**处理方式**:
```python
def get_counter_data_with_fallback(hero_name):
    try:
        return fetch_from_protracker(hero_name)
    except Exception as e:
        print(f"⚠️ 无法获取{hero_name}的克制数据,将跳过克制系数调整")
        return None

# 在分析中
if counter_data is None:
    克制系数 = 1.0  # 使用默认值
    add_note("克制数据暂时无法获取,未进行克制系数调整")
```

**用户提示**:
```
⚠️ 克制数据获取失败

ProTracker暂时无法访问,将跳过克制系数调整。

这不会影响主要分析,但可能影响最终畜度评分的准确性。

继续分析中...
```

### 7. 数据不完整

**场景**: 部分玩家数据缺失

**处理方式**:
```python
def handle_incomplete_data(player):
    missing_fields = []

    if not has_laning_data(player):
        missing_fields.append("对线数据")
        player.skip_laning_analysis = True

    if not has_teamfight_data(player):
        missing_fields.append("团战数据")
        player.skip_teamfight_analysis = True

    if missing_fields:
        print(f"⚠️ {player.name}的{', '.join(missing_fields)}缺失,将跳过相关分析")

    return player
```

**在报告中说明**:
```
## ⚠️ 数据说明

以下数据在分析时不可用:
- ssdsdbcx的团战数据缺失,未计算参团率
- tatakai的对线数据缺失,未分析对线表现

分析结果基于可用数据,可能不完全准确。
```

### 8. 赢的一方误触发

**场景**: 用户想分析赢的一方(违反规则)

**检测与提示**:
```python
def analyze_match(match_id):
    winner = get_winner(match_id)

    print(f"""
本场比赛胜者: {winner}

⚠️ 注意: 本工具只分析输掉比赛的一方。

原因: 赢的一方没有畜生,只有英雄!

将分析输方: {'天辉' if winner == '夜魇' else '夜魇'}

继续分析中...
    """)

    loser_side = '天辉' if winner == '夜魇' else '夜魇'
    return analyze_team(loser_side)
```

### 9. 平局或特殊比赛

**场景**: 比赛没有明确胜负

**检测方式**:
```python
def check_match_result(match_data):
    if match_data.radiant_win is None:
        return "draw"
    return "normal"
```

**错误提示**:
```
❌ 无法分析此比赛

此比赛似乎是平局或特殊模式比赛,无法判定胜负。

本工具仅支持分析有明确胜负的天梯比赛。

💡 建议: 尝试分析其他比赛
```

### 10. 数据解析错误

**场景**: 页面结构变化导致数据提取失败

**处理方式**:
```python
def safe_extract_data(selector, default=None):
    try:
        return extract_data(selector)
    except Exception as e:
        log_error(f"数据提取失败: {selector}, 错误: {e}")
        return default

# 使用
gpm = safe_extract_data("td.gpm", default=0)
if gpm == 0:
    print(f"⚠️ 无法提取GPM数据,将使用估算值")
    gpm = estimate_gpm_from_net_worth(net_worth, game_duration)
```

**错误提示**:
```
⚠️ 部分数据提取失败

OpenDota页面结构可能已更新,导致部分数据无法正常提取。

已使用估算值继续分析,结果可能不完全准确。

如果问题持续,请报告此问题。
```

## 进度提示模板

### 完整流程的进度提示

```python
def analyze_with_progress(match_id):
    print("🔍 开始分析比赛{match_id}")

    print("📥 [1/7] 正在获取比赛基础数据...")
    overview_data = fetch_overview(match_id)

    print("⚖️ [2/7] 正在判定胜负...")
    winner = determine_winner(overview_data)
    print(f"   ✅ 胜者: {winner}, 将分析输方")

    print("📊 [3/7] 正在获取详细数据...")
    print("   - 对线数据...")
    laning_data = fetch_laning(match_id)
    print("   - 表现数据...")
    performance_data = fetch_performance(match_id)
    print("   - 团战数据...")
    teamfight_data = fetch_teamfights(match_id)

    print("🎭 [4/7] 正在获取克制数据 (0/5)...")
    counter_data = {}
    for i, hero in enumerate(losing_heroes, 1):
        print(f"   - {hero}... ({i}/5)")
        counter_data[hero] = fetch_counter_data(hero)

    print("🔬 [5/7] 正在分析数据...")
    print("   - 识别位置...")
    identify_positions(players)
    print("   - 识别失误...")
    identify_failures(players)
    print("   - 计算畜度...")
    calculate_scores(players)

    print("🎯 [6/7] 正在生成报告...")
    report = generate_report(players)

    print("💾 [7/7] 正在保存结果...")
    save_report(match_id, report)

    print("✅ 分析完成!")
    print(f"📄 报告已保存到: /d/projects/who-is-to-blame/results/{match_id}.md")

    return report
```

## 用户交互

### 确认继续(数据不完整时)

```python
def ask_user_continue(message):
    response = AskUserQuestion(
        questions=[{
            "question": message,
            "header": "继续分析?",
            "multiSelect": False,
            "options": [
                {
                    "label": "是,继续",
                    "description": "使用可用数据继续分析"
                },
                {
                    "label": "否,取消",
                    "description": "停止分析"
                }
            ]
        }]
    )

    return response["继续分析?"] == "是,继续"
```

### 选择分析模式

```python
def ask_analysis_mode():
    response = AskUserQuestion(
        questions=[{
            "question": "检测到部分数据缺失,请选择分析模式:",
            "header": "分析模式",
            "multiSelect": False,
            "options": [
                {
                    "label": "快速分析",
                    "description": "仅使用基础数据,速度快但不够详细"
                },
                {
                    "label": "完整分析",
                    "description": "等待所有数据加载,更准确但需要更多时间"
                },
                {
                    "label": "取消",
                    "description": "停止分析"
                }
            ]
        }]
    )

    return response["分析模式"]
```

## 日志记录

### 错误日志

```python
import logging
from datetime import datetime

def setup_logging():
    log_file = f"/d/projects/who-is-to-blame/data/logs/{datetime.now().strftime('%Y%m%d')}.log"

    logging.basicConfig(
        filename=log_file,
        level=logging.INFO,
        format='%(asctime)s - %(levelname)s - %(message)s'
    )

def log_error(match_id, error_type, error_details):
    logging.error(f"Match {match_id} - {error_type}: {error_details}")

def log_warning(match_id, warning_type, warning_details):
    logging.warning(f"Match {match_id} - {warning_type}: {warning_details}")

def log_info(match_id, message):
    logging.info(f"Match {match_id} - {message}")
```

### 使用示例

```python
try:
    data = fetch_data(match_id)
except NetworkError as e:
    log_error(match_id, "NetworkError", str(e))
    print_user_friendly_error("网络连接失败")
except ParseError as e:
    log_error(match_id, "ParseError", str(e))
    print_user_friendly_error("数据解析失败")
```

## 降级策略

### 数据缺失的降级

```python
class AnalysisStrategy:
    FULL = "full"        # 所有数据可用
    PARTIAL = "partial"  # 部分数据可用
    MINIMAL = "minimal"  # 仅基础数据可用

def determine_strategy(available_data):
    if has_all_data(available_data):
        return AnalysisStrategy.FULL

    if has_basic_and_laning(available_data):
        return AnalysisStrategy.PARTIAL

    return AnalysisStrategy.MINIMAL

def analyze_with_strategy(players, strategy):
    if strategy == AnalysisStrategy.FULL:
        return full_analysis(players)

    elif strategy == AnalysisStrategy.PARTIAL:
        print("⚠️ 使用部分数据模式分析")
        return partial_analysis(players)

    else:
        print("⚠️ 使用最小数据模式分析")
        return minimal_analysis(players)
```

### 克制数据降级

```python
def get_counter_factor_with_fallback(hero, enemies):
    try:
        return calculate_counter_factor(hero, enemies)
    except CounterDataUnavailable:
        print(f"⚠️ {hero}的克制数据不可用,使用默认系数1.0")
        return 1.0
```

## 总结

错误处理的关键点:

1. ✅ 永远提供清晰的错误信息
2. ✅ 告诉用户如何解决问题
3. ✅ 在可能的情况下降级处理,而不是完全失败
4. ✅ 保持用户知情(进度提示)
5. ✅ 记录错误日志供调试
6. ✅ 优雅的用户交互(使用AskUserQuestion)

**核心理念**: 让用户即使遇到错误也能获得良好的体验
