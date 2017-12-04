---
layout: post
title:  "利用 Facebook Graph API 製作聊天機器人 Chat Bot"
date:   2017-12-04 17:24:00 +0800
---

# 設定 Facebook Developer 開發人員帳號

### 我假設你已經有一般 Facebook 帳號且已登入

### 按照流程填寫資訊

![FB Dev Homepage](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/1.png)

![FB Dev Register](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/2.png)

### 完成後申請後，點右上角 New Apps 再點 Add a New App

![FB Dev Homepage](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/3.png)

### Display Name 是 App 的名稱，Email 是 FB 聯絡時用的，請填你收得到信的信箱

![FB Dev Homepage](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/4.png)

### 到這個頁面後點 Messenger 下方的 Set Up

![FB Dev Homepage](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/6.png)

### Facebook 的 Messenger Chatbot 是跟專頁綁定的，所以如果還沒有專頁就創一個

![FB Dev Homepage](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/7.png)

### 它會要求許可，因為你正在做的 App 未來會代替你幫專頁回信

![FB Dev Homepage](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/8.png)

### 完成後會產生一個 Page Access Token，這個是 FB 用來辨別是你程式傳的代號

![FB Dev Homepage](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/9.png)

### 按下方的 Setup Webhook 會出現很多選項，如果只要傳訊息和用按紐，勾 messages, messages_postbacks，最下面那個是如果被警告時會傳東西給你的程式

### Callback URL 就是你程式會接收訊息的網址，Verify Token 是點下下面 Verify and Save 時 Facebook 傳一則 GET 訊息給 Callback URL 你程式應該要回的

```python

VERIFY_TOKEN = "就是 Setup Webhook 那裡你自己填的東西"
PAT = "就是 Page Access Token 它給你的一串"

app = Flask(__name__)

@app.route('/', methods=['GET'])
def verify():
    # Once the endpoint is added as a webhook, it must return back
    # the 'hub.challenge' value it receives in the request arguments
    if (request.args.get('hub.verify_token', '') == VERIFY_TOKEN):
        print("Verified")
        return request.args.get('hub.challenge', '')
    else:
        print('wrong verification token')
        return "Error, Verification Failed"
        
if __name__ == '__main__':
    app.run()
```

### 目前因為你的聊天機器人在開發模式中，所以除了你自己沒辦法用，如果你要讓別人也收得到機器人的訊息要到 Roles 底下 Roles 中的 Test User 把其他人的帳號加進去才行

![FB Dev Homepage](https://raw.githubusercontent.com/ouvek-kostiva/ouvek-kostiva.github.io/master/assets/img/post/facebook-bot-api-img/11.png)

### 下面部分是你程式接收使用者透過 FB API 傳訊息給你，也放在同一個檔案

# 注意：FB 超過 20 秒沒從你程式收到 OK 200 就會延遲一下再重送，所以一定要盡快送出 OK 200

```python
# FB API 用 POST 收送訊息
@app.route('/', methods=['POST'])
@app.route('/webhook', methods=['POST'])
def handle_messages():
    data = request.get_json()
    entry = data['entry'][0]
    if entry.get("messaging"):
        messaging_event = entry['messaging'][0]
        sender_id = messaging_event['sender']['id'] # 使用者的 ID
        timestamp = entry['time'] # 時間
        if messaging_event.get("message"):
            if messaging_event['message'].get('text'):
                text = messaging_event['message']['text'] #使用者傳的訊息
                insret = insertRec() ### 我在收到訊息後立刻將訊息先存到資料庫
                return 'ok', 200
            if messaging_event['message'].get('sticker_id'):
                if str(messaging_event['message']['sticker_id']) == str(369239263222822): #小讚的按鈕
                    insret = insertRec() ### 我在收到訊息後立刻將訊息先存到資料庫
                    return 'ok', 200
                elif str(messaging_event['message']['sticker_id']) == str(369239343222814): #中讚的按鈕
                    insret = insertRec() ### 我在收到訊息後立刻將訊息先存到資料庫
                    return 'ok', 200
                elif str(messaging_event['message']['sticker_id']) == str(369239383222810): #大讚的按鈕
                    insret = insertRec() ### 我在收到訊息後立刻將訊息先存到資料庫
                    return 'ok', 200
                return 'ok', 200
            return 'ok', 200
        elif messaging_event.get("postback"):
            if messaging_event['postback'].get('payload'):
                payload = messaging_event['postback']['payload'] #你自己設定的按紐的訊息
                insret = insertRec() ### 我在收到訊息後立刻將訊息先存到資料庫
                return 'ok', 200
            return 'ok', 200
        else:
            print(messaging_event)
            return 'ok', 200
    return 'ok', 200
```

### 這部分是接收訊息的程式，下次再發一篇回傳訊息的部分

### 設定使用者第一次點傳送訊息給專頁的歡迎文字（這個在 Linux 上直接貼到 Bash 執行）

```shell
curl -X POST -H "Content-Type: application/json" -d '{
  "setting_type":"greeting",
  "greeting":{
    "text":"歡迎文字"
  }
}' "https://graph.facebook.com/v2.6/me/thread_settings?access_token=這邊要填你的PageAccessToken"

```

### 設定下方會一直顯示的常用按紐

```shell
curl -X POST -H "Content-Type: application/json" -d '{
"persistent_menu":[
    {
    "locale":"default",
    "composer_input_disabled":false,
    "call_to_actions":[
        {
        "title":"Sublist Button Text",
        "type":"nested",
        "call_to_actions":[
            {
            "title":"Button 1 Title",
            "type":"postback",
            "payload":"Payload Reply"
            },
            {
            "title":"Button 2 Title",
            "type":"postback",
            "payload":"Payload Reply"
            },
            {
            "title":"Button 3 Title",
            "type":"postback",
            "payload":"Payload Reply"
            },
            {
            "title":"Button 4 Title",
            "type":"postback",
            "payload":"Payload Reply"
            },
            {
            "title":"Button 5 Title",
            "type":"postback",
            "payload":"Payload Reply"
            }
        ]
        },
        {
        "type":"web_url",
        "title":"Website Button Text",
        "url":"https://www.example.com",
        "webview_height_ratio":"full"
        }
    ]
    },
    {
    "locale":"zh_TW",
    "composer_input_disabled":false,
    "call_to_actions":[
        {
        "title":"更多按鈕",
        "type":"nested",
        "call_to_actions":[
            {
            "title":"按鈕文字",
            "type":"postback",
            "payload":"實際收到的回應"
            },
            {
            "title":"按鈕文字",
            "type":"postback",
            "payload":"實際收到的回應"
            },
            {
            "title":"按鈕文字",
            "type":"postback",
            "payload":"實際收到的回應"
            },
            {
            "title":"按鈕文字",
            "type":"postback",
            "payload":"實際收到的回應"
            },
            {
            "title":"按鈕文字",
            "type":"postback",
            "payload":"實際收到的回應"
            }
        ]
        },
        {
        "type":"web_url",
        "title":"網站按鈕文字",
        "url":"https://www.example.com",
        "webview_height_ratio":"full"
        }
    ]
    }
]
}' "https://graph.facebook.com/v2.6/me/messenger_profile?access_token=這邊要填你的PageAccessToken"
```
