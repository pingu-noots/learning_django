# Model

## 基本的な流れ
1. setting.pyのDATABASESにDB接続方法を記述する
2. models.pyにdjango.db.models.Modelを継承したクラスを作成しテーブルを定義する
3. Modelの変更内容をファイルに出力する
   ```
   python manage.py makemigrations (アプリ名)(--name マイグレーションの名前)
4. マイグレーションをしてテーブル定義の変更をDBに反映させる
   ```
   python manage.py migrate
   ```

## 管理画面の利用
Modelに格納されたデータを確認したり、データを挿入したりする画面  
djangoではデフォルトの管理画面を利用できる  
<br>
管理画面のパスの設定
```py
urlpatterns = [
    path('admin', admin.site.urls)
]
```
管理画面ログイン用のスーパーユーザの作成
```
python manage.py createsuperuser
```
管理画面で扱うModelの追加(admin.py)
```py
admin.site.register(Model)
```

## テーブルの設定方法手順
1. マイグレーション名を確認
```
python manage.py showmigrations <アプリ名>
```
2. マイグレーションを設定変更前のものに戻す
```
python manage.py migrate <アプリ名>　<マイグレーション名>
```
3. マイグレーションファイルを削除する
<アプリ名>/migrations/<マイグレーション名>.py を削除
migrateコマンドは<アプリ名>/migrations/<マイグレーション名>.pyを見るので設定変更するにはここを削除して新しく同じファイルを作る必要がある
4. マイグレーションファイルを生成する
設定編集後の<アプリ名>/migrations/<マイグレーション名>.pyを生成する
```
python manage.py makemigrations <アプリ名> --name <マイグレーション名>
```
5. マイグレーションファイルを適用させる
設定編集後の<アプリ名>/migrations/<マイグレーション名>.pyを適用させる
```
python manage.py migrate ModelApp
```

## ModelのMetaオプション
### Modelの継承
自前のクラスを定義して継承させることもできる(抽象クラスの作成)  
その場合、クラスの中にMetaクラスを作成し、abstract = Trueを記入する
```py
class BaseMeta(models.Model):
    create_at = models.DateTimeField(default=timezone.datetime.now)
    update_at = models.DateTimeField(default=timezone.datetime.now)
    class Meta:
        abstract = True
```
利用する際は抽象クラスを親クラスに指定するだけ
```py
class Person(BaseMeta):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    salary = models.FloatField(null=True)
```
### Metaオプションの利用
Metaオプションを利用することによって機能を追加することができる  
その場合、クラスの中にMetaクラスを作成する  
Modelの継承のabstractも実はMetaオプションの利用
```py
class Person(BaseMeta):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    salary = models.FloatField(null=True)
    class Meta:
        # db_tableでテーブル名の指定
        db_table = 'person'
        # index_togetherで複合インデックスの指定
        index_together = [['first_name', 'last_name']]
        # orderingで取り出す時のデフォルトの順序の指定
        ordering = ['salary']
```

## レコードの追加(INSERT)
方法は3つある
1. インスタンスのsaveメソッドの実行  
   DBに同じものが存在しない場合はINSERT、存在する場合はUPDATE
    ```py
    web_site = WebSite(url='www.sample.com', name='sample')
    web_site.save()
    ```
2. クラスメソッドのcreateメソッドの実行  
   DBに同じものが存在しない場合はDBへ挿入、存在する場合はIntegrityError
   ```py
   WebSite.objects.create(url='www.sample.com', name='sample')
   ```
3. get_or_createメソッドの実行
   同じ値のものが存在する場合はそのものを返し、存在しない場合は作成して返す
   ```py
   obj, create = Person.objects.get_or_create(first_name='Jiro', last_name='Sato')

## レコードの取得(SELECT)
1. getメソッドで1件だけレコードデータを取得  
   主キーやUniqueカラムなど1件だけを絞り込めることがわかるカラムでの絞り込みで用いる
   ```py
   # 主キーで絞り込み
   entry = Entry.objects.get(pk=1)
   # first_nameによる絞り込み
   # 取得できなかった場合Model.DoesNotExitエラー
   # 複数取得できた場合はModel.MultipuleObjectsReturnedエラー
   person = Person.objects.get(first_name='Taro')
   # 複数指定もできる
   p = Person.objects.get(first_name='Taro', last_name='Sato')
   ```
