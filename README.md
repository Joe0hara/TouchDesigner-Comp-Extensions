# TouchDesigner-Comp-Extensions

CompのExtensionsの使い方についての解説

## Extensionsとは
<a href="https://www.derivative.ca/wiki099/index.php?title=Extensions">公式ドキュメント</a>
**めっちゃ簡単に言うと、**Componentに変数や関数などを拡張するためのものです。
CompのCustom Parametersだけでは足りなく、機能を拡張したいときに使うことができます。
パラメーターのみを追加したい場合は<a href="https://qiita.com/kodai100/items/0dea9936bc6204012781">こちらの記事がオススメ</a>

## Setup
まず、Containerを作り、ParameterのExtensionsタブを選択し確認します。

Extensionsを追加するにはComponent Editorからと直接書く方法の２つがあります。
Component Editorからの作成が簡単でオススメですのでそちらの解説します。
オペレータを右クリックし **Customize Component** からComponent Editorを開きます。
<table><td><img src="https://qiita-image-store.s3.amazonaws.com/0/175399/15d48f1c-5716-c3eb-bbf5-7a0e97f83473.png">
</td>
<td><img src="https://qiita-image-store.s3.amazonaws.com/0/175399/1c7e6350-5d26-7f6d-99db-5210feb394dd.png">
</td>
<td><img src="https://qiita-image-store.s3.amazonaws.com/0/175399/cc0d6779-3ae3-2340-c49e-26dafa02229a.png">
</td>
</table>

Extensions1に TestExt と入れAddします。
そうするとContainerの中にTextExtと名前のついたTextDATが生成されます。
<img src="https://qiita-image-store.s3.amazonaws.com/0/175399/1e8fee0e-1298-6764-a967-1427869ad8f3.png">

これでExtensionsを使う準備ができました！


### attribute
```
self.a = 0  # attribute
self.B = 1  # promoted attribute
```
属性とその値を設定します。（属性名がキャピタライズされているとpromoted Attributeとされる。説明後述）

### Property
```
TDF = op.TDModules.mod.TDFunctions
TDF.createProperty(self, 'MyProperty', value=0, readOnly=False)
```
MyPropertyというプロパティを作ります。
valueにデフォルト値、readOnlyで読み取り専用かどうかの設定ができます。

### Stored Items
```
from TDStoreTools import StorageManager
storedItems = [{'name': 'StoredProperty', 'default': None, 'readOnly': False, 'property': True},]
self.stored = StorageManager(self, ownerComp, storedItems)
```
StoredPropertyというStored Itemsを作ります。
これは再起動などをしても初期化されないものです。

### Function
```
def myFunction(self, v):
    debug(v)
def PromotedFunction(self, v):
    debug(v)
```
関数もこのように定義できます。


# #Promote
変数名、関数名の頭文字が大文字であると、それはPromotedとなるんです(なんて訳すべきかわかりません！)。
Promotedであるかないかでアクセスの仕方が変わります。
publicやprivateみたいな感じです。
ExtensionsタブのExtension NameとPromote Extensionという値が重要となります。



## Extensionsへのアクセス方法
+ ExtensionsタブのPromote ExtensionがONの場合

```
op('container1').B
op('container1').MyProperty
op('container1').StoredProperty
op('container1').PromotedFunction('test')
```
これらにはアクセスできるが、

```
op('container1').a
op('container1').myFunction('test')
```

これらにはアクセスできません。

+ ExtensionsタブのPromote ExtensionがOFFの場合

OFF時にデータにアクセスしたいときは、まずExtensionsタブのExtension Nameを設定します。
<img src="https://qiita-image-store.s3.amazonaws.com/0/175399/9f36d0f8-7fa9-ce07-8141-74f2e829ffa4.png">
※Extensionsタブの値を変えたら必ずRe-Init Extensionsのボタンを押すこと
<img src="https://qiita-image-store.s3.amazonaws.com/0/175399/e439b8e5-aa34-183d-3fc8-2de7587fc0c3.png">


```
op('container1').ext.TestNameExt.B
op('container1').ext.TestNameExt.PromotedFunction('test')
op('container1').ext.TestNameExt.a
op('container1').ext.TestNameExt.myFunction('test')
```
このように記述することで先ほどアクセスできなかったところまでアクセスできるようになります。

## 例
```:testExt
from TDStoreTools import StorageManager
TDF = op.TDModules.mod.TDFunctions

class TestExt:
	def __init__(self, ownerComp):
		self.ownerComp = ownerComp

		self.baseColorList = [
			[1, 0, 0],
			[0, 1, 0],
			[0, 0, 1],
			[1, 1, 1]
		]
 
		storedItems = [
			{'name': 'ColorIndex', 'default': 0, 'readOnly': True},
		]
		self.stored = StorageManager(self, ownerComp, storedItems)
 
		TDF.createProperty(self, 'BaseColor',
				   value=self.baseColorList[self.ColorIndex],
				   readOnly=True, dependable=True)
 
	def IncrementBaseColor(self):
		self.stored['ColorIndex'] += 1
		if self.ColorIndex == len(self.baseColorList):
			self.stored['ColorIndex'] = 0
		self._BaseColor.val = self.baseColorList[self.ColorIndex]
```
<img src="https://qiita-image-store.s3.amazonaws.com/0/175399/c0fd870b-4571-4380-9bbd-bdad52775089.png">

```:textDAT
op("container1").IncrementBaseColor()
```

![](https://qiita-image-store.s3.amazonaws.com/0/175399/4f5fc758-6823-50b9-66b0-62c3ec3dc131.gif)

<a https://github.com/Joe0hara/TouchDesigner-Comp-Extensions">toeFile</a>

## 最後に
TouchDesignerではこんなことしなくても色々と簡単にできるので、ほとんど使わないかもしれませんが機能紹介ということで記事を書きました。
