# Form
## forms.pyでフォームを作る
```py
class UserForm(forms.Form):
    name = forms.CharField(label='Your Name', max_length=100)
    age = forms.IntergerField()
    mail = forms.EmailField()
```
## views.pyでtemplateにformを渡す
```py
form = UserForm()
# contextで渡す
return render(request, 'template_path', context={'form': form})
```
## templateで記述して表示する
```html
<!-- formタグで囲む -->
<form action="/your-name/" method="post">
    <!-- CSRF攻撃対策用に乱数を生成する -->
    {% csrf_token %}
    <!-- formを表示 -->
    {{ form }}
    <!-- 送信用のボタン -->
    <input type="submit" value="Submit">
<form>
```
### フォーマットして表示も可能  
{{ form.as_p }} \<p>タグでラップして表示  
{{ form.as_ul }} \<li>タグでラップして表示  
{{ form.as_table }} \<tr>タグでラップして表示  

## Formを受け取ったViewの処理　　
基本的にrenderするviewsの関数がformを受け取る  
1. リクエストがPOSTかをチェック
2. 送られたリクエストのデータでフォームクラスを初期化
3. 初期化したものが正しい処理かをチェック
4. formから値を取り出す
```py
# リクエストがPOSTかをチェック
if request.method == 'POST':
    # 送られたリクエストのデータでフォームクラスを初期化
    form = forms.UserInfo(request.POST)
    # 初期化したものが正しい処理かをチェック
    if form.is_valid():
        # formから値を取り出す
        form.cleaned_data['subject']
```

## FormのFieldのカスタマイズ
ラベルの変更
```py
field = forms.CharField(label='名前')
```
入力必須を外す
```py
field = forms.CharField(required=False)
```
ウィジット(入力形式)を変更
```py
field = forms.CharField(widget=forms.Textarea)
```
初期値を設定
```py
field = forms.CharField(initial="hoge")
```
プレースホルダー(入力のヒント)を追加
```py
field = forms.CharField(widget=forms.TextInput(attrs={'placeholder': 'hoge@mail.com'}))
```
文字列の最大、最小の長さを指定
```py
field = forms.CharField(max_length=10)
```

## 要素のID、クラスの追加
### widget内にattrsを指定する方法
```py
form = forms.EmailField(widget=forms.TextInput(
    attrs={'class': 'hoge','id': 'fuga'}
))
```
### formクラスのコンストラクタ__init__をオーバーライドする方法
```py
class UserForm(forms.Form)
    # 色々書いて...
    def __init__(self, *args, **kwargs):
        # 親クラスを呼び出す処理 必ず必要
        super(Form, self).__init__(*args, **kwargs)
        # widget.attrs 以降でid名を指定
        self.fields['job'].widget.attrs['id'] = 'id_job'
        # class名を指定
        self.fields['hobbies'].widget.attrs['class'] = 'class_hobbies'
user_form = UserForm()
```
### updateメソッドで付与する方法
```py
class CommentForm(forms.Form):
    name = forms.CharField()
    url = forms.URLField()
    comment = forms.CharField()
    # フィールド.widget.attrs.update({'class or id': 'name'})
    name.widget.attrs.update({'class': 'hoge'})
    comment.widget.attrs.update(size='40)
```

## staticファイル
### 読み込み先の設定
settings.py の STATICFILES_DIRS にディレクトリを指定する
```py
# 静的ファイル配置先のディレクトリへのURL。http://ドメイン名:ポート名/static/フォルダ/ファイル名でアクセスできるようになる
STATIC_URL = '/static/'
# 例
STATIC_DIR = os.path.join(BASE_DIR, "static")
STATICFILES_DIRS = [
    STATIC_DIR,
]
```
### staticファイルの利用
templateにて {% load static %} し "{% static '場所' %}" で指定
```html
{% load static %}
<link rel="stylesheet" href="{% static 'css/style.css' %}">
```