2. 全行取得  
   ```py
   persons = Person.objects.all()
   ```
3. filterで絞り込んで取得  
   ```py
   # Personクラスからfirst_nameがTaroのもののみ所得
   # 取得できない場合でもエラーにはならない
   p = Person.objects.filter(first_name='Taro').all()
   ```

## Modelからレコードの更新
1. Modelインスタンスのsaveメソッドによる更新(丁寧に更新する)  
   ```py
   person = Person.objects.get(first_name='taro')
   person.first_name = 'Jiro'
   person.save()
   ```
2. updateメソッドを使用して更新(一度に更新する)  
   ```py
   # id=4のレコードに対してevent_dateを更新
   Event.objects.fiter(id=4).update(event_data=event_date)
   ```

## Modelからレコードを削除
1. filterでレコードを絞り込んでdeleteメソッド実行  
   ```py
   Person.objects.filter(first_name='taro').delete()
   ```
2. allで全権取得してdeleteメソッドを実行し、レコードを全て削除  
   ```py
   Person.objects.all().delete()
   ```

## 外部キー
### 1対多の紐付け
models.ForeignKeyで紐付ける  
第一引数：外部キーとして参照先したいモデル名  
第二引数：on_deleteで外部キー元が削除されたときの挙動を決める  
```py
class Car(models.Model):
    manufacturer = models.ForeignKey(
        Manufacturer,
        on_delete = models.CASCADE,
    )
class Manufacturer(models.Model):
    pass
```
#### on_deleteの指定一覧
- models.CASCADE  
ForeignKey を含むオブジェクトも削除する  
- models.PROTECT  
    ProtectedErrorを発生させ、照されたオブジェクトの削除を防ぐ  
    削除するには参照元の対象レコードを全て削除する必要がある  
- models.RESTRICT  
    RestrictedErrorを発生させ、参照されたオブジェクトの削除を防ぐ  
    ただし、同じ操作で参照先の参照先が削除されるときに、それがCASCADEで紐づけられていた場合は、参照されたオブジェクトの削除が許可される  
- models.SET_NULL  
    参照先が削除された場合、NULLが格納される  
    使用するためにはForeignKey()の第三引数にnull=Trueが必要  
- models.DEFAULT  
    参照先が削除された場合、デフォルト値が格納される  
- models.SET()  
    参照先が削除された場合、SETで指定した値が格納される  

### 1対1の紐付け
基本的には1対1はテーブル構造として使わないと思うが、一応  
OneToOneFieldを使用する  
```py
class Base_emplyee(models.Model):
    Birth_employee = models.OneToOneField(
        Birth_employee,
        on_delete=models.CASCADE,
        primary_key=True
    )
class Birth_employee(models.Model):
    pass
```
object.save()としたときに、objectの主キーが同じである場合はUPDATEになってしまうため値が書き変わってしまう。注意！  
- アクセス方法  
  1:多 の場合の 1 の方ではインスタンス名.モデル名_setで取ってこれる  
  多 の場合はインスタンス名.モデル名で取ってこれる  
### 多対多の紐付け
1. ManyToManyFieldで紐づける  
```py
class Teachers(models.Model):
    pass

class Subjects(models.Model):
    teachers = models.ManyToManyField(
        Teachers
    )
```
2. 中間テーブルが自動でできる  
名前は<ManyToManyFieldをしたテーブル名>_<ManyToManyFieldで指定されたテーブル名>
3. 中間テーブルにデータを入れる  
MTMFしたテーブルのオブジェクト.MTMFした時の関数名.add(MTMFされたオブジェクト1, MTMFされたオブジェクト2...)  
4. アクセス  
- ManyToManyFieldがない側(Teachers)  
  インスタンス名.モデル名_set.all()でアクセスできる
  ```py
  teacher = Teachers.objects.get(pk=1)
  teacher.subject_set.all()
  ```
- ManyToManyFieldがある側(Subjetcs)  
  インスタンス名.フィールド名.all()でデータを取得できる  
  または前と同じようにインスタンス名.モデル名_set.all()でアクセスできる
  ```py
  subject = Subjects.objects.get(pk=1)
  subject.teachers.all()
  # or
  subject.teachers_set.all()
  ```

