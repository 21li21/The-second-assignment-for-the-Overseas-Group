# assigment2
# 小组分工方案

## 文档组（2人）

### 李婧涵同学
- 负责第一个项目的设计思路文档（`assignment2.md`中设计部分）
  - 系统架构设计
  - 主题可配置化方案
  - 数据流设计

### 田嘉铭同学
- 负责第二个项目的设计思路文档（`assignment2.md`中设计部分）
  - 系统架构设计
  - 主题可配置化方案
  - 数据流设计

---

## 代码组（4人，分两组）

### 第一组（项目1：胖猫自杀事件）

#### 刘星雨同学
- 数据采集模块
  - 对接搜索工具API（如serpAPI）
  - 实现微博/知乎等平台的数据抓取

#### 郭梓良同学
- 前端开发
  - 实现主题配置界面
  - 展示监测结果的动态列表

### 第二组（项目2：华中农大学术不端事件）

#### 张君仪同学
- 大模型API对接
  - 使用siliconflow.cn或其他大模型
  - 实现文本语义分析、情感分类等功能

#### 钟圣曦同学
- 后端与系统整合
  - 整合前后端数据流
  - 完善异常处理与测试

---

# 项目一开发过程文档说明

## 一、开发背景
针对网络热点事件（如"胖猫自杀事件"）的舆情分析需求，开发本工具实现以下功能：
- 📡 实时数据采集：抓取新闻标题、内容等关键信息
- 🤖 情感倾向分析：通过大语言模型（如DeepSeek）判断舆情正负
- 🚨 自动化预警：识别敏感内容后触发告警，辅助舆情管控

## 二、技术选型
| 模块          | 技术栈                     | 作用                          |
|---------------|---------------------------|-------------------------------|
| 环境管理      | Python 3.10 + venv虚拟环境 | 隔离依赖                      |
| 数据采集      | requests + SerpAPI         | 爬取公开新闻                  |
| 敏感词检测    | dotenv + 环境变量          | 安全存储API密钥               |
| 深度分析      | DeepSeek API               | 文本情感分类                  |
| 开发工具      | PyCharm + Git              | 代码调试与版本控制            |

## 三、实现步骤如下


pythonProject1/
├── venv/                    # 虚拟环境
├── lib/
│   ├── .gitignore           # 忽略配置文件
│   └── pydeck.json          # 数据可视化配置（可选）
├── network_monitoring_tool.py  # 主程序
└── network_monitoring_utils.py # 工具函数

# 创建虚拟环境并激活
python -m venv venv
.\venv\Scripts\activate

# 安装依赖
pip install requests python-dotenv serpapi

import os
from dotenv import load_dotenv

load_dotenv()  # 加载环境变量

def get_api_keys():
    """验证API密钥是否存在"""
    deepseek_key = os.getenv("DEEPSEEK_API_KEY")
    serp_key = os.getenv("SERP_API_KEY")
    if not deepseek_key or not serp_key:
        raise ValueError("请配置 .env 文件中的 API 密钥！")
    return deepseek_key, serp_key


import requests
from serpapi import GoogleSearch
from .utils import get_api_keys

# 全局变量
EVENT_TOPIC = "胖猫自杀事件"  # 可配置化
MAX_RESULTS = 10  # 搜索结果数量

def fetch_news(keyword, num=MAX_RESULTS):
    """抓取新闻数据"""
    params = {
        "q": keyword,
        "num": num,
        "hl": "zh-CN",
        "api_key": get_api_keys()[1]  # SerpAPI密钥
    }
    search = GoogleSearch(params)
    results = search.get_dict()
    return [result["title"] + " " + result["snippet"] for result in results["organic_results"]]

def analyze_sentiment(news_list):
    """情感分析核心逻辑"""
    from deepseek import ChatCompletion  # 动态加载避免启动报错
    
    deepseek_key, _ = get_api_keys()
    client = ChatCompletion(api_key=deepseek_key)
    
    news_text = "\n".join(news_list)
    response = client.chat.completions.create(
        messages=[
            {"role": "system", "content": "你是一个舆情分析师，需判断以下新闻内容的情感倾向（正/负/中立）"},
            {"role": "user", "content": news_text}
        ],
        model="deepseek-chat",
        temperature=0.1
    )
    return response.choices[0].message["content"].strip()
    
def monitor():
    """主流程控制"""
    print(f"正在获取关于 '{EVENT_TOPIC}' 的新闻数据...")
    news_data = fetch_news(EVENT_TOPIC)
    if not news_data:
        print("未找到相关新闻。")
        return
    
    sentiment_result = analyze_sentiment(news_data)
    print(f"舆情趋势分析结果：{sentiment_result}")
    
    if "负" in sentiment_result:
        print("\033[31m[警报] 检测到敏感舆情，请及时处理！\033[0m")

if __name__ == "__main__":
    monitor()


python network_monitoring_tool.py

