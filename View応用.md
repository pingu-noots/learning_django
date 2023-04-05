# リダイレクトの実行
## 管理画面の追加
### admin.py に管理者が管理したいモデルを追加する
```py
from .models import モデル名
admin.site.register(モデル名)
```
### 管理者を作成する
```
python manage.pcreatesuperuser
```
### 管理者としてページに入る
http://127.0.0.1:8000/admin/

## 次の画面に値を渡す
### urls.py のpath()の第一引数のurlの後に <型:変数名> で入れる
```py
    path('url_sample<int:id>', views.url_sample, name='url_sample'),
```
### templateで画面遷移の際にurlsに値を入れる
url記述する際に フォルダ名:HTML名　変数=値 を入れる
```html
<a href="{% url 'store:item/detail' id=item.id %}">{{item.name}}</a>
```

## リダイレクトでページ遷移する
redirect('フォルダ名:htmlファイル', 渡したい変数など)
```py
# redirect のimport
from django.shortcuts import render, redirect
def one_item(request):
    return redirect('store:item_detail', id=1)
```
ユーザーから意図しないrenderをされた時や、完全別ページに遷移したい時などに、内部でredirectとして画面遷移先を決定する使い方

# エラーハンドリング
## エラーハンドリングを有効に設定する
setting.pyを編集
```py
DEBUG = False
ALLOW_HOSTS = ['127.0.0.1']
```
## エラーの際に表示したいhtmlを返す関数をviews.pyにつくる
```py
# 第二引数に exception が必要
def page_not_found(request, exception):
    # 第三引数に status=エラーコード が必要
    return render(request, 'store/404.html', status=404)
```
## プロジェクトのurls.pyに設定したいステータスのハンドラーを追加する
```py
# handler404などは固定変数名
handler404 = views.404.htmlを返す関数(404エラーの場合に実行したい関数(引数は2つ))
handler500 = views.関数名(500エラーの場合に実行したい関数(引数は1つ))
```
## 意図的に404エラーを発生させることも可能
```py
from django.http import Http404
raise Http404
```
## モデルから値を取得するときに404エラーを発生させる方法
```py
# 指定したモデルを呼び出し、getを行う。値を取得できなかった場合、raise Http404とする
get_object_or_404(モデル名, get条件)
# 指定したモデルを呼び出し、filterを行う。値を取得できなかった場合、raise Http404とする
get_list_or_404(モデル名, filter条件)
```

# ログイン機能の実装
## settings.py のパスワードの設定をいじる
```py
# AUTH_PASSWORD_VALIDATORS の中の物を適宜変更する
'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS' : {
            'min_length': 8
        }

# PASSWORD_HASHERS を宣言してハッシュ化アルゴリズムを指定する
# 複数指定でき、上から順に使われる
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
]

# ログインに利用するURLを指定する
LOGIN_URL = 'hoge'
```

## ログイン用Userクラスの作成
**django.contrib.auth.models.User** でログイン用のユーザとして利用する (管理画面のログインにも用いられるが、これを用いてwebサイトのログインも行う)  
デフォルトのフィールドにフィールドを追加する場合、OneToOneFieldで紐付けたモデルを作成する
```py
from django.contrib.auth.models import User

class UserProfile(models.Model):
    # ここでの第一引数のUserは django.contrib.auth.models.Userである
    user = models.OneToOneField(User, on_delete=models.CASCADE)
```

## ビューで利用する処理
```py
from django.contrib.auth import authenticate, login, logout

# 引数に名前とパスワードを取り、ユーザが存在してパスワードが正しいかチェックする
user = authenticate(username, password)
# ユーザーがアクティブかをbooleanで返す
user.is_acitive()

# ログインを行う
login(request, user)

# ログアウトを行う
logout()

# ログインが必要な関数にデコレータとして付与すると、その関数はログインしていない場合にはエラーとすることができる
from django.contrib.auth.decorators import login_required
@login_required
def user_logout(request):
    logout(request)
    return redirect('user:index')
```

## テンプレートで利用する処理
```html
{% if user.is_authenticated %}
    <!-- ログインしている時だけ実行される -->
{% endif %}
```

## パスワードの妥当性判断
```py
# パスワードの変更
django.contrib.auth.password_validation.password_changed
# ユーザのパスワードが適切か(短すぎないか、ありきたり過ぎないか、ユーザ名と類似していないか)
validate_password(password, user)

# 適切でない場合はvalidation error
try:
    validate_password(user_form.cleaned_data.get('password', user))
except ValidationError as e:
    return render(request, 'template.html')
```

