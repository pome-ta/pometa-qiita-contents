---
title: Pythonista3 で3DCG やろうぜ！SceneKit のプリミティブなもので遊ぶ
tags:
  - Python
  - Mobile
  - SceneKit
  - MobileApp
  - Pythonista3
private: false
updated_at: '2026-01-09T23:28:15+09:00'
id: 551bf5fb2448ddcacae0
organization_url_name: null
slide: false
ignorePublish: false
---
この記事は、[Pythonista3 Advent Calendar 2022](https://qiita.com/advent-calendar/2022/pythonista3) の17日目の記事です。

https://qiita.com/advent-calendar/2022/pythonista3

一方的な偏った目線で、Pythonista3 を紹介していきます。

ほぼ毎日iPhone（Pythonista3）で、コーディングをしている者です。よろしくお願いします。

以下、私の2022年12月時点の環境です。

```sysInfo.log
--- SYSTEM INFORMATION ---
* Pythonista 3.3 (330025), Default interpreter 3.6.1
* iOS 16.1.1, model iPhone12,1, resolution (portrait) 828.0 x 1792.0 @ 2.0
```

他の環境(iPad や端末の種類、iOS のバージョン違い)では、意図としない挙動(エラーになる)なる場合もあります。ご了承ください。

ちなみに、`model iPhone12,1` は、iPhone11 です。

## この記事でわかること

- Pythonista3 での基礎的なsceneKit の使い方
  - 球体やボックスを出す
  - カメラを操作する
  - 物理演算を使う
  - debugOptions を使う
- Swift -> Objective-C -> `objc_util` モジュールへの書き換え



![img221208_175438.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/8f8fd12f-9d9d-4b5c-8d01-da5b1b17f483.gif)



![img221209_004533.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/ce07bb49-b315-22f2-f3cd-1a3ae815e212.gif)



## Pythonista3 の`scene` モジュールじゃないの。SceneKit Framework なの

混同してしまいますが（過去の私だけ？）、2D アニメーションやゲームに特化した、[scene — 2D Games and Animations — Python 3.6.1 documentation](https://omz-software.com/pythonista/docs/ios/scene.html) モジュールではなく。

https://omz-software.com/pythonista/docs/ios/scene.html

3DCG のFramework である、[SceneKit | Apple Developer Documentation](https://developer.apple.com/documentation/scenekit?language=objc) を`objc_util` モジュールで呼び出してPythonista3 で遊んでいきます。

https://developer.apple.com/documentation/scenekit?language=objc

Qiita に他の方が書かれた記事もあります。

[PythonistaでSceneKitを使った3D描画 - Qiita](https://qiita.com/yohei_takada201/items/af1cca8d96a8b49097c5)

https://qiita.com/yohei_takada201/items/af1cca8d96a8b49097c5

[pythonista3で物理シミュレーション - Qiita](https://qiita.com/runomee/items/b22e5bd1a95eb0ab1dab)

https://qiita.com/runomee/items/b22e5bd1a95eb0ab1dab

SceneKit（Swift やObjective-C で書かれた）コードから、Pythonista3 へ実装できるよう進めていきます。

当たり前ですが、Pythonista3 のSceneKit 実装のコードよりもSwift やObjective-C で書かれたSceneKit のサンプルの方が多いので。

前回までの`AVAudioEngine` は地獄でしたが、今回は比較的簡単だと思います！

### 基本的な考え方

[SCNScene | Apple Developer Documentation](https://developer.apple.com/documentation/scenekit/scnscene?language=objc)

https://developer.apple.com/documentation/scenekit/scnscene?language=objc

Overview のイラストにあるように、描画するためのView（[SCNView | Apple Developer Documentation](https://developer.apple.com/documentation/scenekit/scnview?language=objc)）があり、3DCG 世界の土台としてのScene（[SCNScene | Apple Developer Documentation](https://developer.apple.com/documentation/scenekit/scnscene?language=objc)）があります。

我々は、View を通して3DCG の世界を覗かせて頂いています。

Scene には`rootNode` というものが生えています。我々が3DCG 世界に登場させたいモノは、`Node` というカタチに納めて`rootNode` に取り込んでもらいます（`addChildNode`）。

```イメージ
SCNView → UIView にaddSubview することで見れる
  |
SCNScene
  |
rootNode
↑ ↑ ↑ ↑ addChildNode(Node)
さまざまなNode たち

```

Node（[SCNNode | Apple Developer Documentation](https://developer.apple.com/documentation/scenekit/scnnode?language=objc)）は、さまざまなモノを取り付けられます。

https://developer.apple.com/documentation/scenekit/scnnode?language=objc


- ジオメトリ（3D の物体オブジェクト、メッシュ）
- 色情報や質感
- ライト
- カメラ

またNode 自体では

- （3DCG内の）位置
- 回転
- スケール

の情報を持ち操作をしていきます。

Node に出したいモノや使いたいモノを付けて、位置や大きさなどを決め、`rootNode` へ`addChildNode` することで、3DCG 世界に登場させることができるのです。

UIView の`addSubview` で他のView を取り込んで画面を構築していく親子関係と、大きくは変わりません。

ちなみに、SceneKit は右手座標系です。OpenGL, Vulkan と同じ座標系です。

ちなみにちなみに、Metal は、左手系なのでDirectX と同じですね。

### 参照先

[Apple Engine](https://appleengine.hatenablog.com/)

https://appleengine.hatenablog.com/


こちらの`SceneKit` カテゴリに大変お世話になっています。

[SceneKit カテゴリーの記事一覧 - Apple Engine](https://appleengine.hatenablog.com/archive/category/SceneKit)

https://appleengine.hatenablog.com/archive/category/SceneKit


しかし[更新終了のお知らせ - Apple Engine](https://appleengine.hatenablog.com/entry/2020/12/14/190949)と、あるように今後消えてしまう可能性もあるので、読めるうちに読んでおきましょう。

https://appleengine.hatenablog.com/entry/2020/12/14/190949

今回は「iOS で SceneKit を試す(Swift 3) 」シリーズ参考に進めていきます。

その1 〜その3 の概要は、先ほどの私の説明よに数億倍わかりやすいのでおすすめです。

まずは、[iOS で SceneKit を試す(Swift 3) その4 - SceneKit の構造 - Apple Engine](https://appleengine.hatenablog.com/entry/2017/05/30/192435) より始めていきます。都度対応するパートも提示するので、Swift のコード等は、リンク先を参照ください。

https://appleengine.hatenablog.com/entry/2017/05/30/192435

## Pythonista3 に実装

### 土台つくり

Pythonista3 の`ui.View` にSceneKit を出せるようにしましょう。

その3、その4を参考にしています。

[iOS で SceneKit を試す(Swift 3) その4 - SceneKit の構造 - Apple Engine](https://appleengine.hatenablog.com/entry/2017/05/30/192435)

https://appleengine.hatenablog.com/entry/2017/05/30/192435

[iOS で SceneKit を試す(Swift 3) その5 - シーンエディタを使用しない空のテンプレートをつくる - Apple Engine](https://appleengine.hatenablog.com/entry/2017/06/01/124340)

https://appleengine.hatenablog.com/entry/2017/06/01/124340


![img221207_202303.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/3f366807-97b6-eb82-d6ec-a13c6fde66cd.gif)




```py
from objc_util import load_framework, ObjCClass, on_main_thread
from objc_util import UIColor
import ui

load_framework('SceneKit')

SCNScene = ObjCClass('SCNScene')
SCNView = ObjCClass('SCNView')


class GameScene:
  def __init__(self):
    self.scene: SCNScene
    self.setUpScene()

  def setUpScene(self):
    scene = SCNScene.scene()
    # ---
    # ここに処理を書いていく
    # ---
    self.scene = scene


class View(ui.View):
  def __init__(self, *args, **kwargs):
    ui.View.__init__(self, *args, **kwargs)
    self.name = '土台作り'
    self.bg_color = 'maroon'

    self.scene: GameScene
    self.scnView: SCNView

    self.viewDidLoad()
    self.objc_instance.addSubview_(self.scnView)

  #@on_main_thread
  def viewDidLoad(self):
    scene = GameScene()

    # --- SCNView
    _frame = ((0, 0), (100, 100))
    scnView = SCNView.alloc().initWithFrame_(_frame)
    # ui.View.flex = 'WH' と同じ
    scnView.setAutoresizingMask_((1 << 1) | (1 << 4))
    scnView.backgroundColor = UIColor.blackColor()

    scnView.showsStatistics = True
    scnView.autorelease()

    scnView.scene = scene.scene
    self.scene = scene
    self.scnView = scnView

  def touch_began(self, touch):
    pass


if __name__ == '__main__':
  view = View()
  view.present(style='fullscreen', orientations=['portrait'])

```

ようこそ！SceneKit の世界へ！

デバッグ情報を出すのになかなかコツがいるのですが、デバッグ情報のバーの左側に`+` のアイコンがあり、その近辺をポチポチしてると出てきます。。。


![img221207_203225.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/6db6d13a-ff86-9ecf-26fc-c9950a039a25.png)



#### `ui.View` と`SCNView` の繋ぎ合わせ

Pythonista3 の世界に`objc_util` で呼び出したSceneKit を繋げるために、`SCNView` を介しています。

`ui` モジュールのView と`objc_util` のView は、気軽には繋げません。

```py
ui.View.objc_instance.addSubview_(scnView)
```

`ui.View` 側で、`objc_util` のView として立ち回れる`objc_instance` として、Objective-C の`UIView` のメソッドの`addSubview_` を使い`SCNView` を取り込んでいます。

#### `SCNView` の事前準備

View なので、サイズ等を指定しなければならないのですが、`ui.View` よりも少し手間ですが、設定します。

```py
SCNView = ObjCClass('SCNView')

_frame = ((0, 0), (100, 100))
scnView = SCNView.alloc().initWithFrame_(_frame)
# ui.View.flex = 'WH' と同じ
scnView.setAutoresizingMask_((1 << 1) | (1 << 4))
```

`frame` を`((位置x: 0, 位置y: 0), (横幅: 100, 縦幅: 100))` として、仮で決めています。

次の`setAutoresizingMask_` が肝ですが、`ui.View` の`flex` のように画面設定をしています。`WH` 幅と高さを最大。ということですね。

`(1 << 1) | (1 << 4)` は、整数値で`18` です。直接`18` と入れても機能します。

面白いですね。

この準備を終えたら、`ui.View` のView に`addSubview_` してもらい、晴れてPythonista3 で描画されます。

ここでしれっと、背景を黒色にしています。

```py
scnView.backgroundColor = UIColor.blackColor()
```

#### デコレータ`@on_main_thread`

`ui` の処理にブロックされずに、デコレータの部分も処理をしてくれます。

実行時に`ui.View` で設定した赤の背景が見えてから`SCNView` が表示されていました。

![img221207_210945.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/755c44ba-aa49-efbe-c593-63c0d5120404.png)



上記のコードではコメントアウトしています。

コメントアウトを外し実行すると、赤背景は出ずに、立ち上がり直後`SCNView` の黒画面が表示されるのが確認できます。

アプリとしての格好はいいのですが、`on_main_thread` を使うことでエラー箇所が見つかり辛いこともあります。

今回はその程度ですが、他の事例では「`ui.View` であれして、`objc_util` でこれして、、、」とジャグリング状態になり、必須の場面も出てきます。

#### class `GameScene`

往々にしてSceneKit は、class 内がfat になりがちです。

View の処理と、Node の処理を意識的に分ける意味合いで`GameScene` class として宣言しています。

View 側で、`scene` を触りたい場面には、`self` を生やしていく方針です。

### 箱を出して色付け、ライト設置で影も付けるし、カメラ登場の3D 空間ぐりんぐりんする

何もない3D 空間から、ドカっと登場させます。

- 赤色`ambient`

![img221208_175016.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/b432535e-d1b9-7682-faed-8c72f83246fc.gif)



- boxに青色

![img221208_175438.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/40aa44f4-691a-8b03-5912-ec8805659447.gif)


```py
from objc_util import load_framework, ObjCClass, on_main_thread
from objc_util import UIColor
import ui

import pdbg

load_framework('SceneKit')

SCNScene = ObjCClass('SCNScene')
SCNView = ObjCClass('SCNView')
SCNNode = ObjCClass('SCNNode')

SCNLight = ObjCClass('SCNLight')
SCNCamera = ObjCClass('SCNCamera')

SCNAction = ObjCClass('SCNAction')

SCNBox = ObjCClass('SCNBox')


class GameScene:
  def __init__(self):
    self.scene: SCNScene
    self.setUpScene()

  def setUpScene(self):
    scene = SCNScene.scene()
    # 呼び出しが面倒なので、変数化
    scene_rootNode_addChildNode_ = scene.rootNode().addChildNode_

    box = SCNBox.boxWithWidth_height_length_chamferRadius_(2, 2, 2, 0.2)
    #box.firstMaterial().diffuse().contents = UIColor.blueColor()
    geometryNode = SCNNode.nodeWithGeometry_(box)
    geometryNode.runAction_(
      SCNAction.repeatActionForever_(
        SCNAction.rotateByX_y_z_duration_(0.0, 0.2, 0.1, 0.3)))
    scene_rootNode_addChildNode_(geometryNode)

    # --- SCNLight
    lightNode = SCNNode.node()
    lightNode.light = SCNLight.light()
    lightNode.position = (0.0, 10.0, 10.0)
    scene_rootNode_addChildNode_(lightNode)

    ambientLightNode = SCNNode.node()
    ambientLightNode.light = SCNLight.light()
    ambientLightNode.light().type = 'ambient'
    ambientLightNode.light().color = UIColor.redColor()
    #ambientLightNode.light().color = UIColor.darkGrayColor()
    scene_rootNode_addChildNode_(ambientLightNode)

    # --- SCNCamera
    cameraNode = SCNNode.node()
    cameraNode.camera = SCNCamera.camera()
    cameraNode.position = (0.0, 0.0, 10.0)
    scene_rootNode_addChildNode_(cameraNode)

    self.scene = scene


class View(ui.View):
  def __init__(self, *args, **kwargs):
    ui.View.__init__(self, *args, **kwargs)
    self.name = ''
    self.bg_color = 'maroon'

    self.scene: GameScene
    self.scnView: SCNView

    self.viewDidLoad()
    self.objc_instance.addSubview_(self.scnView)

  #@on_main_thread
  def viewDidLoad(self):
    scene = GameScene()

    # --- SCNView
    _frame = ((0, 0), (100, 100))
    scnView = SCNView.alloc().initWithFrame_(_frame)
    # ui.View.flex = 'WH' と同じ
    scnView.setAutoresizingMask_((1 << 1) | (1 << 4))
    scnView.backgroundColor = UIColor.blackColor()

    scnView.allowsCameraControl = True
    scnView.showsStatistics = True
    scnView.autorelease()

    scnView.scene = scene.scene
    self.scene = scene
    self.scnView = scnView

  def touch_began(self, touch):
    pass


if __name__ == '__main__':
  view = View()
  view.present(style='fullscreen', orientations=['portrait'])

```

引き続きその3、その4を参考にしています。

#### class `GameScene` の部分

`setUpScene` 内で、たくさん登場させています。

上から見ていきましょう。

基本的にオブジェクトは、`alloc.init` （`new`）せずに呼び出せるのが特徴です。

##### `scene.rootNode().addChildNode_` の呼び出し

生成した、Node たちを3DCG 世界（`scene`）に登場させる、クサビ的な立ち位置です。

Node は、`rootNode` に全集合させます。

毎回、`scene.rootNode().addChildNode_` と入力するのが面倒なので、`scene_rootNode_addChildNode_` と変数化しています。

あまり望ましい方法ではありませんが、`scene.rootNode()` と`.` を繋ぐ場合でもインスタンス化しなければならず、その点が（私的に）罠だったりするので変数化しています。

気持ち悪い場合には、素直に:

```py
scene.rootNode().addChildNode_(Node)
```

で良いと思います。

##### `SCNBox`

`SCNBox.boxWithWidth_height_length_chamferRadius_` にて、サイズと角丸を指定した`Geometry` を生成しています。

色を付けるには、`firstMaterial().diffuse().contents` より`UIColor` を使います。

マテリアルの細かい質感は、[iOS で SceneKit を試す(Swift 3) その33 - ジオメトリの質感を決めるマテリアルについて - Apple Engine](https://appleengine.hatenablog.com/entry/2017/07/19/161307) こちらにて、説明があります。

https://appleengine.hatenablog.com/entry/2017/07/19/161307


`rootNode` に取り込んでもらうために、`SCNNode.nodeWithGeometry_(box)` として、ジオメトリの`box` を`SCNNode` に格納します。

`SCNAction` で、Node にアニメーション設定をして、今回は常にくるくると回ってもらうことにしています。

[iOS で SceneKit を試す(Swift 3) その4 - SceneKit の構造 - Apple Engine](https://appleengine.hatenablog.com/entry/2017/05/30/192435) では、地球のテクスチャを貼り付けていますが、外部データ読み込みは次回説明予定なので、今回は、球体ではなくBox に回ってもらうことにしています。

https://appleengine.hatenablog.com/entry/2017/05/30/192435


##### `SCNLight`, `SCNCamera`

ジオメトリ生成とほぼ同様です。最終的に`SCNNode` へNode として、存在していないと`rootNode` へ`addChildNode_` できない点を忘れずに意識します。

Swift コードですと:

```swift
let cameraNode = SCNNode()
cameraNode.camera = SCNCamera()
```

と、class 直接の呼び出しです。

`objc_util`（Objective-C）ですと、`ObjCClass` からメソッドを呼び出す一手間が必要です:

```py
cameraNode = SCNNode.node()
cameraNode.camera = SCNCamera.camera()
```

[camera | Apple Developer Documentation](https://developer.apple.com/documentation/scenekit/scncamera/1436602-camera?language=objc)

https://developer.apple.com/documentation/scenekit/scncamera/1436602-camera?language=objc

Documentation や、Pythonista3 上での`print` デバッグなどをして確認します。

#### 視点を動かす

`SCNView` に、`.allowsCameraControl = True` とすることで画面を動かしたり、ピンチインアウトで拡大縮小ができます:

```py
scnView.allowsCameraControl = True
```

ダブルタップすると、カメラの位置に戻ります。

設置した`SCNCamera` を動かしているのではなく、カメラから「幽体離脱」的に抜け出して傍観者モードのようになるみたいです。

### この世界に重力を導入することとする

その9とその10を参考にしながら、物理演算を設定しボールを落下させたり衝突させたりしましょう。

[iOS で SceneKit を試す(Swift 3) その9 - 物理アニメーションを試す - Apple Engine](https://appleengine.hatenablog.com/entry/2017/06/05/194211)

https://appleengine.hatenablog.com/entry/2017/06/05/194211

[iOS で SceneKit を試す(Swift 3) その10 - ノードをコピーして端末負荷を下げる - Apple Engine](https://appleengine.hatenablog.com/entry/2017/06/13/194707)

https://appleengine.hatenablog.com/entry/2017/06/13/194707


![img221209_004533.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/5c849d31-33f5-bb74-9b15-5fd72bb98828.gif)


```py
from objc_util import load_framework, ObjCClass, on_main_thread
from objc_util import UIColor
import ui

import pdbg

load_framework('SceneKit')

SCNScene = ObjCClass('SCNScene')
SCNView = ObjCClass('SCNView')
SCNNode = ObjCClass('SCNNode')

SCNLight = ObjCClass('SCNLight')
SCNCamera = ObjCClass('SCNCamera')

SCNAction = ObjCClass('SCNAction')
'''
Static = 0
Dynamic = 1
Kinematic = 2
'''
SCNPhysicsBody = ObjCClass('SCNPhysicsBody')
SCNPhysicsShape = ObjCClass('SCNPhysicsShape')

SCNSphere = ObjCClass('SCNSphere')
SCNFloor = ObjCClass('SCNFloor')


class GameScene:
  def __init__(self):
    self.scene: SCNScene
    self.setUpScene()

  def setUpScene(self):
    scene = SCNScene.scene()
    # 呼び出しが面倒なので、変数化
    scene_rootNode_addChildNode_ = scene.rootNode().addChildNode_

    # --- SCNFloor
    floor = SCNFloor.floor()
    floorNode = SCNNode.nodeWithGeometry_(floor)
    floorNode.position = (0.0, -4.0, 0.0)
    floorNode.eulerAngles = (-0.001, 0.0, 0.0)
    floorNode.physicsBody = SCNPhysicsBody.bodyWithType_shape_(0, None)
    scene_rootNode_addChildNode_(floorNode)

    # --- SCNSphere
    ball = SCNSphere.sphereWithRadius_(0.5)
    ballNode = SCNNode.nodeWithGeometry_(ball)
    ballNode.position.y = 2

    physicsBall = SCNPhysicsShape.shapeWithGeometry_options_(ball, None)
    #physicsBall = SCNPhysicsShape.shapeWithNode_options_(ballNode, None)

    ballNode.physicsBody = SCNPhysicsBody.bodyWithType_shape_(1, physicsBall)
    scene_rootNode_addChildNode_(ballNode)

    # --- SCNLight
    lightNode = SCNNode.node()
    lightNode.light = SCNLight.light()

    lightNode.position = (0.0, 10.0, 10.0)
    scene_rootNode_addChildNode_(lightNode)

    # --- SCNCamera
    cameraNode = SCNNode.node()
    cameraNode.camera = SCNCamera.camera()
    cameraNode.position = (0.0, 0.0, 10.0)
    scene_rootNode_addChildNode_(cameraNode)

    self.scene = scene
    self.scene_rootNode_addChildNode_ = scene_rootNode_addChildNode_
    self.ballNode = ballNode


class View(ui.View):
  def __init__(self, *args, **kwargs):
    ui.View.__init__(self, *args, **kwargs)
    self.name = ''
    self.bg_color = 'maroon'

    self.scene: GameScene
    self.scnView: SCNView

    self.viewDidLoad()
    self.objc_instance.addSubview_(self.scnView)

  #@on_main_thread
  def viewDidLoad(self):
    scene = GameScene()

    # --- SCNView
    _frame = ((0, 0), (100, 100))
    scnView = SCNView.alloc().initWithFrame_(_frame)
    # ui.View.flex = 'WH' と同じ
    scnView.setAutoresizingMask_((1 << 1) | (1 << 4))
    scnView.backgroundColor = UIColor.blackColor()

    scnView.allowsCameraControl = True
    scnView.showsStatistics = True
    '''
    OptionNone = 0
    ShowPhysicsShapes = (1 << 0)
    ShowBoundingBoxes = (1 << 1)
    ShowLightInfluences = (1 << 2)
    ShowLightExtents = (1 << 3)
    ShowPhysicsFields = (1 << 4)
    ShowWireframe = (1 << 5)
    RenderAsWireframe = (1 << 6)
    ShowSkeletons = (1 << 7)
    ShowCreases = (1 << 8)
    ShowConstraints = (1 << 9)
    ShowCameras = (1 << 10)
    '''
    _debugOptions = ((1 << 0) | (1 << 1) | (1 << 4) | (1 << 10))
    scnView.debugOptions = _debugOptions

    scnView.autorelease()

    scnView.scene = scene.scene
    self.scene = scene
    self.scnView = scnView

  def touch_began(self, touch):
    ballNode = self.scene.ballNode.clone()
    self.scene.scene_rootNode_addChildNode_(ballNode)


if __name__ == '__main__':
  view = View()
  view.present(style='fullscreen', orientations=['portrait'])

```

床に少し傾斜をつけています、微妙に角度があればよかったので適当に設定しています。

`physicsBall` に対し`Geometry` か`Node` どちらがいいかわらず、現在検証中です。

また、その10の`clone` が、多分効いていない状態だと思われます。こちらも調査中です。。。

#### デバッグオブションを設定して3DCG 世界をもっと深く見る

個人的にはテンションあがるやつです。

上がる下がるの問題ではなく、ジオメトリのワイヤーやライトの位置。物理計算の範囲などの情報を可視化してくれます。

[SCNDebugOptions | Apple Developer Documentation](https://developer.apple.com/documentation/scenekit/scndebugoptions?language=objc)

https://developer.apple.com/documentation/scenekit/scndebugoptions?language=objc

View の時の`setAutoresizingMask` に似た呼び出し方です。

```py
_debugOptions = ((1 << 0) | (1 << 1) | (1 << 4) | (1 << 5) | (1 << 6) | (1 << 10))
scnView.debugOptions = _debugOptions
```


![img221209_005508.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2953777/d9d10c2b-d4fe-30b7-6230-589e848eedd0.png)



ワイヤー表示っていいですよね。

## 次回は

Pythonista3 で SceneKit Framework を呼び出し、3DCG 世界を体験してみました。

案外、サンプルコードの読み替えで実装できることを知っていただけましたら嬉しいです。

SceneKit と`objc_util` の関係性の理解が深まったら、以下リポジトリのコードを読んでみるのもおすすめです。

[pulbrich/sceneKit-wrapper-for-Pythonista: SceneKit module access from Pythonista, pure python package using objc_util](https://github.com/pulbrich/sceneKit-wrapper-for-Pythonista)

https://github.com/pulbrich/sceneKit-wrapper-for-Pythonista



今回の我々のように、生な`objc_util` を使うのではなく、Wrapper としてPythonista3 で気軽にSceneKit が使えるように作成した方がいらっしゃいます。 なんという情熱なんでしょう。。。

Swift やObjective-C で書かれたサンプル実装で困った時に、該当の実装内容を見にいくとヒントがあったりして勉強になります。

次回は、もっと素敵な絵を出したいので、外部からデータを持ってきてSceneKit 上に登場させたりしたいと思います。

絵力がグッと上がると思いますよー。

ここまで、読んでいただきありがとうございました。

https://qiita.com/pome-ta/items/843ecbff44d4bdc07fd0


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
