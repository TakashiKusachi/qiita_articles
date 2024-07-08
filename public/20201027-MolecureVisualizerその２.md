---
title: 'フロントエンド(TypeScript, Three.js)のみで動く、Molecular Visualizerの開発（その２）'
tags:
  - three.js
  - TypeScript
  - Vue.js
  - chemoinformatics
  - Threads.js
private: false
updated_at: '2020-10-27T07:02:54+09:00'
id: 95495c5af218c10390f4
organization_url_name: null
slide: false
ignorePublish: false
---
## 概要
詳しくは[その１](https://qiita.com/TakashiKusachi/items/0c44dd2efcd631dab3e3)を参照ください

## 現状
![heat.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/364025/41a85420-fae4-f563-2cf6-97ee144e2ada.png)
色が変わったくらいでほとんど見た目は前回から変わってない。


## (前回との）更新内容
画面構成を部分的にVue.js化
headerにCSSでhover時の色の変化を加えた。
他細かな修正とまだ記事にしてない機能の追加


## 分子構造データクラスとパーサー
現状対象としている構造データは、原子が3D空間上に点在していて、任意の原子間に結合が存在している。
つまりデータ構造としては、原子のリストと、結合のリストと、それのコンテナである。
これを愚直に抽象化したインターフェースを次のように設計した。AtomにはelementType以外にFFtypeなどが必要ではないかという話は、いったんおきましょう。

Atom　インターフェース(./src/systems/system.ts)

```
enum ElemType{
    ANY,H,He,Li,Be,B,C,N,O,F,Ne,
}

export type Position = Array<number>;
export type Name = string;

export interface IAtom{
    position: Position;
    name: Name;
    element: ElemType;
    index: number;
}
```

Bond インターフェース(./src/systems/system.ts)

```
export type BOND_TYPE= number;

export interface IBond<AtomType extends IAtom = IAtom>{
    atoma: AtomType;
    atomb: AtomType;

    vector: number[];
    distance: number;

    position: Position; 
}
```

System　インターフェース(./src/systems/system.ts)

```
export interface ISystem<AtomType extends IAtom,BondType extends IBond<AtomType>>{
    /**getterでの実装を禁ズ */
    systemName: string;

    natoms: number;
    nbond: number;
    getNames(): string[];
    getPositions(): Position[];
    getElements(): ElemType[];
    getAtom(index:number): AtomType;
    atomIndexOf(name:string): number;
    getAtoms():Generator<AtomType,string,unknown>;
    getBond():Generator<BondType,void,unknown>;
}
```

実現クラスは設計では一つしか作らない予定であるため、インターフェースクラスを用意する意味はほとんどないが、ひとまず経由することとした。

このSystemクラスは、描画するクラスに引き渡され描画のためのオブジェクトの作成や、IAtom.nameを使ってクリックイベントの対象物の識別に使われる（予定）。
