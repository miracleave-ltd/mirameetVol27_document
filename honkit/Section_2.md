# RSpecの概要解説
## 本クションで行うこと
1. RSpecとは何か
2. RSpecのメリット・デメリット
3. RSpecでできること

## RSpecとは？

突然ですが、、、、皆さんの開発現場ではテストはどのように実施されてますか？  

![image](https://user-images.githubusercontent.com/52161269/157041610-880f46a7-5bc7-4366-bdec-9982b269fa74.png)


私が経験した現場では、いろいろなテストの形がありました。  
動作確認≒テストになっている、エクセルにエビデンスのキャプチャを張って行うレガシーなテスト、テストコードを書いて行う自動テスト、、、、、など現場によって様々です。


[公式ドキュメント](https://relishapp.com/rspec/)よりRSpecについて下記のように説明されています。
```
RSpec is a Behaviour-Driven Development tool for Ruby programmers. BDD is an approach
to software development that combines Test-Driven Development, Domain Driven Design,
and Acceptance Test-Driven Planning. RSpec helps you do the TDD part of that equation,
focusing on the documentation and design aspects of TDD.
```

一言で言うと、  
**RSpecは、（Rubyプログラマー為の）BDDツールである。**  

BDD・・・振る舞い駆動開発(Behaviour-Driven Development)のこと。TDD（テスト駆動開発）から派生した開発手法。BDDでは実装前に要求される振る舞い(≒仕様)をテストに書き出してから実装し、  
また、要求される振る舞いは自然言語(英語など)に近い形で表現されます。

Railsプロジェクトで、RSpecでテストを行うには「rspec-rails」というライブラリ（gem）のインストールが必要になります。



## RSpecのメリット・デメリット

RSpecを導入することで、以下のようなメリットがあります。

**メリット**
- テストのコストが抑えられる  
例）エクセルにテスト仕様書を書き出す場合、、、  
<img width="898" alt="スクリーンショット 2022-03-08 23 07 46" src="https://user-images.githubusercontent.com/52161269/157253926-1606b105-94ed-42fc-9082-c87288fcc717.png">
    RSpecを利用すると、、、、、
<img width="912" alt="スクリーンショット 2022-03-08 23 30 57" src="https://user-images.githubusercontent.com/52161269/157258416-5bad2eb7-9c5f-4ca9-9f63-57ddf44ef52b.png">

- 誰がやっても同じテストができる
- テストの時間を削減できる（中長期なメリット）
- 開発後のデグレ発生を防止できる（中長期なメリット）


**デメリット**
- RSpecの学習コストが必要
- 開発の工数が増加する


## RSpecでできること

- 単体テストができる
- 結合テストができる
- 統合テストができる





