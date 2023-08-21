Linebot + Django 專案實作
===

- targets: Linebot + Django framework, chatbot implementation.

Django 介紹
---

### 架構

Django 是一個基於 Python 的網頁框架，其架構不同於三大前端框架

- `Django`: Model-Template-Views 架構
- `React.js` / `Angular.js` / `Vue.js`: Model-View-Contorller 架構

| MTV | MVC | 
| -------- | -------- |
| Model | Model	|
| Template | View |
| Views | Controller |

![](https://i.imgur.com/tx0zsYj.png)

- Model: 取用後端資料 
- Template: 原始 html
- Views: 顯示出來的網頁內容

### 完整流程

![](https://i.imgur.com/E1bQ5Sp.png)

實作專案
---

### 建立虛擬環境

```bash=
pip install virtualenv

# 建立虛擬環境
virtualenv 'linebot-env'

# 啟動虛擬環境
.\linebot-env\Scripts\activate

# 退出虛擬環境
deactivate
```

若發生權限問題

![](https://i.imgur.com/7PqZUGN.png)

[解決方法](https://israynotarray.com/other/20200510/1067127387/ " UnauthorizedAccess")

```bash=
# Windows PowerShell
Set-ExecutionPolicy RemoteSigned
```

安裝相關套件
---

```bash=
pip install Django
pip install line-bot-sdk
```

建立專案
---

```bash=
cd D:/
django-admin startproject '專案名稱'
cd '專案名稱'
```

### 建立 APP

可以進行多個 Line Bot 開發

```bash=
python manage.py startapp 'APP名稱'
```

Django 基礎設定 Setting.py
---

1. 新增兩個資料夾

    ```bash=
    md static
    md templates
    ```

- static 用於靜態資料如圖片、檔案
- templates 用於放寫好的 html

2. 設定 Channel Access Token & Channel Secret

    ![](https://i.imgur.com/3svMhdL.png)

3. 於 INSTALLED_APPS 中設定建立的 APP 名稱

    ![](https://i.imgur.com/ijFM6yl.png)

4. 於 TEMPLATES 中設定 templates 的資料夾路徑

    ![](https://i.imgur.com/XWdj2iA.png)

5. 語系、時區的設定

    ![](https://i.imgur.com/pT3WV8d.png)

6. 新增 static 路徑

    ![](https://i.imgur.com/9MdavbV.png)

7. 開發階段設定

    ![](https://i.imgur.com/rJ3R5PW.png)

8. 資料庫遷移的初始化

    ```bash=
    python manage.py makemigrations
    python manage.py migrate
    ```

9. 建立管理者帳號

    ```bash=
    python manage.py createsuperuser
    ```

    ![](https://i.imgur.com/dUfn81s.png)

10. 執行 runserver 進行測試

    ```bash=
    python manage.py runserver
    # 開啟瀏覽器進入 127.0.0.1:8000/admin
    ```

    ![](https://i.imgur.com/YuLOVYZ.png)

- 登入畫面

    ![](https://i.imgur.com/NZEEbiN.png)

### 設定 urls.py

```python=
from django.contrib import admin
from django.urls import path
from example import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('callback', views.callback),
]
```

### 設定 views.py

```python=
from django.shortcuts import render

# Create your views here.
from django.conf import settings
from django.http import HttpResponse, HttpResponseBadRequest, HttpResponseForbidden
from django.views.decorators.csrf import csrf_exempt

from linebot import LineBotApi, WebhookParser
from linebot.exceptions import InvalidSignatureError, LineBotApiError
from linebot.models import MessageEvent, TextSendMessage

line_bot_api = LineBotApi(settings.LINE_CHANNEL_ACCESS_TOKEN)
parser = WebhookParser(settings.LINE_CHANNEL_SECRET)


@csrf_exempt
def callback(request):
    if request.method == 'POST':
        signature = request.META['HTTP_X_LINE_SIGNATURE']
        body = request.body.decode('utf-8')

        try:
            events = parser.parse(body, signature)
        except InvalidSignatureError:
            return HttpResponseForbidden()
        except LineBotApiError:
            return HttpResponseBadRequest()

        for event in events:
            if isinstance(event, MessageEvent):
                mtext=event.message.text
                message=[]
                message.append(TextSendMessage(text=mtext))
                line_bot_api.reply_message(event.reply_token,message)

        return HttpResponse()
    else:
        return HttpResponseBadRequest()
```

### 說明

- LINE server 收到訊息後主動傳送 LINE BOT，因此需要告訴 server，當收到訊息時的傳送位置，此為 webhook URL
- 當 callback 函數被呼叫時，是收到由 LINE server 發過來 LINE BOT 的 webhook

ngrok 設定
---

https://ngrok.com/download

### 開啟 Django 伺服器 + 啟動 ngrok

```bash=
    python manage.py runserver
    ./ngrok http 8000
```

### LINE Developer 後台貼上 webhook URL
```bash=
    #webhook URL範例
    https://yourdomain_name/callback
```

完成!
---

![](https://i.imgur.com/oUbUsZk.png)