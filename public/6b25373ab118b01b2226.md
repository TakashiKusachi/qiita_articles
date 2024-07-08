---
title: Ubuntu 16.04の仮想端末ttyでログインできなかった時のお話
tags:
  - Ubuntu
  - tty
  - TeamViewer
private: false
updated_at: '2019-02-19T16:46:03+09:00'
id: 6b25373ab118b01b2226
organization_url_name: null
slide: false
ignorePublish: false
---
#ことの発端は、workstationを完全リモート化しようとした時
　Deep Learning workstation兼データサーバのworkstationを完全リモート化するために、いろいろインストールしてた時、guiがgpuを使用しているのでこれをどうにかしたいと思いました。
　詳しく調べると、

```
#systemctl stop lightdm
#./NVI...(driver) -s --no-opengl-files
#./cuda... -silent --no-opengl-libs 
```
の様にインストールすると、vncでguiを使用しないらしい。（参考サイトはそのうちまとめます。

そこで、Ctrl+F1で仮想端末に入り、ログインしようとしたら

```
user name: ***
passwork : (勝手に空白で入力）
Login incorrect
```
ふぁ！！！！

#virtual boxにubuntuをなんどもインストールして検証ー＞teamviewerが原因
　何が原因だかあまりに不明だったため、workstationになんどもubuntuをインストールし直し、なんども環境を再構築し、なんども問題が再現し・・・
　nvidiaのdriverであったり、cudaの問題ではなさそうだったため、lxdeやvncの問題を疑いvirtualbox下で検証（実ハードは起動が手間）。
　一コマンドずつreboot、環境再構築を行ったところteamviewerインストール直後から当該エラーが発生。uninstallすると直ることを確認。

#teamviewerで調べてもこのような情報は出てこなかった。
おそらく、teamviewerを使うかつttyにアクセスを行うような人間の数を考えると、かなり希少な情報なのではないかと思った。
