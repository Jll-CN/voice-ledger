## Project Description 
Say a sentence, enter text, click a button, and automatically write the expenditure record into Google Sheets in a lazy way.

### System Input

| Input | Example |
| :-----: | :----- |
| From chat box |  eg. I had lunch for noodles. It cost 18RMB|
| From  frontend |   I had beef noodle soup for lunch today and it cost 20 RMB. |

### Qucikly use
1. Use the front-end voice input buttons or use the platform-provided chatbox
2. Just say the expenses. Eg. I spent 20 yuan on a taxi yesterday, or I ate 5 yuan for breakfast today.
3. System feedback, successful operation information.
4. Open Google Sheets to see new records

### Architecture Description
Input ---> HTTP Node ---> AI Agent ----> DeepSeek Analysis ---> Save to Google Sheets.
| Node               | Function                                                  | Key Configuration          |
| :----------------- | :-------------------------------------------------------- | :------------------------- |
| HTTP In (Webhook)  | Unified entry point that receives JSON:<br>`{"text":"60 yuan for taxi ride","user":"zhang"}` | Method: POST               |
| AI Agent           | Converts raw text to structured JSON according to Prompt  | See "Prompt Template" below |
| DeepSeek (LLM)     | Secondary verification and completion of missing fields (e.g., date, currency) | temperature=0.1 for stability |
| Google Sheets      | Writes fields to specified worksheet                       | Authorization + Sheet ID only |

### Agent Prompt
```You are a Chinese expense extractor.
Return JSON only:
{
  "date": "yyyy-mm-dd",
  "amount": <纯数字>,
  "category": "<food|transport|...>",
  "desc": "<事件描述>",
  "raw": "<用户原文>"
}

日期计算规则（必读）
统一基准
当前真实日期由调用 get_date 获得，格式固定为 yyyy-mm-dd。
该日期即为“今天”。
相对日期映射表
• “今天” → 基准日
• “昨天” → 基准日 − 1 天
• “前天” → 基准日 − 2 天
• “明天” → 基准日 + 1 天（如出现）
计算步骤
(1) 调用 get_date 得到 today。
(2) 根据上表计算目标日期，格式化为 yyyy-mm-dd。
(3) 将结果写入返回 JSON 的 "date" 字段。
边界处理
• 若出现未列出的相对词（如“上周三”），强制使用 get_date 返回值。
• 若出现绝对日期（如“2025-01-30”），直接原样使用，不再计算。

示例1（有相对日期）：
输入："2025-06-21，我吃早餐花了10.5元"。
输出：
{
  "date": "2025-06-21",
  "amount": 10.5,
  "category": "food",
  "desc": "买豆浆",
  "raw": "今天早上买豆浆花了 10.5 元"
}
示例2（无日期）：
{
  "date": "2025-06-21",
  "amount": 25,
  "category": "food",
  "desc": "吃拉面",
  "raw": "中午吃拉面，25"
}
```

### Setup
1. Open n8n → Workflows → Import → Select workflow.json
2. Fill in credentials:
   - DeepSeek API Key
   - Google Sheets OAuth JSON
3. The frontend application needs to be updated to use the backend request path.

### Google Sheets
| date       | amount | category  | description | raw_input                  |
| :--------- | :----- | :-------- | :---------- | :------------------------- |
| 2025-06-01 | 10     | transport | 打车        | 今天打车花十块钱。         |
| 2025-05-31 | 15     | food      | 吃牛肉面    | 昨天吃牛肉面花了15块钱。   |
| 2025-05-31 | 10     | food      | 早饭        | 昨天早饭花了十块。         |