### 中間テーブルの独自定義
**Person** と **Group** の中間テーブル **Membership** を作成する  
**through = Membership** でDjangoに中間テーブルを作らせず自前のモデル(Membership)を使わせる  
中間テーブルに1対多で関連付けられるモデルクラスが2つ以上ある時、ManyToManyField内のフィールドオプションとして**through_fields**内に、関連付けるフィールドを明示的に指定する必要がある。

(中間テーブルのForeignKeyに逆参照名(related_name)を定義しようとすると怒られる)　　
```py
class Person(models.Model):
    name = models.CharField(max_length=50)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        through='Membership',
        through_fields=('group', 'person'),
    )

class Membership(models.Model):
    group = models.ForeignKey(
        Group, 
        on_delete=models.CASCADE
    )
    person = models.ForeignKey(
        Person,
        on_delete=models.CASCADE
    )
    inviter = models.ForeignKey(
        Person,
        on_delete=models.CASCADE,
        related_name="membership_invites",
    )
    invite_reason = models.CharField(max_length=64)
```
ここまでくるとかなりリレーションが複雑になってくる。だからあえてManyToManyFieldを使わないという選択肢も視野に入れるべきでは無いかと思われる。

## 複雑なクエリの作成
### Where句
絞り込みを行う(filter)  
他にもあるのでドキュメント読む  
```py
# field=''で絞り込む
Model.objects.filter(field='')
# 複数条件
Model.objects.filter(field1='', field2='')
# Aで始まる行だけ _startswith
Model.objects.filter(field_startswith='A')
# Aで終わる行だけ _endswith
Model.objects.filter(field_endswith='A')
# 20より大きい __gt
Model.objects.filter(age__gt=20)
# 20より小さい __lt
Model.objects.filter(age__lt=20)
# 20以上 __gte
Model.objects.filter(age__gte=20)
# 20以下 __lte
Model.objects.filter(age__lte=20)
# 1つのフィールドに対して複数条件どれかにヒットするレコードのみ
Model.objects.filter(field__in=['AAA', 'BBB'])
# 〇〇を含むレコードのみ(大文字小文字を区別)
Model.objects.filter(field__contains='hoge')
# 〇〇を含むレコードのみ(大文字小文字を区別しない)
Model.objects.filter(field__icontains='fuga')
# Nullレコードのみ
Model.objects.filter(field__isnull=True)
```

指定した条件以外のレコードのみ(exclude)  
```py
# field='hoge' 以外のレコードのみ
Model.objects.exclude(field='hoge')
```

OR条件を使った絞り込み(Q)  
```py
# Q と | を使う
from django.db.models import Q
Model.objects.filter(Q(field1=''), | Q(field2=''))
```

### Select句
一部のカラムのみ取り出す(values)
```py
# column1とcolumn2 だけ取り出す
Model.objects.values('column1', 'column2').all()
```

### Order by句（集計）
順番を並び替える(order_by)  
```py
# column1で昇順に並び替え
Model.objects.order_by('column1')
# column1で降順に並び替え
Model.objects.order_by('-column1')
# col1で1で昇順にした後、col2で昇順に並び替え
Model.objects.order_by('col1', 'col2')
```

### Group by句
集計する
```py
from django.db.models import Max, Min, Avg, Sum
# 件数をカウントする count
Model.objects.count()
# 最大値 aggregate(Max())
Model.objects.aggregate(Max('col1'))
# 最小値 aggregate(Min())
Model.objects.aggregate(Min('col1'))
# 平均値 aggregate(Avg())
Model.objects.aggregate(Avg('col1'))
# 合計値 aggregate(Sum())
Model.objects.aggregate(Sum('col1'))
```
GROUP BY でカラムに応じて集計する(annonate)  
```py
# col1 で Group by して col2 の最大値を計算する
Model.objects.values('col1').annonate(
    Max('col2')
)
```

### 外部テーブルも含めて処理を行う(外部テーブル__)
外部テーブルのカラムを指定して絞り込み
```py
# Foreign 外部テーブルの field カラムの中の AAAで絞り込む
Model.objects.filter(Foreign__field='AAA')
# 外部テーブルは連結できる　外部テーブルの外部テーブルのカラムを指定
Model.objects.filter(Foreign1__Foreign2__field='AAA')
# order by も同様
Model.objects.order_by(Foreign__field)
# group by も同様
Model.objects.values('Foreign__col1').annonate(
    Count('col2')
)
```

実際のSQLの確認方法(.query)
```py
print(Model.objects.filter(age__lte=20).query)
```
