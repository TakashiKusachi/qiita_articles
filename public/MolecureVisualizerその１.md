---
title: 'フロントエンド(TypeScript, Three.js)のみで動く、Molecular Visualizerの開発（その１）'
tags:
  - three.js
  - TypeScript
  - フロントエンド
  - chemoinformatics
  - Threads.js
private: false
updated_at: '2020-12-09T02:06:24+09:00'
id: 0c44dd2efcd631dab3e3
organization_url_name: null
slide: false
ignorePublish: false
---
## 概要
source code (https://github.com/TakashiKusachi/TS-GLMolViwer)
### 目的
ブラウザ上で動く分子などの構造の可視化するVisualizerの開発がしたくなったので、開発するというただの自己満足

### 既往開発
GLMol (https://glmol.com)
ChemDoodleWeb (https://web.chemdoodle.com/)

## 現状
とりあえず、構造ファイルを読み込み3Dモデルを表示するところまでできました。
![キャプチャ.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/364025/b741c3aa-cdcf-23a1-27e9-14b83b01ad5a.png)
公開URL (https://TakashiKusachi.github.io/TS-GLMolViwer/front/app)
編集2020/12/09　URLを変更しました
構造ファイルは.carに対応しています。

## 開発方針
既往開発に挙げたプログラムはJavascriptで書かれているが、今回はECMAScript2015ベースにTypescriptでの開発を基本方針とした。また、画面が3秒以上フリーズしないようにThree.jsの処理をバックグラウンドへ流すようにする。

### 妥協点
* タンパク質などのリボン表記はあきらめる（タンパク質詳しくない）
* 軌跡ファイルのアニメーションはだいぶ後半に実装する。（モデリングツールとして開発する）
* 二～三種の構造ファイルに対応する。（拡張性は維持する）
* Vue.jsなどのフレームワークを使いたかったけど、うまくいかなかったので諦め

# 開発環境の構築
* IDE: Visual Studio Code
* ブラウザ: Google Chrome(WebGLとOffscreenCanvasに対応しているブラウザ）
* typescript compiler: Node.jsより構築(+Webpack)
* WebGL Flamework: Three.js
* マルチスレッド処理バックエンド: Threads.js

## 
# 画面構成
画面構成はIDEなどでよくある2カラムのheader,fotterありの構成。大手商用のvisualiserも同様の構成であるため、この構成を採用しました。
![_キャプチャ - コピー.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/364025/36b6479b-33b3-18f8-5a1c-9315ef4b4737.png)

HTMLコード

```
<!DOCTYPE html>
<html>
    <head>
        <title>index page</title>
        <link rel="stylesheet" href="style/main.css">
    </head>
    <body>
        <div id="loader-bg" class="is-hide">
            <div id="loader" class="is-hide">
                <p>NowLoading</p>
            </div>
        </div>
        <header id="header-contents">
            <ul class="header_single">
                <li>
                    File
                    <ul> 
                        <li><label>New<input id="new" type="button"></label></li>
                        <li><label>OpenFile<input id="file" type="file" multiple></label></li>
                    </ul>
                </li>
                <li>
                    Viewer
                    <ul>
                        <li><label>BackGraund<input id="back" type="color"></label></li>
                    </ul>
                </li>
            </ul>
        </header>

        <div id="main-contents">
            <div class="content" id="view-area">
                <canvas id="gl_canvas"></canvas>
            </div>
            <div class="content" id="control-area">
            </div>
        </div>
        <div id="fotter-contents">
            <div id="state"></div>
        </div>
        <script src="main.js" type="text/javascript"></script>
    </body>
</html>
```

CSS

```
* {
    margin: 0px;
    padding: 0px;
}

html, body{
    height: 100%;
    overflow: hidden;
}

.is-hide{
    display:none;
}

#loader-bg{
    width:100%;
    height: 100%;
    position: fixed;
    left: 0;
    top: 0;
    z-index: 100;

    background-color: rgba(0,0,0,0.5);
}

#loader{
    text-align: center;
    position: absolute;
    width: 150px;
    height: 50px;
    background-color: rgb(21, 6, 92);
    border: black;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
}

#loader > p{
    color: antiquewhite;
}

#header-contents{
    height: 20px;
}

ul.header_single{
    display: flex;
    height: 20px;
    background-color: rgb(137, 187, 221);
    list-style: none;
}

ul.header_single > li{
    height: 20px;
    padding-right: 10px;
    padding-left: 10px;
    background-color: rgb(140, 221, 137);
    position: relative;
}

ul.header_single > li > ul{
    background-color: rgb(137, 187, 221);
    position: absolute;
    top: 100%;
    margin: 0px;
    left: 0px;

    display: none;

    list-style: none;
}

ul.header_single > li:hover > ul{
    display: block;
}

ul.header_single > li > ul > li{
    padding-right: 10px;
    padding-left: 10px;
    margin: 0px;
}

ul.header_single > li > ul > li > label > input{
    display: none;
}

#main-contents{
    overflow: hidden;
    display: block;
    width: 100%;
    height: calc(100% - 40px);
}

#main-contents > .content{
    height: 100%;
}

#view-area {
    float: left;
    width: 70%;
    height: 100%;
}

#gl_canvas {
    width: 100%;
    display: block;
}

#main-contents > #control-area{
    height: calc(100% - 6px);
    float: right;
    overflow: scroll;
    width: 28%;
    border: 2px solid black;
}

#fotter-contents{
    height: 20px;
}
```

その２へ続く（といいな）

なにかコメントがあればぜひ、記事のコメント欄、もしくはgithubのissuesまで