## Formのバリデーション
1. ユーザーがformをsubmitする
2. views の form.is_valid()でvalidationを実行する
3. forms の form.clean()が呼び出される
4. forms の clean_フィールド名 が呼び出される
### is_valid()の挙動
is_valid()はooleanを返す  
Falseが帰った時、form.errorsにエラーの内容が格納される
### form.clean()
複数フィールドのvalidationをいっぺんに書きたい時もclean()を使う
```py
class ContactForm(forms.Form):
    def clean(self):
        # 親クラスのcleanを呼び出す
        # 定義された他のvalidationも自動で実行される
        cleaned_data = super().clean()
        # ユーザから送られてきた値を得る
        cc_myself = cleaned_data.get('cc_myself')
```
### form.clean_フィールド名
フィールドごとのvalidationを作りたい時もこちら
```py
class ContactForm(forms.Form):
    def clean_recipients(self):
        data = self.cleaned_data['recipients']
        return data
```
### フィールドに対して validators=[]で属性を追加して実装もできる
```py
class ContactForm(forms.Form):
    age = forms.IntergerField(
        label="年齢", 
        validators=[validators.MinValueValidator(20)]
    )
```
### 自作validatorも作成できる
```py
def my_valid(val):
    if val == "":
        raise raise forms.ValidationError()

class UserInfo(forms.Form):
    name = forms.CharField(
        label='名前',
        min_length=5, max_length=10,
        # ここで指定 もちろん複数も指定できる
        validators=[my_valid]
    )
```

## ModelForm
Formでバリデーションして、チェックした内容をそのままModelに格納する  
ModelにあるDBテーブルの内容をそのままviewに表示して記入を求めるときに便利  
### 元となるmodelの定義
```py
class Post(models.Model):
    aaa = models.CharField(max_length=50)
```
### Form側でModelFormの定義
```py
# 継承はModelForm
class PetModelForm(forms.ModelForm):
    # 元のmodelの内容を修飾したい場合はここに書く
    memo = forms.CharField(
        widget=forms.Textarea(attrs={'rows':30, 'cols':30})
    )
    class Meta:
        # Modelを指定
        model = hoge
        # 画面に表示するフィールドを配列で指定('__all__'とすると全フィールドを指定)
        fields = []
        # 表示しないフィールドを配列で指定
        exlude = []
```

### View側でModelFormの保存
```py
if form.is_valid():
    form.save()
```

## save()のカスタマイズ
Modelにデータを保存するsave()をオーバーライドしてデータ保存直前に値の書き換えなどができる
```py
class MyModelForm(forms.ModelForm):
    memo = forms.CharField(max_length=200)
    slug = forms.SlugField()
    # save()のオーバーライド
    def save(self, *args, **kwargs):
        self.slug = slugify(self.title)
        # 親クラス(forms.ModelForm)のsave()の呼び出してmodelの保存を行う(本来呼び出されるべきsave())
        super(MyModelForm, self).save(*args, **kwargs)
```
formsのクラスにフィールドを持っていない時(ModelFormの時など)は、super().save()の時にcommit=Falseとすることで、保存はせずに保存可能状態のオブジェクトのみを取ってこれる
```py
class PostModelForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = '__all__'
    
    def save(self, *args, **kwargs):
        # commit=Falseとしてobjのみとってくる
        obj = super(PostModelForm, self).save(commit=False, *args, **kwargs)
        # 加工する
        obj.name = obj.name.upper()
        # 保存する
        obj.save()
        # オブジェクトを返す(必ず)
        return obj
```

## 共通の機能(ログ出力とか)の作り方
### 共通化したい処理を書く
forms.ModelFormを継承した子クラスを作る  
```py
class BaseForm(forms.ModelForm):
    # 今回はsave()が呼ばれた時の共通処理
    def save(self, *args, **kwargs):
        # save()が呼ばれたときにprint()を行う共通処理
        print(f'Form: {self.__class__.__name__}実行')
        # forms.ModelFormのsave()を行う
        return super(BaseForm, self).save(*args, **kwargs)
```
### 共通化したい処理を呼び出す
BaseFormを継承したクラス、つまりforms.ModelFormの孫クラスを定義する
```py
class PostModelForm(BaseForm):    
    def save(self, *args, **kwargs):
        # 親クラス(BaseForm)のsave() ただしコミットは行わない
        obj = super(PostModelForm, self).save(commit=False, *args, **kwargs)
        obj.name = obj.name.upper()
        obj.save()
        return obj
```

