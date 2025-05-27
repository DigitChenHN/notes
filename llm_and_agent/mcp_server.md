## MCP（Model Context Protocol）
MCP是模型上下文协议的缩写，用来标准化大语言模型的上下文格式。如何理解 MCP 可以参看 [什么是 function call、MCP 和 agent](/llm_and_agent/什么是function%20call、mcp和agent/什么是function%20call、mcp和agent.md)。  
本文将根据mcp官方文档的[案例](https://mcp-docs.cn/quickstart/server)来解释一下如何构建一个mcp server。  

## 构建一个天气服务器  

### 导入packages并设置instance
```python
from typing import Any
import httpx
from mcp.server.fastmcp import FastMCP

# 初始化 FastMCP server
mcp = FastMCP("weather")

# Constants
NWS_API_BASE = "https://api.weather.gov"
USER_AGENT = "weather-app/1.0"
```
> FastMCP class 使用 Python type hints 和 docstrings 来自动生成 tool definitions，从而轻松创建和维护 MCP tools。  

Python type hints 是 Python 3.5 引入的一个特性，用于在代码中添加类型注解。
比如在定义一个函数时：  
```python 
from typing import Any

def get_weather(location: str) -> dict[str, Any]:
    """
    获取指定位置的天气信息。  
    Args:
        location (str): 位置的名称或坐标。
    """
    location: str = 'San Francisco, CA'
```
这种在变量后面添加上类型的形式就是所谓的 type hints。python是一种动态语言，不需要像c++一样在声明变量时指定类型，也就说这种注解本质上不会影响代码的运行，而是作为一种文档和提示的作用。  
docstrings 是 Python 中用于为函数、类和模块提供文档的字符串。它们通常位于定义的第一行，并用三重引号括起来。  

我们知道mcp和agent的核心是给llm的prompt中添加工具函数的描述，因此这些 type hints 和 docstrings 在mcp server的开发中就尤为重要。  


### 定义helper functions
```python
async def make_nws_request(url: str) -> dict[str, Any] | None:
    """向 NWS API 发送请求，并进行适当的错误处理。"""
    headers = {
        "User-Agent": USER_AGENT,
        "Accept": "application/geo+json"
    }
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(url, headers=headers, timeout=30.0)
            response.raise_for_status()
            return response.json()
        except Exception:
            return None

def format_alert(feature: dict) -> str:
    """将警报 feature 格式化为可读的字符串。"""
    props = feature["properties"]
    return f"""
事件: {props.get('event', 'Unknown')}
区域: {props.get('areaDesc', 'Unknown')}
严重性: {props.get('severity', 'Unknown')}
描述: {props.get('description', 'No description available')}
指示: {props.get('instruction', 'No specific instructions provided')}
"""
```
定义两个辅助函数，在我们预想的流程中，这些函数不需要被llm直接调用到。  

### 实现tool execution  
```python
@mcp.tool()
async def get_alerts(state: str) -> str:
    """获取美国州的天气警报。

    Args:
        state: 两个字母的美国州代码（例如 CA、NY）
    """
    url = f"{NWS_API_BASE}/alerts/active/area/{state}"
    data = await make_nws_request(url)

    if not data or "features" not in data:
        return "无法获取警报或未找到警报。"

    if not data["features"]:
        return "该州没有活跃的警报。"

    alerts = [format_alert(feature) for feature in data["features"]]
    return "\n---\n".join(alerts)

@mcp.tool()
async def get_forecast(latitude: float, longitude: float) -> str:
    """获取某个位置的天气预报。

    Args:
        latitude: 位置的纬度
        longitude: 位置的经度
    """
    # 首先获取预报网格 endpoint
    points_url = f"{NWS_API_BASE}/points/{latitude},{longitude}"
    points_data = await make_nws_request(points_url)

    if not points_data:
        return "无法获取此位置的预报数据。"

    # 从 points response 中获取预报 URL
    forecast_url = points_data["properties"]["forecast"]
    forecast_data = await make_nws_request(forecast_url)

    if not forecast_data:
        return "无法获取详细预报。"

    # 将 periods 格式化为可读的预报
    periods = forecast_data["properties"]["periods"]
    forecasts = []
    for period in periods[:5]:  # 仅显示接下来的 5 个 periods
        forecast = f"""
{period['name']}:
温度: {period['temperature']}°{period['temperatureUnit']}
风: {period['windSpeed']} {period['windDirection']}
预报: {period['detailedForecast']}
"""
        forecasts.append(forecast)

    return "\n---\n".join(forecasts)
```
这两个函数是具体的工具函数，是整个过程中直接提供给llm进行调用的函数，通过装饰器 `@mcp.tool()` 来注册到 MCP server 中。（关于装饰器的理解在[这里](/python/decorator.md)提到过）  
这里的关键是mcp这个FastMCP示例的方法`tool()`，这是一个装饰器函数。这个装饰器会将工具函数注册到当前的mcp对象中。  

### 启动服务器
```python   
if __name__ == "__main__":
    # 初始化并运行 server
    mcp.run(transport='stdio')
```

