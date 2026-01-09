---
title: Pythonista3 のscene モジュールでShader コーディング。GLSL をやる。
tags:
  - Python
  - Mobile
  - GLSL
  - MobileApp
  - Pythonista3
private: false
updated_at: '2022-12-08T07:01:13+09:00'
id: 57edede9a7e3ce73cdc7
organization_url_name: null
slide: false
ignorePublish: false
---


この記事は、[Pythonista3 Advent Calendar 2022](https://qiita.com/advent-calendar/2022/pythonista3) の08日目の記事です。

https://qiita.com/advent-calendar/2022/pythonista3

一方的な偏った目線で、Pythonista3 を紹介していきます。

ほぼ毎日iPhone（Pythonista3）で、コーディングをしている者です。よろしくお願いします。

以下、私の2022年12月時点の環境です。

```sysInfo.log
--- SYSTEM INFORMATION ---
* Pythonista 3.3 (330025), Default interpreter 3.6.1
* iOS 16.0.2, model iPhone12,1, resolution (portrait) 828.0 x 1792.0 @ 2.0
```

他の環境(iPad や端末の種類、iOS のバージョン違い)では、意図としない挙動(エラーになる)なる場合もあります。ご了承ください。

ちなみに、`model iPhone12,1` は、iPhone11 です。

## Shader（しぇーだー）？ GLSL（じーえるえすえる）？🤔

[shader | scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html#shader)

https://omz-software.com/pythonista/docs/ios/scene.html#shader

> シェーダーオブジェクトは、カスタムOpenGLフラグメントシェーダーを表し、スプライトノードおよびエフェクトノードオブジェクトのレンダリング動作を（それぞれのシェーダー属性を通じて）変更するために使用することができます。

> シェーダプログラミングは複雑なトピックであり、完全な紹介はこのドキュメントの範囲外ですが、基本的なコンセプトは非常にシンプルです。フラグメントシェーダは本質的に GPU 上で直接実行される関数／プログラムであり、各ピクセルの色値を生成する責任を負っています。シェーダは GLSL（GL Shading Language）で書かれ、これは C 言語に非常によく似ています。

私は、`ui` モジュールの`draw` メソッドや、`scene` での描画（Processing や、JavaScript のcanvas(CanvasRenderingContext2D) のようなもの）では、鉛筆を`x` と`y` 座標に持ってきて線を引く（それを繰り返す）。といった処理のイメージを持っています。

Shader （これ以降は、GLSLと記載）では、事前にGPU で演算をし、結果の全てをドカっと描画する。みたいなイメージで、理解をしています。

「線を引く」ことも、今までとは全く違う書き方をするので、最初はびっくりするかもしれませんが、GPU 演算のパワーで面白い表現ができます。


Pythonista3 Documentation でも「複雑なトピック」とあるように、よちよち歩きの私では簡単に概略を説明することは難しいです。。。

### サンプルを動かしてみる

とにかく、Pythonista3 でどうやってGLSL やるのか体験してみましょう。

[shader | scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html#shader)

https://omz-software.com/pythonista/docs/ios/scene.html#shader


`shader` の項目から下へスクロールすると、サンプルコードがあり`[Open in Editor]` をタップすると、エディタにサンプルコードがコピーされた状態で出てきます。

`▷` をタップすると、コードが実行されます。

![img221125_150211.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/4832ee38-ed62-a6f4-bdff-7d94f2eb9bfa.gif)



画像に波紋のようなエフェクトが出ており、タップするとその場所を中心とした波紋に変わります。

このような処理はGLSL 以外で実装すると、結構処理が重くなってしまう実装の部類になると思われますが、GLSL のおかげでぬるぬる動いてくれています。

### サンプルのGLSL コード

GLSL のコード部分です:

```GLSL
precision highp float;
varying vec2 v_tex_coord;
// These uniforms are set automatically:
uniform sampler2D u_texture;
uniform float u_time;
uniform vec2 u_sprite_size;
// This uniform is set in response to touch events:
uniform vec2 u_offset;

void main(void) {
    vec2 p = -1.0 + 2.0 * v_tex_coord + (u_offset / u_sprite_size * 2.0);
    float len = length(p);
    vec2 uv = v_tex_coord + (p/len) * 1.5 * cos(len*50.0 - u_time*10.0) * 0.03;
    gl_FragColor = texture2D(u_texture,uv);
}
```

GLSL のコードをPython の文字列として、変数`ripple_shader` に代入しています:

```py
ripple_shader = '''
precision highp float;
varying vec2 v_tex_coord;
// These uniforms are set automatically:
uniform sampler2D u_texture;
uniform float u_time;
uniform vec2 u_sprite_size;
// This uniform is set in response to touch events:
uniform vec2 u_offset;

void main(void) {
    vec2 p = -1.0 + 2.0 * v_tex_coord + (u_offset / u_sprite_size * 2.0);
    float len = length(p);
    vec2 uv = v_tex_coord + (p/len) * 1.5 * cos(len*50.0 - u_time*10.0) * 0.03;
    gl_FragColor = texture2D(u_texture,uv);
}
'''
```

画像が入っている`SpriteNode` へ、`.shader` でGLSL コードを反映させることになります:

```py
self.sprite = SpriteNode('test:Pythonista', parent=self)
self.sprite.shader = Shader(ripple_shader)
```

サンプルコードというのは、往々にして複数の要素が混じり合い複雑になっているものなので、シンプルに解体し整理していきます。

## GLSL の`Hello World！`

### サンプルコードを改造する

少々力技ではありますが、先ほど使った波紋のコードをお借りし、ゴリッと書き換えます。

```GLSL
precision highp float;
varying vec2 v_tex_coord;
// These uniforms are set automatically:
uniform sampler2D u_texture;
uniform float u_time;
uniform vec2 u_sprite_size;
// This uniform is set in response to touch events:
uniform vec2 u_offset;

void main(void) {
    vec2 p = -1.0 + 2.0 * v_tex_coord + (u_offset / u_sprite_size * 2.0);
    float len = length(p);
    vec2 uv = v_tex_coord + (p/len) * 1.5 * cos(len*50.0 - u_time*10.0) * 0.03;
    gl_FragColor = texture2D(u_texture,uv);
}
```
↑ のコードを↓ に書き換える。

```glsl
precision highp float;
varying vec2 v_tex_coord;

void main(void) {
    gl_FragColor = vec4(v_tex_coord, 0.0, 1.0);
}

```

余裕がある方は、最初のサンプルコードから何が残って何が削られたか見比べてみましょう。

GLSL の1行コメントアウトは`//` 、`/* 複数行 */` で1行以上の範囲でコメントアウトできます。Python のようにオフサイドルールはありません。インデントは気にしなくて大丈夫です。

`▷` をタップし実行してみましょう。

![img221126_134233.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/f96100df-71c7-0a10-325f-0d56a357386e.png)



この絵が出てきたら成功です。

サンプルコードから、コメントアウトして実行させるならこのようになります:

```glsl
precision highp float;
varying vec2 v_tex_coord;
// These uniforms are set automatically:
//uniform sampler2D u_texture;
//uniform float u_time;
//uniform vec2 u_sprite_size;
// This uniform is set in response to touch events:
//uniform vec2 u_offset;

void main(void) {
    /*
    vec2 p = -1.0 + 2.0 * v_tex_coord + (u_offset / u_sprite_size * 2.0);
    float len = length(p);
    vec2 uv = v_tex_coord + (p/len) * 1.5 * cos(len*50.0 - u_time*10.0) * 0.03;
    gl_FragColor = texture2D(u_texture,uv);
    */
    gl_FragColor = vec4(v_tex_coord, 0.0, 1.0);
}

```

おめでとうございます！！GLSL の第一歩を踏み出せましたね☺️

### `SpriteNode` とGLSL 間で、何が起きているのか？

GLSL の基本に入る前に、Pythonista3 側での動きを整理します。

複数の事象が絡み合っている時には、それぞれ切り分けて考えることが大事ですね！

#### `Scene` と`Node`

`scene` モジュールを実行するための構成要素は、ひとつの`Scene` と複数の`Node` の組み合わせで実装されます。

[scene.Scene | scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html#scene)

https://omz-software.com/pythonista/docs/ios/scene.html#scene


[scene.Node | scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html#node)

https://omz-software.com/pythonista/docs/ios/scene.html#node


外側の`Scene` の中に、さまざまな役割を持った`Node` を`.add_child` し、`update` 時や`touch` イベントにて状態変化をさせていくのです。

```
Scene
  | (.add_child してNode を取り込む)
  ├ Node(並列で存在も可能)
  └ Node
     | (Node がNode を.add_child　も可能)
     └ Node
```

`ui` モジュールの、View に`add_subview` して、画面を構成していく流れと同様。と、今のところは考えていても問題ありません。

#### さまざまな`Node`

`Node` 自体は、基本的な`Node` の機能しか持ち合わせていませんが、他機能を持ち合わせた`Node` として

- SpriteNode
  - テクスチャ（画像）を描画
  - [scene.SpriteNode | scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html#scene.SpriteNode)
- LabelNode
  - 文字を描画
  - [scene.LabelNode | scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html#scene.LabelNode)
- ShapeNode
  - Path を描画
  - [scene.ShapeNode | scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html#scene.ShapeNode)
- EffectNode
  - ポストエフェクトを描画
  - [scene.EffectNode | scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html#scene.EffectNode)

もう少し詳細な継承関係としては

- Node
  - SpriteNode
    - LabelNode
    - ShapeNode
  - EffectNode

今回「GLSL の`shader` について」に注目すると、`.shader('glslコード')` が呼び出せるのは

- SpriteNode
- EffectNode

の、2つの`Node` です。

#### `SpriteNode` で、テクスチャを呼び出しているのに表示されていない？

書き換えたGLSL のコードでは、Pythonista のアイコンを`SpriteNode` で呼び出しているのに、最終的な描画では表示がされていませんでした。

通常の、`scene.Shader` を呼び出さずに実行した場合にはアイコンが表示されます:

```py
self.sprite = SpriteNode('test:Pythonista', parent=self)
# ↓.shader で代入しない
#self.sprite.shader = Shader(ripple_shader)
```

`scene.Shader` を使用することにより、GLSL コード内で最終描画の決定が一任されます。`SpriteNode` は、描画されるはずであった「エリアの範囲」や「サイズ」の情報以外は、GLSL 処理のされるがままとなるのです。


GLSL に処理を託す関係上、`SpriteNode` が持っている情報を渡して、GLSL にいい感じにやってもらうための処理が必要です。

そのための橋渡し処理が:

```glsl
// These uniforms are set automatically:
uniform sampler2D u_texture;
uniform vec2 u_sprite_size;
// This uniform is set in response to touch events:
uniform vec2 u_offset;
```

コードの先頭部分で宣言をし:

```テクスチャ情報を渡す.glsl
gl_FragColor = texture2D(u_texture,uv);
```

渡したテクスチャ情報を、他の処理と混ぜ合わせて描画してね。という流れになります。

```テクスチャ情報を渡していない.glsl
precision highp float;
varying vec2 v_tex_coord;

void main(void) {
    // texture2D(u_texture,uv) を渡していない
    gl_FragColor = vec4(v_tex_coord, 0.0, 1.0);
}

```

`Hello World!` 的なGLSL コードでは、橋渡しの情報を何も記載せずに、`SpriteNode` が保持している描画範囲にGLSL 側で好きに描画指示を出しているだけなので、テクスチャは出ずに、グラデーションがかかった絵が出力されたのです。

#### 「描画される」とかは、まだ考えなくていいです

Pythonista3 側の処理の整理なので、GLSL で何が起こっているか？については、いまは意味不明で問題ありません。

ここでは、`scene` モジュールの処理として

- 実行するための外側の呼び出し
- `SpriteNode` にテクスチャを呼んでいる
- `.shader` で、よしなにGLSL が処理をしている
- 最終的な出力が決まる

の流れと、

- 描画される範囲は`SpriteNode` が持っている

と、いう部分を知っていただければとOKです。

## Pythonista3 でGLSL やろうぜ

GLSL のコードを書いて、Pythonista3（`scene` モジュール）で実行できることがわかりました。

今後もサンプルのコードを使い、書き換える方法でも問題は無いですが、Python の文字列で宣言されており少々編集が面倒です。

```py
# 文字列として、毎回書き換えるのは気が引ける
ripple_shader = '''
precision highp float;
varying vec2 v_tex_coord;
// These uniforms are set automatically:
uniform sampler2D u_texture;
uniform float u_time;
uniform vec2 u_sprite_size;
// This uniform is set in response to touch events:
uniform vec2 u_offset;

void main(void) {
    vec2 p = -1.0 + 2.0 * v_tex_coord + (u_offset / u_sprite_size * 2.0);
    float len = length(p);
    vec2 uv = v_tex_coord + (p/len) * 1.5 * cos(len*50.0 - u_time*10.0) * 0.03;
    gl_FragColor = texture2D(u_texture,uv);
}
'''
```


GLSL のコードは別ファイルで書き、個別管理する方が何かと建設的ですよね。

そこで、[Pythonista3 Advent Calendar 2022](https://qiita.com/advent-calendar/2022/pythonista3) で大人気のEditor Action を使って気軽にGLSL コードを実行できるようにしてみましょう。

### Editor Action にするコード

（Editor Action の設定方法は、過去記事を参照してください）

[Editor Action | Pythonista3 のeditor module を使い、Pythonista3 のコーディングを楽にする - Qiita](https://qiita.com/pome-ta/items/c3902a0f6a0de691df8d#editor-action)

https://qiita.com/pome-ta/items/c3902a0f6a0de691df8d#editor-action


![img221126_190752.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/27a6563b-c91f-ee70-573e-b0f547c20828.gif)

```py
import scene
import editor

shader_code = editor.get_text()


class MyScene(scene.Scene):
  def setup(self):
    self.div = scene.Node()
    self.add_child(self.div)

    self.shdr = scene.SpriteNode(parent=self.div)
    self.shdr.shader = scene.Shader(f'{shader_code}')

    self.set_shader_area()

  def did_change_size(self):
    self.set_shader_area()

  def set_shader_area(self):
    self.div.position = self.size / 2
    _size = min(self.size.x, self.size.y) * .957
    self.shdr.size = (_size, _size)


if __name__ == '__main__':
  main = MyScene()
  scene.run(main, show_fps=True, frame_interval=2)

```

1. GLSL のコードを（Pythonista3 で）開き編集
2. Editor Action で、該当スクリプトを実行
   1. `editor.get_text` で、編集中のコードを取得
   2. `scene.Shader` で、取得したコードを適用
   3. `scene` で実行
3. GLSL のコードが描画される


![img221126_190717.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/1c2acbd8-fb75-736f-6e12-e4a4c1047e8b.gif)


画面サイズから`0.957` 倍のサイズで表示させることにより、端の描画確認をしやすいようにしています。

テクスチャ情報や、touch 情報を渡す`uniform` は設定していないので、独自に使いやすいように追加してみてください。

#### ファイル拡張子を気にせず実行できる

GLSL のコードが「記載」してあるファイルの「内容」を取得しているので、`.py` でも、`.js` でも拡張子を気にせず使用できます（Pythonista3 で認識できない拡張子は、ファイル開く確認を都度されるので、逆に`.glsl` や、`.frag` 等はあまりおすすめできません）。

`.py` であれば、コード上に記載がある内容は一部補完が効きます、`.js` であれば、補完は効きませんが、コメントアウトのシンタックスハイライト等がわかりやすいです。



## よっしゃ！GLSL 入門や！（（ここでは）しません）

ものすごく記事が長くなり、内容の自信もないので、私が参照している先を紹介していきます。

- [The Book of Shaders](https://thebookofshaders.com/?lan=jp)
  - GLSL をしている方なら誰でも通る道（サイト）かもしれません
  - 原文が英語ですが、有志の方々により日本語翻訳があります
  - [The Book of Shaders: What is a shader?](https://thebookofshaders.com/01/?lan=jp)
    - GPU のイラスト図解がわかりやすくて好き

https://thebookofshaders.com/?lan=jp

- [[連載]やってみれば超簡単！ WebGL と GLSL で始める、はじめてのシェーダコーディング（１） - Qiita](https://qiita.com/doxas/items/b8221e92a2bfdc6fc211)
  - 日本WebGL の父doxas さんの連載記事
  - WebGL ？あ、アタラシイ、コトバ、、タンゴ、、、と混乱させてすみません。いまは、気にせずで大丈夫（な、はず）

https://qiita.com/doxas/items/b8221e92a2bfdc6fc211

このあたりを主軸にしつつ、独自に調べたことにチャレンジしてみるのがおすすめです。

### 注意点

Pythonista3 でGLSL をチャレンジしてみる際に、数点読み替えが必要な箇所があります。

#### `uniform` 変数

- `u_time`
  - `iTime` 等で表記されている可能性あり
  - [The Book of Shaders: uniforms](https://thebookofshaders.com/03/?lan=jp)
- `u_resolution`
  - （ざっくり言うと）`scene.Shader` では、存在していません
  - `v_tex_coord` でいい感じになんとかします

https://thebookofshaders.com/03/?lan=jp

#### 正規化

GLSL コードで（当たり前のように）使われる処理として正規化があります。

```glsl
vec2 p = (gl_FragCoord.xy * 2.0 - u_resolution) / min(u_resolution.x, u_resolution.y);
```

スクリーンの中心位置を原点にする処理です。

[[連載]やってみれば超簡単！ WebGL と GLSL で始める、はじめてのシェーダコーディング（４） - Qiita](https://qiita.com/doxas/items/f3f8bf868f12851ea143)

https://qiita.com/doxas/items/f3f8bf868f12851ea143

[[連載]やってみれば超簡単！ WebGL と GLSL で始める、はじめてのシェーダコーディング（５） - Qiita](https://qiita.com/doxas/items/40898f7ec279b8ab5358)

https://qiita.com/doxas/items/40898f7ec279b8ab5358

動くオーブのサンプルコードをお借りし、Pythonista3 で適用するとなると:

```glsl
precision highp float;

uniform float u_time;
varying vec2 v_tex_coord;


void main(){
  vec2 p = (v_tex_coord - vec2(0.5)) * 2.0;
  
  p += vec2(cos(u_time * 5.0), sin(u_time)) * 0.5;
  float l = 0.1 / length(p);
  
  gl_FragColor = vec4(vec3(l), 1.0);
}

```


![img221126_200639.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/b0fe8694-71c7-26de-f175-9a1ea156c47d.gif)



正方形だからこそ使える力技で、なんとかする方法もあります。

#### `SpriteKit` でググる

Pythonista3 の`scene` モジュールでは、解決策を検索してもなかなか見つかりません。方法として一番可能性が高いのが`SpriteKit` をキーワードに追加して検索をしてみることです。

`scene` モジュールは、iOS, iPadOS のSpriteKit framework がベース（ラップ）となっています。案外`SpriteKit` で調べた方が解決が早い場合があります（情報が突然増え混乱する可能性もありますが）。

## 次回は

Pythonista3 でGLSL が実行できるんです。という、紹介のみでGLSL 自体が疎かになってしまいました。

GLSL 奥が深いので、興味が湧いた方はぜひ調べてみてください！「は？意味わからん」の感動や疑問や恐怖が入り混じる素敵な世界です。

ここ最近は、GLSL が実行できる環境も簡単になってきました。が、まだまだ事前準備が必要な手順も多いです。

[The Book of Shaders: Running your shader](https://thebookofshaders.com/04/?lan=jp)

https://thebookofshaders.com/04/?lan=jp

準備で迷子になってしまい、`Hello World!` まで辿り着けないこともあります。

また、他の簡単な実行環境はブラウザがメインであり。PC での体験は最高なのですが、モバイルとなるとコード編集の煩わしさを強く感じます。

そういった観点でみると、Pythonista3 の`scene` を使ったGLSL 実行環境は比較的カジュアルに実装できるのではないかと感じています。

決してモバイルオンリーでGLSL を完結させたい訳ではないです。ふとした時に「この処理だとどうなる？」の思いつきに、素早く編集ができ実行ができる環境が素敵だなと思っているだけです。

次回は、今回使用した`scene` モジュールと、`ui` モジュールの合わせ技で、いい感じにアプリっぽくする方法を紹介します。


ここまで、読んでいただきありがとうございました。

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

コードのエラーや変なところや改善点など。ご指摘やPR お待ちしておりますー

https://github.com/pome-ta/Pythonista3AdventCalendar2022sampleCode

- Twitter

なんしかガチャガチャしていますが、お気兼ねなくお声がけくださいませー

https://twitter.com/pome_ta93

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">やれるか、やれないか。ではなく、やるんだけども、紹介説明することは尽きないと思うけど、締め切り守れるか？って話よ！（クズ）<br><br>Pythonista3 Advent Calendar 2022 <a href="https://t.co/JKUxA525Pt">https://t.co/JKUxA525Pt</a> <a href="https://twitter.com/hashtag/Qiita?src=hash&amp;ref_src=twsrc%5Etfw">#Qiita</a></p>&mdash; pome-ta (@pome_ta93) <a href="https://twitter.com/pome_ta93/status/1588570697777152006?ref_src=twsrc%5Etfw">November 4, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- GitHub

基本的にGitHub にコードをあげているので、何にハマって何を実装しているのか観測できると思います。

https://github.com/pome-ta

