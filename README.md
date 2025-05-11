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











