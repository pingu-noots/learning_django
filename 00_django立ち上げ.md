# django 立ち上げメモ

## conda で環境を作る

```
$ conda create -n <env_name> python=x.xx
$ conda activate env_name
$ conda install django
```

## django でプロジェクトを作る

```
$ django-admin startproject <Project_name>
```

## django でアプリを作る

```
$ python manage.py startapp <App_name>
```

App_name/ に urls.py を置き、編集する

```python
from django.urls import path
from . import views

app_name = 'template_app'
urlpatterns = [
    path('', views.index, name='index')
]
```

Project_name/setting.py の INSTALLED_APPS にアプリを追加

```python
INSTALLED_APPS = [
    # hogehoge,
    # fugafuga,
    'App_name',
]
```

App_name/ に templates フォルダを作成

## マイグレーションを適用する

```
python namage.py migrate
```

## サーバーを建てる

```
python manage.py runserver
```

## templates の場所を変更する
アプリごとにtemplateを持たせたくない時の設定  
Project_name と同じ階層に templates フォルダを作成  
setting.py を編集

```py
import os
TEMPLATE_DIR = os.path.join(BASE_DIR, "templates")
TEMPLATES = [
    {
        #hogehoge,
        'DIRS': [TEMPLATE_DIR,],
    }
]
```

**templates を App_name から出すことによってアプリを複数開発するプロジェクトでも対応できる。**
