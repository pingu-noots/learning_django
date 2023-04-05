# シグナル
ある特定の処理を実行した際に、自動で呼び出される処理を定義したい場合に用いる  
特にモデルでよく実装される  
post_save で特定のモデルのデータを保存した後に実行する関数を定義する
```py
from django.contrib.auth.models import User
from django.db.models.signals import post_save

# instance=モデルで指定したクラスのオブジェクトが作成された場合の、そのレコードのインスタンス
def save_profile(sender, instance, **kwargs):
    instance.profile.save()

# post_save.connect(関数名、sender=モデル名)
post_save.connect(save_profile, sender=User)
```
このようにすると、Userクラスで新たにオブジェクトが追加される度に save_profile が呼び出される　　
シグナルにはpost_save以外にもある
```py
# 保存処理の前に実行
pre_save
# 削除処理の前に実行
pre_delete
# 削除処理の後に実行
post_delete
```
## @receiver
特定のモデルのデータを保存した後に実行する関数を指定する方法はもう一つあり、それが @receiver
```py
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

# @receiver(関数名、sender=モデル名)
@receiver(post_save, sender=User)
def save_profile(sender, instance, **kwargs):
    instance.profile.save()
```
同様にUserクラスで新たにオブジェクトが追加されるたびに save_profile が実行される

# Modelマネージャー
モデルを用いる時には、**テーブルの定義を記述するクラス**と、**テーブルのデータ挿入と取り出しをするクラス**とを分けることがある  
テーブルのデータ挿入と取り出しをするクラスは **models.Manager** を継承して作成する
```py
# Managerの作成
class UserManager(models.Manager):
    def counts(self):
        pass

class User(models.Model):
    name = models.CharField(max_length=200)

    # Managerの指定
    objects = UserManager()
```
UserManagerのcounts()の呼び出しは User.objects.count()

# クエリ付きのページ遷移
urls.py にて以下のように設定した時
```py
urlpatterns = [
    path('activate_user/<uuid:token>', views.activate_user, name='activate_user')
]
```
path()の第一引数にURLを記入し、**<>でクエリを記入する**  
**<型:引数>**の形である  
<>がなんだろうと、前の activate_user にviews処理が行く  
views.pyのactivate_user()では<>で渡されたクエリを**引数として**受け取る  
```py
def activate_user(request, token):
    pass
```