![301eeb0455d3216f092de73d7511553](https://github.com/user-attachments/assets/e11c2763-098c-4920-90ed-41ce359ae404)


# 学术举报事件舆情监测系统开发文档

## 一、开发背景
针对学术举报类事件（如"华中农大举报导师学术造假"）的舆情分析需求，开发本系统实现以下功能：
- 📡 多阶段舆情监测：自动识别舆情发展的不同阶段（爆发期、发酵期、分化期、长尾期）
- 🤖 深度情感分析：通过大语言模型进行多维度情感倾向分析
- 🚨 风险预警：识别敏感内容和舆情风险点
- 📊 趋势可视化：生成舆情发展曲线和情感分布图表

## 二、技术选型
| 模块         | 技术栈                      | 作用描述                     |
|--------------|----------------------------|----------------------------|
| 数据采集     | SerpAPI + Google News      | 获取多平台新闻数据           |
| 情感分析     | DeepSeek API               | 深度文本情感分析             |
| 阶段识别     | 关键词匹配+时间序列分析     | 判断舆情发展阶段             |
| 风险预警     | 自定义敏感词库+情感阈值     | 触发舆情风险警报             |
| 可视化       | Pyecharts                  | 生成舆情趋势图表             |
| 开发环境     | Python 3.10 + venv         | 环境隔离与管理               |

## 三、系统架构
```
舆情监测系统/
├── config/
│   ├── keywords.json    # 各阶段关键词配置
│   └── sensitive_words.txt  # 敏感词库
├── src/
│   ├── data_collector.py   # 数据采集模块
│   ├── phase_analyzer.py   # 阶段分析模块
│   ├── sentiment_analysis.py # 情感分析模块
│   └── visualization.py    # 可视化模块
├── outputs/
│   ├── reports/        # 分析报告
│   └── charts/        # 生成图表
└── main.py            # 主入口
```

## 四、核心功能实现

### 1. 多阶段舆情监测
```python
# phase_analyzer.py
EVENT_PHASES = {
    "爆发期": ["举报", "曝光", "热搜", "实名"],
    "发酵期": ["调查", "回应", "质疑", "证据"],
    "分化期": ["结果", "处理", "争议", "处罚"],
    "长尾期": ["政策", "影响", "反思", "制度"]
}

def detect_phase(news_data: list):
    """通过关键词匹配判断当前舆情阶段"""
    phase_scores = {phase: 0 for phase in EVENT_PHASES}
    for text in news_data:
        for phase, keywords in EVENT_PHASES.items():
            phase_scores[phase] += sum(1 for kw in keywords if kw in text)
    return max(phase_scores.items(), key=lambda x: x[1])[0]
```

### 2. 深度情感分析
```python
# sentiment_analysis.py
def analyze_sentiment(topic, news_list):
    prompt = f"""作为舆情分析师，请对以下关于【{topic}】的新闻内容进行多维分析：
    1. 情感倾向（正面/负面/中立）
    2. 主要情绪（愤怒/同情/质疑等）
    3. 讨论焦点（学术诚信/导师权力/学生权益等）
    4. 风险等级评估（低/中/高）
    
    新闻内容：
    {chr(10).join(news_list)}
    """
    # 调用DeepSeek API实现...
```

### 3. 风险预警系统
```python
# alert_system.py
CRITICAL_KEYWORDS = ["撤稿", "学术不端", "压榨", "报复", "轻描淡写"]
WARNING_THRESHOLDS = {
    "负面情感比例": 0.6,
    "愤怒情绪占比": 0.4,
    "日均新增报道": 50
}

def check_risks(analysis_result):
    """综合评估舆情风险"""
    risks = []
    if analysis_result["negative_ratio"] > WARNING_THRESHOLDS["负面情感比例"]:
        risks.append("高负面情绪")
    if any(kw in analysis_result["summary"] for kw in CRITICAL_KEYWORDS):
        risks.append("敏感关键词触发")
    return risks
```

## 五、舆情分析模型（以学术举报事件为例）

### 1. 舆情爆发期
- **监测重点**：举报内容传播速度、首发平台影响力、关键词热度
- **技术实现**：实时监控社交媒体API，检测关键词突增

### 2. 舆情发酵期
- **监测重点**：校方回应时效性、媒体报道角度变化、专家参与度
- **技术实现**：建立回应时效评估模型，跟踪信源分布

### 3. 舆情分化期
- **监测重点**：调查结果公信力、舆论场分化程度、次生议题
- **技术实现**：观点聚类分析，对立观点识别

### 4. 舆情长尾期
- **监测重点**：政策影响范围、同类事件关联度、国际影响
- **技术实现**：长期趋势预测模型，跨事件关联分析

## 六、系统输出示例
```
[舆情日报] 华中农大举报事件 - 2023-12-15
----------------------------------------
当前阶段：发酵期（已持续3天）
情感分布：负面72% | 中立23% | 正面5%
主要情绪：愤怒（58%）、质疑（27%）、同情（15%）
热点议题：导师权力监管（43%）、举报制度（32%）、论文审核（25%）
风险警报：⚠️ 高负面情绪 | ⚠️ 敏感词"压榨"高频出现
趋势预测：未来3天热度可能上升20-30%
```

## 七、部署与优化建议
1. **数据源扩展**：接入微博、知乎等社交平台API
2. **模型优化**：加入时间序列预测算法
3. **响应机制**：与企业微信/钉钉报警系统集成
4. **性能优化**：采用异步IO处理大规模请求

## 八、注意事项
1. 学术类舆情具有专业性强、周期长的特点，建议：
   - 配置学科专业术语库
   - 延长监测周期（建议至少3个月）
   - 建立专家咨询接口
2. 遵守《网络安全法》相关规定，不存储原始数据