## 独自のpassword validatorの定義
適当な場所にファイルを作ってsettings.pyのAUTH_PASSWORD_VALIDATORSに突っ込む
```py
from django.core.exceptions import ValidationError
import re

class CustomPasswordValidator():
    def __init__(self):
        pass
    def validate(self, password, user=None):
        if all((re.search('[0-9]', password), re.search('[a-z]', password), re.search('[A-Z]', password))):
            return
        raise ValidationError('Need 0-9, a-z, A-Z')
    def get_help_text(self):
        return 'Need 0-9, a-z, A-Z'
```
```py
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'utils.validators.CustomPasswordValidator'
    }
]
```

# 実践的なユーザーのクラスの作り方
ユーザーのクラスを利用する場合には、元のユーザーをカスタマイズして用いる  
この際に、AbstractUser か AbstractBaseUser を用いる
**AbstractUser**: すでに存在するフィールドをそのまま流用してusernamaフィールドを削除したい場合に用いる(あまり実践的ではない)  
**AbstractBaseUser**: 初めからUserを作りたい  
以下の手順で作成する
## カスタムマネージャーとカスタムユーザーのクラスを作成する
カスタムマネージャーは django.contrib.auth.models.BaseUserManager を継承して作成し、**create_user()** と **create_superuser()** メソッドを追加して、ユーザー作成時、スーパーユーザー作成時の処理を追加する。  
```py
class UserManager(BaseUserManager):

    def create_user(self, username, email, password=None):
        if not email:
            raise ValueError('Enter Email!')
        user = self.model(
            username=username,
            email=email
        )
        user.set_password(password)
        user.save(using=self._db)
        return user
    
    def create_superuser(self, username, email, password=None):
        user = self.model(
            username=username,
            email=email,
        )
        user.set_password(password)
        user.is_staff = True
        user.is_active = True
        user.is_superuser = True
        user.save(using=self._db)
        return user
```
カスタムユーザーは django.contrib.auth.models.AbstractBaseUser を継承して作成し、必要なフィールド情報を記入する  
カスタムしたユーザーにsuperuserなどのDjangoのパーミッションを取り入れるには PermissionsMixin を用いる
```py
class User(AbstractBaseUser, PermissionsMixin):
    username = models.CharField(max_length=150)
    email = models.EmailField(max_length=255, unique=True)
    is_active = models.BooleanField(default=False)
    is_staff = models.BooleanField(default=False)
    website = models.URLField(null=True)
    picture = models.FileField(null=True)
    # ユーザーを一意に識別するためのもの 今回はemail
    USERNAME_FIELD = 'email'
    # スーパーユーザ作成時に必要なもの
    REQUIRED_FIELDS = ['username']
    # パスワードは AbstractBaseUser にあるのでここで定義する必要はない
    # PermissionsMixin にスーパーユーザかを識別する is_superuser が入っているのでここで定義する必要はない

    # modelを関連付け
    # UserManager 側でself.modelとすると関連付けられたmodelにアクセスできる
    objects = UserManager()
```
## settings.pyを修正してユーザーはカスタムしたクラスを指すようにする
```py
# 'アプリ名.関数名'
AUTH_USER_MODEL = 'users.CustomUser'
```
## マイグレーションを行う
## フォームやAdminを作成する
フォーム
```py
class UserCreationForm(forms.ModelForm):
    password = forms.CharField(label='password', widget=forms.PasswordInput)
    confirm_password = forms.CharField(label='Reenter password', widget=forms.PasswordInput)
    
    class Meta:
        model = User
        fields = ('username', 'email', 'password')

    def clean(self):
        cleaned_data = super().clean()
        password = cleaned_data.get('password')
        confirm_password = cleaned_data.get('confirm_password')
        if password != confirm_password:
            raise ValidationError('Not match password')
        
    def save(self):
        user = super().save(commit=False)
        user.set_password(self.cleaned_data.get('password'))
        user.save()
        return user
```
Admin
```py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from django.contrib.auth import get_user_model
from .forms import UserChangeForm, UserCreationForm

User = get_user_model()

class CustomizeUserAdmin(UserAdmin):
    form = UserChangeForm # ユーザ編集画面でつかうForm
    add_form = UserCreationForm # ユーザ作成画面
    # 一覧画面で表示する
    list_display = ('username', 'email', 'is_staff')
    # ユーザ編集画面で表示する要素
    fieldsets = (
        ('ユーザ情報', {'fields': ('username', 'email', 'password', 'website', 'picture')}),
        ('パーミッション', {'fields': ('is_staff', 'is_active', 'is_superuser')}),
    )
    add_fieldsets = (
        ('ユーザ情報', {
            'fields': ('username', 'email', 'password', 'confirm_password')
        }),
    )

admin.site.register(User, CustomizeUserAdmin)
```