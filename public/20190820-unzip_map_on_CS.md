---
title: .Net(C#)を用いてZipアーカイブから解凍無しにOpenCV：Matに変換する方法（短報）
tags:
  - C#
  - OpenCV
  - .NETFramework
private: false
updated_at: '2019-08-20T13:34:30+09:00'
id: b14ea954f608b3f755ab
organization_url_name: null
slide: false
ignorePublish: false
---
# abstract
Deep LearningによるCVが多用されるようになり、これにかかわる技術開発をされている方が多いでしょう。
出力結果の画像を管理・表示する際、PythonのプログラムでGUI（それこそPyOpenCVなど）を表示することもできるでしょう。
ただ、windowsでプログラムをされている方は何となく.Netで作りたくならないでしょうか（？）。近年は.Net coreによってLinux環境下でも同じソースで動かすことができるようになっております。

# method
さて、学習させて何らかの予測・画像生成を行ったとき出力を何らかの圧縮形式で一つのファイルにすることがあるでしょう。これを.Net Form上に表示しようとしたとき.Net の画像形式ImageもしくはBitmapが用いられる。これらのコンストラクタは開いたファイルオブジェクト引数にとれる。

```
using (var entry = ZipFile.GetEntry(filename).Open())//filenameはzipファイル内の画像ファイルを指す
{
    var bit = Bitmap(entry);
}
```
しかし、以下のコードは動かない。

```
var Mat = Mat(entry); //開いたファイルは引数にとれない
```

ただ、表示するにもただ結果を出力するだけでなく、少し画像の加工が必要な場合OpenCVで加工したくなる。ただ、zipの解凍はしたくない。
そこでBitmap -> Mat変換が可能であれば、それができる。
少し調べたところ

```
using OpenCvSharp.Extensions;//ここが重要。ToMatメソッドが追加される。

using (var entry = ZipFile.GetEntry(filename).Open())//filenameはzipファイル内の画像ファイルを指す
{
    var mat = Bitmap(entry).ToMat();
}

```
以上、zipの解凍を行わず,Matを生成して画像処理できるようになった。