## formの各要素ごとの表示
as_p, as_ul, as_tableなどで自動的に各フィールドを表示させることもできるが、フィールド1つずつ直接指定して表示させることもできる
```html
<!-- フィールドのnameの表示 -->
{{ form.name }}
<!-- フィールドnameのラベルの表示 -->
{{ form.name.label }}
<!-- フィールドnameに対するエラーを表示 -->
{{ form.name.errors }}
<!-- validationで発生したフィールド単体でないエラーの表示 -->
{{ form.non_field_errors }}
<!-- 全フィールドのエラーをループさせて、１箇所に表示
templateがModelFormにgetterなどを投げている構図となる -->
<!-- 全体のエラーをまとめて表示させる時に便利 -->
{% for key, value in form.errors.item %}
    <p>{{ key }}: {{ value.as_text }}</p>
{% endfor %}
```

## 外部のhtmlを表示する
### includeで別のHTMLを読み込む方法
```html
<!-- other.html -->
<p>AAA</p>

<!-- main.html -->
{% include "other.html" %}
```
### includeと一緒にwithで値を渡す方法
```html
<!-- other.html -->
<p>{{ var1 }}</p>

<!-- main.html -->
{% include "other.html" with var1='hoge' %}
```

## Formsetで複数のFormを表示する
### viewで何を何回記述するかを記述する
formset(formクラス名, extra=何回表示させるか)
```py
TestFormset = formset_factory(SampleForm, extra=3)
formset = TestFormset()
```
### templateでformsetの中身を表示させる
as_pまたはforループで表示させる
```html
{{ formset.as_p }}
<!-- or -->
<!-- fomsetをループする際に直前に記載する -->
{{ formset.management_form }}
{% for form in formset %}
    {{ form }}
{% endfor %}
```
### viewでデータを取り出す
同じformを複数回表示させる中で、その複数回の内、全てのフィールドについて記入がなくてもis_valid()ではエラーとはならない  
複数回の内、一部のフィールドについて記入がなかった場合はis_valid()でエラーとなる  
これはformsetの仕様である
```py
if formset.is_valid():
    for form in formset:
        # cleaned_dataの処理
```

## ModelFormSetで複数一括で保存する
### viewの記載
```py
# 対象のモデルとフォームを指定
# 第一引数：利用したいモデル名、第二引数：
TestFormset = modelformset_factory(Model, form=ModelForm, extra=1)
# or
# 対象のモデルと表示するフィールド
Test
```
### Templateの記載
前回のFormsetで複数のFormを表示すると一緒

### viewでデータを保存する
```py
if formset.is_valid():
    # 丸ごと保存
    formset.save()
```

## 単純な画像のアップロード
### settings.pyにファイルの保存先設定を記述する
```py
# メディアファイルを公開する際のURL
MEDIA_URL = '/media/'
# ファイルの保存先
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```
### テンプレートにインターフェースを作る
```html
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    <input type="file" name="myfile">
    <input type="submit" value="送信">
</form>
```
### viewの記述
```py
# 送られたファイルの取り出し
myfile = request.FILES['myfile']
# ファイル保存用のクラス
fs = FileSystemStorage()
# ファイルの保存
filename = fs.save(myfile.name, myfile)
# ファイルのurl
uploaded_file_url = fs.url(filename)
```

## Modelを利用した画像のアップロード
他にフィールドがありその一部として画像がある場合で、画像のurlをmodelで保存したい時など  
基本こっちを使うことになる  
### urls.pyにファイルを表示するための設定を追加
```py
from django.conf import settings
from django.conf.urls.static import static
# DEBUG(開発環境)モードでのみ付け加える(apatch, nginxといったwebサーバがある場合は不要)
# ファイルとかはwebサーバに置くものである。applicationサーバに置くときは以下が必要
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
### ModelにFileFieldの追加
```py
class Image(models.Model):
    description = models.CharField(max_length=255, blank=True)
    # アップロードするファイルの格納先
    # upload_to = 'documents/%Y/%m/%d/' とすると documents/yyyy/mm/dd で保存できる
    path = models.FileField(upload_to='documents/)
```
### viewの記載
```py
def model_form_upload(request):
    if request.method == 'POST':
        # 保存するファイルを第二引数に書く
        form = ImageForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('home')
```
### テンプレートの記載
```html
<form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="保存">
</form>
<!-- picture.urlをsrcに入れて画像を表示 -->
<img src="{{ user.picture.url }}">
```
