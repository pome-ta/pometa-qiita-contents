---
title: Pythonista3 でPythonista3 内のディレクトリを探訪しよう！
tags:
  - Python
  - Mobile
  - MobileApp
  - Pythonista3
  - pathlib
private: false
updated_at: '2022-12-10T07:01:17+09:00'
id: d7ce9fe64e1ba774fae4
organization_url_name: null
slide: false
ignorePublish: false
---

この記事は、[Pythonista3 Advent Calendar 2022](https://qiita.com/advent-calendar/2022/pythonista3) の 10 日目の記事です。

👇 : 09 日目

https://qiita.com/pome-ta/items/0e2d3bd3e22fe922f2c0

👇 : 11 日目

https://qiita.com/pome-ta/items/3315732c8db62a55b86b

https://qiita.com/advent-calendar/2022/pythonista3

一方的な偏った目線で、Pythonista3 を紹介していきます。

ほぼ毎日 iPhone（Pythonista3）で、コーディングをしている者です。よろしくお願いします。

以下、私の 2022 年 12 月時点の環境です。

```sysInfo.log
--- SYSTEM INFORMATION ---
* Pythonista 3.3 (330025), Default interpreter 3.6.1
* iOS 16.1.1, model iPhone12,1, resolution (portrait) 828.0 x 1792.0 @ 2.0) 828.0 x 1792.0 @ 2.0
```

他の環境(iPad や端末の種類、iOS のバージョン違い)では、意図としない挙動(エラーになる)なる場合もあります。ご了承ください。

ちなみに、`model iPhone12,1` は、iPhone11 です。

## グラフィクスプログラミングハラスメント

前回までは、絵を出すプログラミングで Shader の GLSL や、path で線を引く方法を紹介しました。

しかも、かっこいい絵を出すテクニックではなく、グラフィクスプログラミングをするための環境作りのみで、残りはみなさんにぶん投げる着地でございました。。。

path を引く際の数式をゴリゴリ紹介し、「こんないい感じになるんやで！」と、したかったのですが、センス・知識・紹介技量全てが足りず、いつかは紹介したいと思っております。

今回は、前回の path とは違う path を使って、絵を出すとは違うプログラミングをやっていこうと思います。

## Pythonista3 から、Pythonista3 を覗き見る

Pythonista3 では「ScriptLibrary」内に自分のコードを置いたり、「ExternalFiles」で外部のフォルダやファイルにアクセスできたり、モジュール関係のファイルが入っているディレクトリがあったり、名称によりさまざまな意味でディレクトリが多数存在します。

- ScriptLibrary
  - 主に自分の作成したコードを格納する場所
- ExternalFiles
  - 外部（ファイル App や他のアプリ）のファイルやフォルダを参照する
- Python Modules
  - 標準モジュール、Pythonista3 で用意されているモジュールの格納場所
  - 自作モジュールも格納できるよう準備されれいる

それらは、Pythonista3 の GUI でタップすることで、行ったり来たり選んだりが、気軽にできます。

また GUI 操作以外でも、Python は、標準でファイル・ディレクトリ操作ができるモジュールがあるので、コードから該当のファイルへアクセスしたり、新規作成等が可能です。

これまででも、特に Editor Action を使ったコードでファイルの新規作成などで使ってきました。

プログラミングをするにあたり、フォルダ構成の理解は必要な部分かと思われます。

まずは、簡単なコードで状況を確認してみましょう。

### 注意事項

:::note warn
免責事項として書いています。
:::

アプリや OS 内部構造を操作することになります。

変更や削除で、アプリや OS が起動不可になることもありえます。

不具合に関しては自己責任でお願い致します。

### `pathlib` モジュール

[11.1. pathlib --- オブジェクト指向のファイルシステムパス — Python 3.6.15 ドキュメント](https://docs.python.org/ja/3.6/library/pathlib.html)

https://docs.python.org/ja/3.6/library/pathlib.html

Pythonista3 が、`3.6` なので一応`3.6.15` のバージョンで見ています。

`os` モジュール等でも可能ですが、私的に`pathlib` が使いやすいので、`pathlib` で確認していきます。`os`、`sys` 等が使いやすい方は、脳内読み替えでお願いいたします 🙇

公式ドキュメントで迷子になった時は、こちらにお世話になってるます。

[Python, pathlib の使い方（パスをオブジェクトとして操作・処理） | note.nkmk.me](https://note.nkmk.me/python-pathlib-usage/)

https://note.nkmk.me/python-pathlib-usage/

#### 現在位置確認

適当な位置でファイルを作成し、現在地を見てみます:

```py
from pathlib import Path

path = Path()
abs_path = path.resolve()

print(abs_path)

```

[pathlib.Path.resolve | pathlib --- オブジェクト指向のファイルシステムパス — Python 3.11.0b5 ドキュメント](https://docs.python.org/ja/3/library/pathlib.html#pathlib.Path.resolve)

https://docs.python.org/ja/3/library/pathlib.html#pathlib.Path.resolve

ScriptLibrary 直下に作成したファイルから呼び出した場合は以下の結果が出力されます。

```
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/Pythonista3/Documents
```

`それぞれ-違う-英数字` には、個別端末（？）ごとに違う表記となります。

ExternalFiles からファイル App の`ダウンロード` 直下にあるコードを実行すると以下の結果で出力されるのが確認できます。

```
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/File Provider Storage/Downloads
```

![img221127_142049.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/84e1f71f-f306-08df-e0a2-a696aec588c4.png)

注意点としては、ExternalFiles へ取り込む際に`Python/Text File...` ではなく`Folder...` から、適用したいフォルダを選択することです。単品ファイルを選択すると取得できません。

![img221127_143357.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/4381408c-57a1-5872-c254-e863e9509beb.png)

```
Traceback (most recent call last):
  File "/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/File Provider Storage/Downloads/pathtest.py", line 4, in <module>
    abs_path = path.resolve()
  File "/var/containers/Bundle/Application/それぞれ-違う-英数字/Pythonista3.app/Frameworks/Py3Kit.framework/pylib/pathlib.py", line 1123, in resolve
    s = self._flavour.resolve(self, strict=strict)
  File "/var/containers/Bundle/Application/それぞれ-違う-英数字/Pythonista3.app/Frameworks/Py3Kit.framework/pylib/pathlib.py", line 349, in resolve
    base = '' if path.is_absolute() else os.getcwd()
PermissionError: [Errno 1] Operation not permitted

```

うーん、面白いですね ☺️

#### ScriptLibrary 直下のフォルダやファイル

ScriptLibrary 直下に（わかりやすく）ファイル名を`rootThisFile.py` として作成して、同階層に何があるか見てみましょう。

```rootThisFile.py
from pathlib import Path

path = Path()
abs_path = path.resolve()

print(abs_path)
for f in abs_path.glob('*'):
  print(f'\t- {f.name}')

```

`.glob('*')` として、ゴリっと取得したものを出力しています。

[Python, pathlib でファイル一覧を取得（glob, iterdir） | note.nkmk.me](https://note.nkmk.me/python-pathlib-iterdir-glob/)

https://note.nkmk.me/python-pathlib-iterdir-glob/

```
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/Pythonista3/Documents
 - site-packages-3
 - Welcome.md
 - rootThisFile.py
 - _objc_exception.txt
 - site-packages-2
 - Templates
 - site-packages
 - myScripts
 - .style.yapf
 - Examples
 - .Trash
 - ws

```

##### 脚注を加えます

ScriptLibrary で、確認できるものや自分で作成したもの以外に意外なフォルダやファイルが出てきました。

```
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/Pythonista3/Documents
 - site-packages-3       ← (*1)
 - Welcome.md
 - rootThisFile.py       ← 今回実行したスクリプト
 - _objc_exception.txt   ← objc_util 系のエラーログ
 - site-packages-2       ← (*1)
 - Templates             ← (*2)
 - site-packages         ← (*1)
 - myScripts             ← 私のEditor Action 用スクリプトが格納されているやつ
 - .style.yapf           ← (*3)
 - Examples
 - .Trash                ← (*3)
 - ws                    ← 私がコードを作成して編集する作業用フォルダ

```

![img221127_152043.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/9825e572-b185-1786-f699-3c8e1702720d.png)

GUI 上で見えている部分が、実は同じ階層で管理されているのですね。

`(*1)` の`site-packages` たちは、自分で入れたモジュールが格納されているディレクトリです。

また`(*3)`ドットファイル（先頭に`.` があるファイルやフォルダ）は、GUI では非表示設定になっています。

Pythonista3 （GUI）上で見えている情報と、出力した階層の情報が違うことがわかりますね。

### 一つ上の階層は？

もっと、探訪したくなりますね！

`pathlib` の`.parent` で上の階層から呼び出してみましょう。

```py
from pathlib import Path

path = Path()
abs_path = path.resolve()
parent_path = abs_path.parent

print(parent_path)
for f in parent_path.glob('*'):
  print(f'\t- {f.name}')

```

なんということでしょう。見たことがないファイルたちが出てきました。

```
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/Pythonista3
 - ExtensionShortcuts.json
 - Documents
 - Bookmarks.plist
 - .matplotlib
 - Library
 - Favorites.plist
 - Tabs.plist
 - KeyboardShortcutDefinitions.json
 - Recents.plist

```

`Documents` が、我々が普段何かしらしているディレクトリです。

`.parent` を追加して、更なる上の階層を見てみましょうか:

```py
from pathlib import Path

path = Path()
abs_path = path.resolve()
parent_path = abs_path.parent.parent

print(parent_path)
for f in parent_path.glob('*'):
  print(f'\t- {f.name}')

```

Pythonista3 アプリ自体のディレクトリになるのでしょうか？

だんだんと表示される情報が減ってきました。

```
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字
 - .com.apple.mobile_container_manager.metadata.plist
 - Library
 - Pythonista3
```

### ファイルに何が書かれているか気になる

階層の探訪をしてきました。フォルダ以外にファイルも表示されているので、見たことある拡張子（`.json` など）の中を読んでみたくなりますよね？私はなります ☺️

`pathlib` モジュールは、`open` を使った`with` を使わずとも、1 行でテキスト情報を取得できるます。

`.read_text` を使って、中身を見てみましょう。

[Python, pathlib でファイルの作成・open・読み書き・削除 | note.nkmk.me](https://note.nkmk.me/python-pathlib-file-open-read-write-unlink/)

https://note.nkmk.me/python-pathlib-file-open-read-write-unlink/

#### 読み取るコード

少々手間ですが、一度ファイル情報を取得してから、取得したいファイルを確認後、指定ファイルを見ていくことにします。

##### `.json` ファイルを探す

```py
from pathlib import Path

root_path = Path().home()

print('検索するディレクトリ:')
print(root_path)

for f in root_path.glob('*.json'):
  print('\n.json ファイル ---')
  print(f)
  print('--- /')

```

`.home` で、`Pythonista3` ディレクトリ内から探します。

ScriptLibrary 直下の場合、`.parent` 一回呼び出しの階層です。

```
検索するディレクトリ:
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/Pythonista3

.json ファイル ---
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/Pythonista3/ExtensionShortcuts.json
--- /

.json ファイル ---
/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/Pythonista3/KeyboardShortcutDefinitions.json
--- /

```

2 つの`.json` ファイルが見つかりました。

##### `.json` ファイルを見る

`ExtensionShortcuts.json` を見てみることにしましょう。

console より出力された、ファイルパスを何かしらでコピーし:

![img221128_224532.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/a9d2c723-b78f-38e3-9e19-03b36b32c971.png)

```py
from pathlib import Path

json_file = Path(
  '/private/var/mobile/Containers/Shared/AppGroup/それぞれ-違う-英数字/Pythonista3/ExtensionShortcuts.json'
)

txt = json_file.read_text()
print(txt)

```

![img221127_163110.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/4f0d6266-abb3-7eed-38dc-30c839c4aa4a.png)

![img221127_163122.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/0f81a762-2469-9e4b-27c6-e3d838c86835.png)

```json
[
  {
    "scriptName": "Examples/Extension/Copy Photo Location.py",
    "iconName": "ios7-world",
    "iconColor": "6CB8FF"
  },
  {
    "scriptName": "Examples/Extension/Image Histogram.py",
    "iconName": "stats-bars",
    "iconColor": "6CFF7F"
  },
  {
    "scriptName": "Examples/Extension/Text Statistics.py",
    "iconName": "document-text",
    "iconColor": "FFBD62"
  },
  {
    "scriptName": "Examples/Extension/URL to QR Code.py",
    "iconName": "grid",
    "iconColor": "62F0FF"
  },
  {
    "scriptName": "Examples/Extension/Preview Markdown.py",
    "iconName": "ios7-eye",
    "iconColor": "4A525A"
  }
]
```

無事に取得できました。

整形すると:

```json
[
  {
    "scriptName": "Examples/Extension/Copy Photo Location.py",
    "iconName": "ios7-world",
    "iconColor": "6CB8FF"
  },
  {
    "scriptName": "Examples/Extension/Image Histogram.py",
    "iconName": "stats-bars",
    "iconColor": "6CFF7F"
  },
  {
    "scriptName": "Examples/Extension/Text Statistics.py",
    "iconName": "document-text",
    "iconColor": "FFBD62"
  },
  {
    "scriptName": "Examples/Extension/URL to QR Code.py",
    "iconName": "grid",
    "iconColor": "62F0FF"
  },
  {
    "scriptName": "Examples/Extension/Preview Markdown.py",
    "iconName": "ios7-eye",
    "iconColor": "4A525A"
  }
]
```

[Share Sheet Extension | App Extensions and Shortcuts — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/pythonista_shortcuts.html#pythonista-share-extension)

https://omz-software.com/pythonista/docs/ios/pythonista_shortcuts.html#pythonista-share-extension

Share Sheet Extension の情報が入っている`.json` ファイルのようですね。

![img221127_163447.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/1f05090c-e91a-fb48-9d5c-33dc948ee973.png)

![img221127_163455.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/f65239a2-5afb-08b2-4994-368085728057.png)

こんな所に格納され設定されていたのですね 😎

## 次回は

`pathlib` で、ディレクトリを移動しながら`print` で情報を出力しました。

GUI のフォルダ構成とは裏腹に、内部の構成がされてましたね。

気になるフォルダ名、ファイル名を探して中身を確認してみてください ☺️

とはいえ、毎回`print` 確認をするのは面倒で骨が折れます。。。

そこで、次回は`webbrowser` モジュール等を使ってディレクトリ探訪をもっと楽にしましょう。

標準モジュールである、`webbrowser` ですが Pythonista3 用に一部カスタマイズされているのです ☺️

ここまで、読んでいただきありがとうございました。

👇 : 11 日目

https://qiita.com/pome-ta/items/3315732c8db62a55b86b

## せんでん

### Discord

Pythonista3 の日本語コミュニティーがあります。みなさん優しくて、わからないところも親身に教えてくれるのでこの機会に覗いてみてください。

https://t.co/0IOW2hn1VC

### 書籍

iPhone/iPad でプログラミングする最強の本。

- [Python や Jupyter で iPhone/iPad 先端機能を簡単･自由にプログラミング！「活用篇」：hirax](https://techbookfest.org/product/1WiyV4LEev3f4YrzesRKWV?productVariantID=axX32srsSgtLtZWUtjkj99)

https://techbookfest.org/product/1WiyV4LEev3f4YrzesRKWV?productVariantID=axX32srsSgtLtZWUtjkj99

- [Python や Jupyter で iPhone/iPad 先端機能を簡単･自由にプログラミング！「土台篇」：hirax](https://techbookfest.org/product/wTZTyeibm5GQ5XgdfMrEBV?productVariantID=kRDmN1udbEYZUWbETdwL8r)

https://techbookfest.org/product/wTZTyeibm5GQ5XgdfMrEBV?productVariantID=kRDmN1udbEYZUWbETdwL8r

### その他

- サンプルコード

[Pythonista3 Advent Calendar 2022](https://qiita.com/advent-calendar/2022/pythonista3) でのコードをまとめているリポジトリがあります。

コードのエラーや変なところや改善点など。ご指摘や PR お待ちしておりますー

https://github.com/pome-ta/Pythonista3AdventCalendar2022sampleCode

- Twitter

なんしかガチャガチャしていますが、お気兼ねなくお声がけくださいませー

https://twitter.com/pome_ta93

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">やれるか、やれないか。ではなく、やるんだけども、紹介説明することは尽きないと思うけど、締め切り守れるか？って話よ！（クズ）<br><br>Pythonista3 Advent Calendar 2022 <a href="https://t.co/JKUxA525Pt">https://t.co/JKUxA525Pt</a> <a href="https://twitter.com/hashtag/Qiita?src=hash&amp;ref_src=twsrc%5Etfw">#Qiita</a></p>&mdash; pome-ta (@pome_ta93) <a href="https://twitter.com/pome_ta93/status/1588570697777152006?ref_src=twsrc%5Etfw">November 4, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- GitHub

基本的に GitHub にコードをあげているので、何にハマって何を実装しているのか観測できると思います。

https://github.com/pome-ta
