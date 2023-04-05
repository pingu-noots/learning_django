# view の仕組み

## ルーティング

Project_name/urls.py がプロジェクト全体のルーティングを担っている  
アプリを作成してアプリごとのルーティングをしたい場合には

1. App_name/views.py を編集  
   関数名は後に使う  
   request はそういうものやと思って  
   render()の第二引数にルーティングしたい url を書く

```py
def index(request):
    return render(request, 'TemplateApp/index.html')
```

2. App_name/urls.py を編集  
   app_name はアプリ名 予約語  
   urlpatterns に path 関数を使ってファイルと views.py のルーティングをつなげる  
   path()  
    第一引数: html 上でのルーティング  
    第二引数: views.py にあるどの関数を使うか  
    第三引数: name="URL 名称"

```py
from django.urls import path
from . import views

app_name = 'template_app'
urlpatterns = [
    path('', views.index, name='index')
]
```

3. Project_name/urls.py を編集

```py
urlpatterns = [
    # hogehoge,
    path('ルーティング名/', include('App_name.urls'))
]
```

## View から template に値を渡す

App_name/views.py を編集

```py
# render()の第三匹数で context={}で値を渡す
# {'変数名': '内容'}
render(request, 'TemplateApp/index.html' context={'value': 'HELLO'})
```

template の html ファイルでの記述  
この書き方を DTL という
DTL では if や for といった処理を書ける

```html
<h1>{{value}}</h1>
<ul>
  {% for fruit in favorite_fruits %}
  <li>{{fruit}}</li>
  {%endfor%}
</ul>

{%if 'Grape' in favorite_fruits%}
<b>Grapeはあります</b>
{%endif%}
```

## Templates の継承

ヘッダーなど共通の処理を書きたい場合は TDL で templates/の直下に html ファイルを作る
html ファイルにて以下のように記述する

```html
<!DOCTYPE html>
<html>
    <head>
        <mata charset='utf-8'>
        <title>{% block title %} My Page {% endblock %}</title>
    </head>
    <body>
        <h1>My Page</h1>
        {% block content %}{% endblock %}
    </body>
</html>
```

継承先の html ファイルでは頭に以下のように書く

```html
{% extends "base.html" %}
```

変数を使用したい場合は以下のように書く

```
{% block title %} Sample2 {{block.super}}{% endblock %}
{% block content %} fugafuga {% endblock %}
```

## DTL での filter 関数

templates の中の html ファイルにて変数を filter 関数を使って修飾することができる  
書き方は {% 変数|関数 %}  
関数にはいろいろ種類がある  
例)  
 大文字化: upper  
 リンクをクリック可能に: urlize  
 デフォルト値を設定: default:デフォルト値  
関数は | をつなげることによって複数つなげることができる

## 独自 filter 関数の定義

1. アプリケーションフォルダ内に templatetags フォルダを作成
2. templatetags の中に**\_\_init\_\_**.py を作成
3. filter を自作するファイルを作成し、中に関数を記述して、register を行う

   ```py
   from django import template
   register = template.Library()

   @register.filter(name='my_filter')
   def hoge (value):
    # 処理を書く
   ```

4. template で load して利用する
   ```html
   {% load filterのファイル名 %} {{ value|my_filter }}
   ```

# Template での画面遷移方法

{% url %}を用いて、urls.py で指定した遷移先に移動する  
urls.py

```py
app_name = hoge
urlpatterns = [
    path('home', views.home, name='home'),
]
```

Template での記述

```html
<a href="{% 'hoge:home' %}"> HOGE </a>
<!-- "{% url 'app_name : pathのname' %}" -->
```

## View に値を渡す場合

template に引数を渡す

```html
<a href="{% 'hoge:home' val1='val1', val2='val2' %}"> HOGE </a>
```

urls.py で/<>で受け取る

```py
app_name = hoge
urlpatterns = [
    path('home/<val1>/<val2>', views.home, name='home'),
]
```

views.py の遷移先で受け取る  
受け取る時の引数は urls.py の引数と同名である必要がある

```py
def home(request, val1, val2):
    value = f'{val1} {val2}'
```

## Template での static の利用

static という、css, js, 画像などの静的コンテンツを入れておくためのディレクトリ  
まずは setting.py に以下を追記する

```py
STATICFILES_DIRS = [
    'staticフォルダのパス'
]
# 識別子を使って複数のstaticフォルダを使いたい場合には
STATICFILES_DIRS = [
    ('フォルダの識別子,' 'staticフォルダのパス')
]
```

html ファイルで利用する

```html
{% load static %} <img src="{% static '識別子/example.jpg' %}" />
<!-- 画像の読み込み -->
<link rel="stylesheet" type="text/css" href="{% static '識別子/style.css' %}" />
<!-- css -->
```
