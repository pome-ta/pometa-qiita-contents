---
title: Pythonista3 で3DCG やろうぜ！外部のデータをSceneKit へ呼び出したりして、絵力をつける
tags:
  - Python
  - Mobile
  - SceneKit
  - MobileApp
  - Pythonista3
private: false
updated_at: '2026-01-09T23:28:16+09:00'
id: 843ecbff44d4bdc07fd0
organization_url_name: null
slide: false
ignorePublish: false
---
この記事は、[Pythonista3 Advent Calendar 2022](https://qiita.com/advent-calendar/2022/pythonista3) の18日目の記事です。

https://qiita.com/advent-calendar/2022/pythonista3


前回👇

https://qiita.com/pome-ta/items/551bf5fb2448ddcacae0


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

- 画像データや、シーンデータなど、外部データの取り込み方
- 取り込み後の、SceneKit 上での操作方法
- さまざまな効果を使い、出力結果をにぎやかにする


![img221209_221509](https://user-images.githubusercontent.com/53405097/207567071-250b5b18-34ca-426f-a5e5-64a274f37d43.gif)



![img221210_004822](https://user-images.githubusercontent.com/53405097/207567369-40b70ca7-3929-49e2-bea2-1dca748338ce.gif)


## サンプルデータ

前回は、SceneKit のみでScene を作りました。

プリミティブな球体やボックスなど、標準的に用意されているもの以外ももちろん、3DCG の世界へ呼び出すことができます。

他のデータを取り込むことにより、より豊かな3DCG 生活を楽しめるわけです。

検索をすれば、いろいろなデータを見つけることができますが、今回はApple が公開しているデータを使ってみましょう。

[Documentation Archive](https://developer.apple.com/library/archive/navigation/#section=Technologies&topic=SceneKit)

https://developer.apple.com/library/archive/navigation/#section=Technologies&topic=SceneKit

iPhone だとなかなか見辛いレイアウトなんですけどね。。。

### データダウンロード先

- [Fox 2: SceneKit WWDC 2017 sample code](https://developer.apple.com/library/archive/samplecode/scenekit-2017/Introduction/Intro.html#//apple_ref/doc/uid/TP40017656)
  - 今回はこちらをメインに使っていきます
  - Max ちゃんかわいい
  - 75.2MB ありました

https://developer.apple.com/library/archive/samplecode/scenekit-2017/Introduction/Intro.html#//apple_ref/doc/uid/TP40017656

他のおすすめな、データたちです。

- [SceneKit State of the Union Demo](https://developer.apple.com/library/archive/samplecode/SceneKitReel/Introduction/Intro.html#//apple_ref/doc/uid/TP40014550)
  - SceneKit でよくみる、飛行機（？）のデータがある

- [SceneKit slides for WWDC 2014](https://developer.apple.com/library/archive/samplecode/SceneKitWWDC2014/Introduction/Intro.html#//apple_ref/doc/uid/TP40014551)
  - 地球のテクスチャ
  - 3D でよくみるティーポットみたいなの
  - データのサンプル数が多い

### iPhone に取り込み、Pythonista3 に取り込む

データはPC からiPhone に移しても問題ありませんが、iPhone のみで完結も可能です。

[Fox 2: SceneKit WWDC 2017 sample code](https://developer.apple.com/library/archive/samplecode/scenekit-2017/Introduction/Intro.html#//apple_ref/doc/uid/TP40017656)

![img221209_220619](https://user-images.githubusercontent.com/53405097/207567602-c93198a2-9f13-4e4c-a24d-21bd03634b4e.png)




https://developer.apple.com/library/archive/samplecode/scenekit-2017/Introduction/Intro.html#//apple_ref/doc/uid/TP40017656

1. `Download Sample Code` をタップして`.zip` をダウンロード
   - ダウンロードフォルダで問題ありません

![img221209_220638](https://user-images.githubusercontent.com/53405097/207567864-3fd6e1d0-83f9-43f2-b205-2b55fb1e44f4.png)


- `(↓)` アイコンでダウンロード完了がわかります

![img221209_220648](https://user-images.githubusercontent.com/53405097/207568017-7cdca818-f9b9-4f83-a1fb-05081f1f1ba0.png)



![img221209_220705](https://user-images.githubusercontent.com/53405097/207568118-bdb90138-195f-43fb-bd37-88296f88e9b4.png)


1. `ファイルApp` で解凍

   - `.zip` をタップすると解凍が始まります

![img221209_220718](https://user-images.githubusercontent.com/53405097/207568261-e66c3936-c245-41cc-bbf4-ebbcea1f7975.png)




![img221209_220910](https://user-images.githubusercontent.com/53405097/207568339-7be82d8d-ea82-4635-bd3c-68b8fccd1470.png)



1. `ファイルApp` から、Pythonista3 へデータ移動

   - `選択` で解凍したファイルを選ぶ

![img221209_220918](https://user-images.githubusercontent.com/53405097/207568443-4dd2a370-3e20-4ca7-92be-3b1b93377906.png)

![img221209_220947](https://user-images.githubusercontent.com/53405097/207568528-7991fdef-4935-4790-9e9f-c78b015320b5.png)


- 共有アクションで`Run Pythonista Script` を選択

![img221209_220957](https://user-images.githubusercontent.com/53405097/207568670-d0e3595d-23ee-4ee9-8c5e-31862126802d.png)



![img221209_221004](https://user-images.githubusercontent.com/53405097/207568758-34b92400-ae90-42c1-a734-3c1fe45564be.png)



- `Import File` で、Pythonista3 に取り込まれます

![img221209_221011](https://user-images.githubusercontent.com/53405097/207568851-135faa5f-7e0f-4336-884c-87e244dffc28.png)



![img221209_221017](https://user-images.githubusercontent.com/53405097/207568919-ddbcccf0-4272-47ce-93b6-8a34d44fcfaf.png)


### サンプルデータ探訪

[WWDC 2017 の SceneKit サンプル Fox 2 を調べる その1 - Apple Engine](https://appleengine.hatenablog.com/entry/2018/04/17/130713)

https://appleengine.hatenablog.com/entry/2018/04/17/130713

<details><summary>Fox2SceneKitWWDC2017samplecode ディレクトリツリー</summary>

```Fox2SceneKitWWDC2017samplecode
.
├── Assets
│   ├── Art.scnassets
│   │   ├── character
│   │   │   ├── jump_dust.scn
│   │   │   ├── max.scn
│   │   │   ├── max_AO.png
│   │   │   ├── max_diffuse.png
│   │   │   ├── max_diffuseB.png
│   │   │   ├── max_diffuseC.png
│   │   │   ├── max_diffuseD.png
│   │   │   ├── max_idle.scn
│   │   │   ├── max_jump.scn
│   │   │   ├── max_roughness.png
│   │   │   ├── max_specular.png
│   │   │   ├── max_spin.scn
│   │   │   ├── max_walk.scn
│   │   │   └── smoke.png
│   │   ├── collision.scn
│   │   ├── enemy
│   │   │   ├── baddy_bad_AO.png
│   │   │   ├── baddy_bad_DIFFUSE.png
│   │   │   ├── baddy_bad_ILLUMINATION.png
│   │   │   ├── baddy_bad_NORMAL.png
│   │   │   ├── baddy_fearful_AO.png
│   │   │   ├── baddy_fearful_DIFFUSE.png
│   │   │   ├── baddy_fearful_ILLUMINATION.png
│   │   │   ├── baddy_fearful_NORMAL.png
│   │   │   ├── baddy_fearful_ROUGHNESS.png
│   │   │   ├── dot.png
│   │   │   ├── enemies.scn
│   │   │   ├── enemy1.scn
│   │   │   ├── enemy2.scn
│   │   │   ├── enemy_explosion.scn
│   │   │   ├── fireMask.png
│   │   │   ├── flare.png
│   │   │   └── gradient.png
│   │   ├── level_scene.scn
│   │   ├── particles
│   │   │   ├── burn.scn
│   │   │   ├── circle.png
│   │   │   ├── collect-big.scnp
│   │   │   ├── collect.scnp
│   │   │   ├── dot.png
│   │   │   ├── enemy_explosion.scn
│   │   │   ├── fire.png
│   │   │   ├── flare.png
│   │   │   ├── glow01.png
│   │   │   ├── glow_ellipse.png
│   │   │   ├── key_apparition.scn
│   │   │   ├── particles_spin.scn
│   │   │   └── unlock_door.scn
│   │   ├── scene.gks
│   │   ├── scene.scn
│   │   ├── textures
│   │   │   ├── Background_sky.png
│   │   │   ├── circle.png
│   │   │   ├── cubeMap4096.png
│   │   │   ├── depart-vieuxVolcan_AO.png
│   │   │   ├── depart-vieuxVolcan_lightmap.exr
│   │   │   ├── depart-vieuxVolcan_lightmap.png
│   │   │   ├── dot.png
│   │   │   ├── env_star.png
│   │   │   ├── fernSpiral_diffuse.png
│   │   │   ├── fernSpiral_illumination.png
│   │   │   ├── fernSpiral_roughness.png
│   │   │   ├── fern_diffuse.png
│   │   │   ├── fern_illumination.png
│   │   │   ├── fern_roughness.png
│   │   │   ├── fire.png
│   │   │   ├── flowmapPanda.png
│   │   │   ├── gem.png
│   │   │   ├── gradient_fire.png
│   │   │   ├── lavaDry_Med_diffuse.png
│   │   │   ├── lavaDry_Med_diffuse.psd
│   │   │   ├── lavaDry_diffuse.png
│   │   │   ├── lavaDry_roughness.png
│   │   │   ├── lava_cold2.png
│   │   │   ├── lava_hot.png
│   │   │   ├── lava_medium2.png
│   │   │   ├── lava_splash01.png
│   │   │   ├── lavalBubble.png
│   │   │   ├── mobilePT_diffuse.png
│   │   │   ├── mobilePlateformBig_diffuse.png
│   │   │   ├── mobilePlateformBig_roughness.png
│   │   │   ├── mobilePlateformBig_specular.png
│   │   │   ├── noisemap.png
│   │   │   ├── planks_albedo.png
│   │   │   ├── planks_metalness.png
│   │   │   ├── planks_normal.png
│   │   │   ├── planks_roughness.png
│   │   │   ├── plantes_AO.png
│   │   │   ├── plantes_lightmap.exr
│   │   │   ├── plantes_lightmap.png
│   │   │   ├── ripples.png
│   │   │   ├── rocks_border_diffuse.png
│   │   │   ├── rocks_border_normal.png
│   │   │   ├── rocks_border_roughness.png
│   │   │   ├── rocks_top_diffuse.png
│   │   │   ├── rocks_top_normal.png
│   │   │   ├── rocks_top_roughness.png
│   │   │   ├── sand_diffuse.png
│   │   │   ├── sand_normal.png
│   │   │   ├── sand_roughness.png
│   │   │   ├── sky_cube.exr
│   │   │   ├── sky_cube.png
│   │   │   ├── smoke_panda.png
│   │   │   ├── tile_blue_diffuse.png
│   │   │   ├── tile_blue_roughness.png
│   │   │   ├── tile_diffuse.png
│   │   │   ├── tile_roughness.png
│   │   │   ├── topLava_diffuse.png
│   │   │   ├── topLava_normal.png
│   │   │   ├── topLava_roughness.png
│   │   │   ├── wall_diffuse.png
│   │   │   └── wall_roughness.png
│   │   └── tile.png
│   ├── Assets.xcassets
│   │   ├── AppIcon.appiconset
│   │   │   ├── Contents.json
│   │   │   ├── icon-1024.png
│   │   │   ├── icon-29.png
│   │   │   ├── icon-29@2x.png
│   │   │   ├── icon-29@3x.png
│   │   │   ├── icon-40.png
│   │   │   ├── icon-40@2x.png
│   │   │   ├── icon-40@3x.png
│   │   │   ├── icon-60@2x.png
│   │   │   ├── icon-60@3x.png
│   │   │   ├── icon-76.png
│   │   │   ├── icon-76@2x.png
│   │   │   └── icon-83.5@2x.png
│   │   ├── Contents.json
│   │   ├── MacAppIcon.appiconset
│   │   │   ├── Contents.json
│   │   │   ├── Icon_Max-1024.png
│   │   │   ├── Icon_Max-128.png
│   │   │   ├── Icon_Max-16.png
│   │   │   ├── Icon_Max-256.png
│   │   │   ├── Icon_Max-32.png
│   │   │   ├── Icon_Max-512.png
│   │   │   └── Icon_Max-64.png
│   │   ├── tvOS App Icon & Top Shelf Image.brandassets
│   │   │   ├── App Icon - Large.imagestack
│   │   │   │   ├── Back.imagestacklayer
│   │   │   │   │   ├── Content.imageset
│   │   │   │   │   │   ├── Contents.json
│   │   │   │   │   │   └── tvos-layer1.png
│   │   │   │   │   └── Contents.json
│   │   │   │   ├── Background.imagestacklayer
│   │   │   │   │   ├── Content.imageset
│   │   │   │   │   │   ├── Contents.json
│   │   │   │   │   │   └── tvos-layer0.png
│   │   │   │   │   └── Contents.json
│   │   │   │   ├── Contents.json
│   │   │   │   ├── Front.imagestacklayer
│   │   │   │   │   ├── Content.imageset
│   │   │   │   │   │   ├── Contents.json
│   │   │   │   │   │   └── tvos-layer3.png
│   │   │   │   │   └── Contents.json
│   │   │   │   ├── Middle.imagestacklayer
│   │   │   │   │   ├── Content.imageset
│   │   │   │   │   │   ├── Contents.json
│   │   │   │   │   │   └── tvos-layer2.png
│   │   │   │   │   └── Contents.json
│   │   │   │   └── Near.imagestacklayer
│   │   │   │       ├── Content.imageset
│   │   │   │       │   ├── Contents.json
│   │   │   │       │   └── tvos-layer4.png
│   │   │   │       └── Contents.json
│   │   │   ├── App Icon - Small.imagestack
│   │   │   │   ├── Back.imagestacklayer
│   │   │   │   │   ├── Content.imageset
│   │   │   │   │   │   ├── Contents.json
│   │   │   │   │   │   └── tvos-layer1.png
│   │   │   │   │   └── Contents.json
│   │   │   │   ├── Background.imagestacklayer
│   │   │   │   │   ├── Content.imageset
│   │   │   │   │   │   ├── Contents.json
│   │   │   │   │   │   └── tvos-layer0.png
│   │   │   │   │   └── Contents.json
│   │   │   │   ├── Contents.json
│   │   │   │   ├── Front.imagestacklayer
│   │   │   │   │   ├── Content.imageset
│   │   │   │   │   │   ├── Contents.json
│   │   │   │   │   │   └── tvos-layer3.png
│   │   │   │   │   └── Contents.json
│   │   │   │   ├── Middle.imagestacklayer
│   │   │   │   │   ├── Content.imageset
│   │   │   │   │   │   ├── Contents.json
│   │   │   │   │   │   └── tvos-layer2.png
│   │   │   │   │   └── Contents.json
│   │   │   │   └── Near.imagestacklayer
│   │   │   │       ├── Content.imageset
│   │   │   │       │   ├── Contents.json
│   │   │   │       │   └── tvos-layer4.png
│   │   │   │       └── Contents.json
│   │   │   ├── Contents.json
│   │   │   ├── Top Shelf Image Wide.imageset
│   │   │   │   └── Contents.json
│   │   │   └── Top Shelf Image.imageset
│   │   │       ├── Contents.json
│   │   │       └── top-shelf.png
│   │   └── tvOS LaunchImage.launchimage
│   │       └── Contents.json
│   ├── Overlays
│   │   ├── MaxIcon.png
│   │   ├── collectableBIG_empty.png
│   │   ├── collectableBIG_full.png
│   │   ├── congratulations.png
│   │   ├── congratulations_pandaMax.png
│   │   ├── dpad.png
│   │   ├── key_empty.png
│   │   └── key_full.png
│   └── audio
│       ├── Explosion1.m4a
│       ├── Explosion2.m4a
│       ├── Music_victory.mp3
│       ├── Step_rock_00.mp3
│       ├── Step_rock_01.mp3
│       ├── Step_rock_02.mp3
│       ├── Step_rock_03.mp3
│       ├── Step_rock_04.mp3
│       ├── Step_rock_05.mp3
│       ├── Step_rock_06.mp3
│       ├── Step_rock_07.mp3
│       ├── Step_rock_08.mp3
│       ├── Step_rock_09.mp3
│       ├── aah_extinction.mp3
│       ├── ambience.mp3
│       ├── attack.mp3
│       ├── collect.mp3
│       ├── collectBig.mp3
│       ├── hit.mp3
│       ├── hitEnemy.wav
│       ├── jump.m4a
│       ├── ouch_firehit.mp3
│       ├── panda_catch_fire.mp3
│       ├── unlockTheDoor.m4a
│       └── volcano.mp3
├── LICENSE.txt
├── Objective-C
│   ├── fox2 Shared
│   │   ├── AAPLCharacter.h
│   │   ├── AAPLCharacter.m
│   │   ├── AAPLGameController.h
│   │   ├── AAPLGameController.m
│   │   ├── AAPLOverlay.h
│   │   ├── AAPLOverlay.m
│   │   ├── GPKComponents
│   │   │   ├── AAPLBaseComponent.h
│   │   │   ├── AAPLBaseComponent.m
│   │   │   ├── AAPLChaserComponent.h
│   │   │   ├── AAPLChaserComponent.m
│   │   │   ├── AAPLPlayerComponent.h
│   │   │   ├── AAPLPlayerComponent.m
│   │   │   ├── AAPLScaredComponent.h
│   │   │   └── AAPLScaredComponent.m
│   │   └── UI
│   │       ├── AAPLButton.h
│   │       ├── AAPLButton.m
│   │       ├── AAPLMenu.h
│   │       ├── AAPLMenu.m
│   │       ├── AAPLSlider.h
│   │       └── AAPLSlider.m
│   ├── fox2 iOS
│   │   ├── AAPLButtonOverlay.h
│   │   ├── AAPLButtonOverlay.m
│   │   ├── AAPLControlOverlay.h
│   │   ├── AAPLControlOverlay.m
│   │   ├── AAPLGameViewController.h
│   │   ├── AAPLGameViewController.m
│   │   ├── AAPLPadOverlay.h
│   │   ├── AAPLPadOverlay.m
│   │   ├── AppDelegate.h
│   │   ├── AppDelegate.m
│   │   ├── Base.lproj
│   │   │   ├── LaunchScreen.storyboard
│   │   │   └── Main.storyboard
│   │   ├── Info.plist
│   │   └── main.m
│   ├── fox2 macOS
│   │   ├── AAPLGameViewController.h
│   │   ├── AAPLGameViewController.m
│   │   ├── AppDelegate.h
│   │   ├── AppDelegate.m
│   │   ├── Base.lproj
│   │   │   └── Main.storyboard
│   │   ├── Info.plist
│   │   └── main.m
│   ├── fox2 tvOS
│   │   ├── AAPLGameViewController.h
│   │   ├── AAPLGameViewController.m
│   │   ├── AppDelegate.h
│   │   ├── AppDelegate.m
│   │   ├── Base.lproj
│   │   │   └── Main.storyboard
│   │   ├── Info.plist
│   │   └── main.m
│   └── fox2.xcodeproj
│       └── project.pbxproj
├── README.md
└── Swift
    ├── Shared
    │   ├── BaseComponent.swift
    │   ├── Character.swift
    │   ├── ChaserComponent.swift
    │   ├── GameController.swift
    │   ├── Overlay.swift
    │   ├── PlayerComponent.swift
    │   ├── ScaredComponent.swift
    │   ├── SimdExtensions.swift
    │   └── UI
    │       ├── Button.swift
    │       ├── Menu.swift
    │       └── Slider.swift
    ├── fox2-swift.xcodeproj
    │   └── project.pbxproj
    ├── iOS
    │   ├── AppDelegate.swift
    │   ├── Base.lproj
    │   │   ├── LaunchScreen.storyboard
    │   │   └── Main.storyboard
    │   ├── ButtonOverlay.swift
    │   ├── ControlOverlay.swift
    │   ├── GameViewController.swift
    │   ├── Info.plist
    │   └── PadOverlay.swift
    ├── macOS
    │   ├── AppDelegate.swift
    │   ├── Base.lproj
    │   │   └── Main.storyboard
    │   ├── GameViewController.swift
    │   └── Info.plist
    └── tvOS
        ├── AppDelegate.swift
        ├── Base.lproj
        │   └── Main.storyboard
        ├── GameViewController.swift
        └── Info.plist

58 directories, 281 files
```

</details>

今回使うデータは、`Assets/Art.scnassets/` の`character` と、`textures/Background_sky.png` です。

```今回使うデータ
.
├── Assets
│   ├── Art.scnassets
│   │   ├── character
│   │   │   ├── jump_dust.scn
│   │   │   ├── max.scn
│   │   │   ├── max_AO.png
│   │   │   ├── max_diffuse.png
│   │   │   ├── max_diffuseB.png
│   │   │   ├── max_diffuseC.png
│   │   │   ├── max_diffuseD.png
│   │   │   ├── max_idle.scn
│   │   │   ├── max_jump.scn
│   │   │   ├── max_roughness.png
│   │   │   ├── max_specular.png
│   │   │   ├── max_spin.scn
│   │   │   ├── max_walk.scn
│   │   │   └── smoke.png
│   │   ├── textures
│   │   │   ├── Background_sky.png
```

実行するファイルのディレクトリに`assets` とフォルダを作り、上記データを格納しておきます。

```
.
├── assets
│   ├── character
│   │   ├── jump_dust.scn
│   │   ├── max.scn
│   │   ├── max_AO.png
│   │   ├── max_diffuse.png
│   │   ├── max_diffuseB.png
│   │   ├── max_diffuseC.png
│   │   ├── max_diffuseD.png
│   │   ├── max_idle.scn
│   │   ├── max_jump.scn
│   │   ├── max_roughness.png
│   │   ├── max_specular.png
│   │   ├── max_spin.scn
│   │   ├── max_walk.scn
│   │   └── smoke.png
│   └── textures
│       └── Background_sky.png
└── 実行するファイル.py

```

これにて準備は完了です。実装していきましょう。

## 実装

![img221209_221315](https://user-images.githubusercontent.com/53405097/207569096-a22529da-9522-4f5b-a796-45430f95edc6.gif)





```py
from objc_util import load_framework, ObjCClass, on_main_thread
from objc_util import UIColor, UIImage, NSData, nsurl
import ui

import pdbg

load_framework('SceneKit')

SCNScene = ObjCClass('SCNScene')
SCNView = ObjCClass('SCNView')
SCNNode = ObjCClass('SCNNode')

SCNLight = ObjCClass('SCNLight')
SCNCamera = ObjCClass('SCNCamera')

SCNTorus = ObjCClass('SCNTorus')


class GameScene:
  def __init__(self):
    self.scene: SCNScene
    self.setUpScene()

  def setUpScene(self):

    # --- import
    foxmax_URL = nsurl('./assets/character/max.scn')
    bkSky_URL = NSData.dataWithContentsOfURL_(
      nsurl('./assets/textures/Background_sky.png'))
    tex_bks = UIImage.alloc().initWithData_(bkSky_URL)

    # --- SCNScene
    scene = SCNScene.sceneWithURL_options_(foxmax_URL, None)
    scene.background().contents = tex_bks
    scene.lightingEnvironment().contents = tex_bks
    scene.lightingEnvironment().intensity = 1.24

    scene_rootNode_addChildNode_ = scene.rootNode().addChildNode_

    foxmaxNode = scene.rootNode().objectInChildNodesAtIndex_(0)

    torus = SCNTorus.torusWithRingRadius_pipeRadius_(0.075, 0.02)
    torus.ringSegmentCount = 8
    torus.pipeSegmentCount = 8

    torus.material().lightingModelName = 'SCNLightingModelPhysicallyBased'

    torus.firstMaterial().reflective().contents = 0.9
    torus.firstMaterial().diffuse().contents = UIColor.yellowColor()

    torusNode = SCNNode.nodeWithGeometry_(torus)
    torusNode.position = (0.0, 0.55, 0.0)

    scene_rootNode_addChildNode_(torusNode)

    # --- SCNLight
    lightNode = SCNNode.node()
    lightNode.light = SCNLight.light()
    lightNode.castsShadow = True
    lightNode.position = (0.0, 10.0, 10.0)
    scene_rootNode_addChildNode_(lightNode)

    # --- SCNCamera
    camera = SCNCamera.camera()

    camera.wantsHDR = True
    camera.bloomBlurRadius = 18.0
    camera.bloomIntensity = 1.0
    camera.bloomThreshold = 1.0
    camera.colorFringeIntensity = 4.0
    camera.colorFringeStrength = 4.0
    camera.motionBlurIntensity = 6

    camera.xFov = 35.0
    camera.yFov = 35.0

    cameraNode = SCNNode.node()
    cameraNode.camera = camera
    cameraNode.position = (0.0, 0.25, 2.0)
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
    _debugOptions = ((1 << 0) | (1 << 1) | (1 << 4) | (1 << 5) | (1 << 10))
    scnView.debugOptions = _debugOptions

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

立ち上がり時のカクツキにドキッとしますが、それ以降は問題なく動きます。

高性能なiPhone をモリモリと使っている気分でいいですよね。

### データインポート

```py
from objc_util import UIImage, NSData, nsurl

# --- import
foxmax_URL = nsurl('./assets/character/max.scn')
scene = SCNScene.sceneWithURL_options_(foxmax_URL, None)

bkSky_URL = NSData.dataWithContentsOfURL_(
  nsurl('./assets/textures/Background_sky.png'))
tex_bks = UIImage.alloc().initWithData_(bkSky_URL)

```

Swift やObjective-C では、標準的過ぎて解説が少ない代表例が、インポート関係です。

今回は、ファイルパスを`objc_util` で読める形式にしてから読み込みをさせています。

他には、`MDLAsset` を使ったデータ読み込みの方法もあります。

`SCNScene` より、`.scn` を読み込ませているので、`max.scn` に依存したScene となっています。

今回は、触れませんでしたが`max` ちゃんの内部を見るには:

```py
foxmaxNode = scene.rootNode().objectInChildNodesAtIndex_(0)
```

これで、Node へアクセスできます。

[WWDC 2017 の SceneKit サンプル Fox 2 を調べる その3 - Apple Engine](https://appleengine.hatenablog.com/entry/2018/04/17/160842)

https://appleengine.hatenablog.com/entry/2018/04/17/160842

### カメラエフェクト

```py
camera.wantsHDR = True
camera.bloomBlurRadius = 18.0
camera.bloomIntensity = 1.0
camera.bloomThreshold = 1.0
camera.colorFringeIntensity = 4.0
camera.colorFringeStrength = 4.0
camera.motionBlurIntensity = 6
```

ゴリゴリにエフェクトを付けてしまい、見た目が悪いですがさまざまななエフェクトがあります。

機能詳細は毎度のごとくの参照先ですが、比較的簡単な設定で効果がすごいのでパラメータを操作するだけでも楽しいです。

[iOS で SceneKit を試す(Swift 3) その38 - Scene Editor カメラの基本設定 - Apple Engine](https://appleengine.hatenablog.com/entry/2017/07/24/164433)

https://appleengine.hatenablog.com/entry/2017/07/24/164433

### キューブマップや他の3DCG データ

さりげなく処理をさせていましたが、`textures/Backgtound_sky.png` にて、キューブマップを設定しています。

ジオメトリの質感が一気に変わりリアル感が増すので、キューブマップもいろいろと差し替えて変化を楽しむのもいいですね。

```py
scene.background().contents = tex_bks
scene.lightingEnvironment().contents = tex_bks
scene.lightingEnvironment().intensity = 1.24
```

[iOS で SceneKit を試す(Swift 3) その82 - キューブマップを設定する - Apple Engine](https://appleengine.hatenablog.com/entry/2017/08/28/193324)

https://appleengine.hatenablog.com/entry/2017/08/28/193324

以下は、過去に縄文土器のデータとNASA　のスターマップをキューブマップにしたキャプチャです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ワイのiPhoneが宇宙や<a href="https://twitter.com/hashtag/NASA?src=hash&amp;ref_src=twsrc%5Etfw">#NASA</a> <a href="https://twitter.com/hashtag/jomon?src=hash&amp;ref_src=twsrc%5Etfw">#jomon</a> <a href="https://twitter.com/hashtag/jomonosp?src=hash&amp;ref_src=twsrc%5Etfw">#jomonosp</a> <a href="https://twitter.com/hashtag/%E7%B8%84%E6%96%87%E3%82%AA%E3%83%BC%E3%83%97%E3%83%B3%E3%82%BD%E3%83%BC%E3%82%B9%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88?src=hash&amp;ref_src=twsrc%5Etfw">#縄文オープンソースプロジェクト</a> <a href="https://twitter.com/hashtag/Pythonista?src=hash&amp;ref_src=twsrc%5Etfw">#Pythonista</a> <a href="https://t.co/2Z5DPsG0oF">https://t.co/2Z5DPsG0oF</a> <a href="https://t.co/MYtRkKJ5kU">pic.twitter.com/MYtRkKJ5kU</a></p>&mdash; pome-ta (@pome_ta93) <a href="https://twitter.com/pome_ta93/status/1308027038008131584?ref_src=twsrc%5Etfw">September 21, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[SVS: Deep Star Maps 2020](https://svs.gsfc.nasa.gov/4851)

https://svs.gsfc.nasa.gov/4851

[縄文オープンソースプロジェクト | 縄文文化発信サポーターズ](https://jomon-supporters.jp/open-source/)

https://jomon-supporters.jp/open-source/

縄文土器は、`.stl` データで、41.9MB でした。。。パケ死に以外は心配せずに、問題なく読み込んで描画してくれます。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">おそら、きれい<br><br>な、火焔型土器さん<a href="https://twitter.com/hashtag/jomon?src=hash&amp;ref_src=twsrc%5Etfw">#jomon</a> <a href="https://twitter.com/hashtag/jomonosp?src=hash&amp;ref_src=twsrc%5Etfw">#jomonosp</a> <a href="https://twitter.com/hashtag/%E7%B8%84%E6%96%87%E3%82%AA%E3%83%BC%E3%83%97%E3%83%B3%E3%82%BD%E3%83%BC%E3%82%B9%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88?src=hash&amp;ref_src=twsrc%5Etfw">#縄文オープンソースプロジェクト</a><a href="https://twitter.com/hashtag/Pythonista3?src=hash&amp;ref_src=twsrc%5Etfw">#Pythonista3</a> <a href="https://twitter.com/hashtag/Python?src=hash&amp;ref_src=twsrc%5Etfw">#Python</a> <a href="https://t.co/GXHh8nEfSY">pic.twitter.com/GXHh8nEfSY</a></p>&mdash; pome-ta (@pome_ta93) <a href="https://twitter.com/pome_ta93/status/1295550764782256128?ref_src=twsrc%5Etfw">August 18, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

3DCG 関係はさまざまな形式でデータがあり、配布されているので。SNS 等で知ったデータをすぐにiPhone で確認できてしまう。というのは面白いですね。

## パーティクル

3DCG で絵力といえば、パーティクル表現なのではないかと、勝手に思っています。

SceneKit にももちろんパーティクル機能があるので、一部の機能のみですが最後に紹介をして終えたいと思います。

[iOS で SceneKit を試す(Swift 3) その78 - パーティクルシステムのパラメーターをみてみる - Apple Engine](https://appleengine.hatenablog.com/entry/2017/08/23/202915)

https://appleengine.hatenablog.com/entry/2017/08/23/202915


![img221210_005139](https://user-images.githubusercontent.com/53405097/207569413-4cc41dc8-3f19-49f3-ab25-755f9d6ebea1.gif)




```py
from objc_util import load_framework, ObjCClass, on_main_thread
from objc_util import UIColor, UIImage, NSData, nsurl
import ui

import pdbg

load_framework('SceneKit')

SCNScene = ObjCClass('SCNScene')
SCNParticleSystem = ObjCClass('SCNParticleSystem')
SCNSphere = ObjCClass('SCNSphere')
SCNNode = ObjCClass('SCNNode')
SCNAction = ObjCClass('SCNAction')
SCNLight = ObjCClass('SCNLight')
SCNCamera = ObjCClass('SCNCamera')
SCNView = ObjCClass('SCNView')


class GameScene:
  def __init__(self):
    self.scene: SCNScene
    self.setUpScene()

  def setUpScene(self):

    # --- import
    bkSky_URL = NSData.dataWithContentsOfURL_(
      nsurl('./assets/textures/Background_sky.png'))
    tex_bks = UIImage.alloc().initWithData_(bkSky_URL)

    # --- SCNScene
    scene = SCNScene.scene()
    scene.background().contents = tex_bks
    scene.lightingEnvironment().contents = tex_bks
    scene.lightingEnvironment().intensity = 1.24

    scene_rootNode_addChildNode_ = scene.rootNode().addChildNode_

    emitterBall = SCNSphere.sphereWithRadius_(0.25)
    emitterBall.geodesic = True
    emitterBall.segmentCount = 32

    # SCNParticleSystem
    particleSys = SCNParticleSystem.particleSystem()
    particleSys.emitterShape = emitterBall
    particleSys.birthRate = 1024
    particleSys.birthRateVariation = 128
    particleSys.birthDirection = 2
    particleSys.particleColor = UIColor.blueColor()
    particleSys.particleColorVariation = (0.1, 0.4, 0.6, 0.5)
    particleSys.particleAngle = 0.1
    particleSys.particleAngleVariation = 8
    particleSys.particleVelocity = 0.1
    particleSys.particleVelocityVariation = 0.2
    particleSys.particleSize = 0.001
    particleSys.particleSizeVariation = 0.005
    particleSys.emissionDuration = 0.001
    particleSys.emissionDurationVariation = 0.1
    particleSys.particleLifeSpan = 1
    particleSys.particleLifeSpanVariation = 8
    particleSys.particleAngularVelocity = 0.01
    particleSys.particleAngularVelocityVariation = 8
    particleSys.idleDuration = 0.4
    particleSys.idleDurationVariation = 0.8
    particleSys.particleIntensity = 2
    particleSys.particleIntensityVariation = 4
    particleSys.isLightingEnabled = True

    particleNode = SCNNode.node()
    particleNode.addParticleSystem(particleSys)
    scene_rootNode_addChildNode_(particleNode)

    # --- SCNCamera
    camera = SCNCamera.camera()
    camera.wantsHDR = True
    camera.wantsExposureAdaptation = True
    camera.exposureAdaptationBrighteningSpeedFactor = 0.02
    camera.exposureAdaptationDarkeningSpeedFactor = 0.1
    camera.minimumExposure = -15
    camera.maximumExposure = 15
    camera.bloomIntensity = 2.0
    camera.bloomThreshold = 0.6
    camera.bloomBlurRadius = 18
    camera.colorFringeIntensity = 4.0
    camera.colorFringeStrength = 4.0
    camera.motionBlurIntensity = 6
    camera.xFov = 35.0
    camera.yFov = 35.0
    camera.zNear = 2
    camera.zFar = 100

    cameraNode = SCNNode.node()
    cameraNode.camera = camera
    cameraNode.position = (0.0, 1.0, 2.0)
    cameraNode.eulerAngles = (-0.5, 0, 0)

    dollyNode = SCNNode.node()
    dollyNode.addChildNode_(cameraNode)

    dollyNode.runAction_(
      SCNAction.repeatActionForever_(
        SCNAction.rotateByX_y_z_duration_(0.0, 2.0, 0.1, 8.0)))

    scene_rootNode_addChildNode_(dollyNode)

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
    _debugOptions = ((1 << 0) | (1 << 1) | (1 << 3) | (1 << 5) | (1 << 10))
    scnView.debugOptions = _debugOptions

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

パラメータが多過ぎて何が何だか😅 な状態ですが、、、、

ここまで、読んでいただきありがとうございました。

次回👇

https://qiita.com/pome-ta/items/29785f0edbe582210d2f

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
