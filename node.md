
<h1>coze_agent 本地调用</h1>

```py
import requests

personal_access_token = 'pat_3iRAsZFNkQc7FAFpzqkKutrcyniiymwXnYcTfTuHkCXJeXz6TnKbtkodjzDwbkNp'
bot_id = '7661930659459956777'

headers = {
    'Authorization': f'Bearer {personal_access_token}',
    'Content-Type': 'application/json',
    'Accept': '*/*',
    'Connection': 'keep-alive'
}

def call_coze_api(query, conversation_id=None):
    payload = {
        "bot_id": bot_id,
        "user": "123",
        "query": query,
        "stream": False
    }
    
    if conversation_id:
        payload["conversation_id"] = conversation_id
    
    try:
        response = requests.post(
            'https://api.coze.cn/open_api/v2/chat',
            headers=headers,
            json=payload,
            timeout=30
        )
        
        if response.ok:
            data = response.json()
            if data.get("code") == 0:
                messages = data.get("messages", [])
                answer = ""
                for msg in messages:
                    if msg.get("type") == "answer" and msg.get("content_type") == "text":
                        answer = msg.get("content", "")
                return answer, data.get("conversation_id")
            else:
                return f"API错误: {data.get('msg', '未知错误')}", None
        else:
            return f"请求失败 ({response.status_code})", None
            
    except requests.exceptions.RequestException as e:
        return f"网络异常: {e}", None

def main():
    print("=" * 50)
    print("      Coze AI 对话助手")
    print("=" * 50)
    print("输入消息开始对话，输入 'exit' 或 'quit' 退出")
    print("-" * 50)
    
    conversation_id = None
    
    while True:
        try:
            user_input = input("\n你: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\n\n再见！")
            break
        
        if user_input.lower() in ("exit", "quit", "退出"):
            print("\n再见！")
            break
        
        if not user_input:
            print("请输入内容")
            continue
        
        print("AI: ", end="", flush=True)
        
        answer, conv_id = call_coze_api(user_input, conversation_id)
        
        if conv_id:
            conversation_id = conv_id
        
        print(answer)

if __name__ == "__main__":
    main()
```

<h2>准备工作</h2>

1. <font size = 3>personal_access_token =". . . . ."</font> &ensp; 个人令牌&ensp; 在coze->API管理->授权->个人访问令牌中获取
2. <font size = 3>bot_id =". . . . "</font> &ensp; 机器人id &ensp; 在coze智能体详细界面/bot/ . . . . . （. . .）就是bot_id
3.  import requests
<h2>具体使用</h2>

1.  <font size = 3>请求头(headers)</font>
>'Authorization': f'Beare{personal_access_token} &ensp; &ensp; 用于验证个人访问令牌</br>
>'Content-Type': 'application/json' &ensp; &ensp; 请求ai的输出格式coze开发指南推荐json形式输出</br>
>'Accept': '* / *' &ensp; 默认写这个接受所有媒体类型</br>
>'Connection': 'keep-alive'&ensp; 持续联网</br>

2.  <font size = 3>请求体(payload)</font>
> 'bot_id': {bot_id} &ensp; &ensp; 要进行对话的机器人ID</br>
>  'user': {. . .} &ensp; &ensp; 用于标识当前与Bot交互的用户 可以设置不同的身份进行不同情况的调用
> 'query': {. . .} &ensp; &ensp; 发给bot的消息
> 'stream': {布尔值} &ensp; 设置流式输出
> 'conversation_id' &ensp; 对话id 一般之后赋值

3. <font size = 3>调用接口</font>
>response = requests.post(</br> &ensp;'https://api.coze.cn/open_api/v2/chat', </br> &ensp;headers=headers,</br>&ensp; json=payload,</br> &ensp; timeout=30 )

<h2>细节处理，逻辑处理</h2>

<font size ="4">1.  会话ID的获取便于持续对话</font></br>
   首先分析main()函数： 定义一个conversation_id变量获取对话id初始为空 </br>
   从 print("AI: ",end="",flush=True) 开始</br>

   ```py
    answer, conv_id = call_coze_api(user_input, conversation_id)
        
        if conv_id:
            conversation_id = conv_id
        
        print(answer)
   ```
给 answer和conv_id赋值 第一次执行时Coze会自动生成一个对话ID 循环一次后 会获取id返回给conv_id 之后main函数调用 call_coze_api 时会进入if判断若存在对话ID会赋值给请求体继续对话 

<font size ="4">2.  response.json()所需信息截取</font></br>
   在请求头中我们定义了以json形式输出但是会有许多非必要信息的输出在代码里具体截取如下

   ```py
    if response.ok:
            data = response.json()
            if data.get("code") == 0:
                messages = data.get("messages", [])
                answer = ""
                for msg in messages:
                    if msg.get("type") == "answer" and msg.get("content_type") == "text":
                        answer = msg.get("content", "")
                return answer, data.get("conversation_id")
            else:
                return f"API错误: {data.get('msg', '未知错误')}", None
        else:
            return f"请求失败 ({response.status_code})", None
   ```

先判断API的请求是否成立 即' if response.ok: '否则返回请求失败 对话ID返回空 </br>
在Coze API规范里 code == 0 表示API调用成功</br>
提取被解析成python字典的json数组所需回复消息在"message"的'content'中读取其中的纯text内容即可