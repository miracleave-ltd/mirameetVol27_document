# RSpecの概要解説
## 本セクションで行うこと
- RSpecとは何か
- RSpecのメリット・デメリット
- RSpecでできること

## RSpecとは？

突然ですが、皆さんの開発現場ではテストはどのように実施されてますか？  

![image](https://user-images.githubusercontent.com/52161269/157041610-880f46a7-5bc7-4366-bdec-9982b269fa74.png)


私が経験した現場では、色々なテストの形がありました。  
動作確認≒テストになっている、エクセルにエビデンスのキャプチャを貼って行うレガシーなテスト、テストコードを書いて行う自動テスト、、、、、など現場によって様々です。


[公式ドキュメント](https://relishapp.com/rspec/)よりRSpecについて下記のように説明されています。
```
RSpec is a Behaviour-Driven Development tool for Ruby programmers. BDD is an approach
to software development that combines Test-Driven Development, Domain Driven Design,
and Acceptance Test-Driven Planning. RSpec helps you do the TDD part of that equation,
focusing on the documentation and design aspects of TDD.
```

一言で言うと、  
**RSpecは、（Rubyプログラマーの為の）BDDツールである。**  

BDD・・・振る舞い駆動開発(Behaviour-Driven Development)のこと。TDD（テスト駆動開発）から派生した開発手法。BDDでは実装前に要求される振る舞い(≒仕様)をテストに書き出してから実装し、また、要求される振る舞いは自然言語(英語など)に近い形で表現されます。

RailsプロジェクトでRSpecを利用するには「rspec-rails」というライブラリ（gem）のインストールが必要になります。




## RSpecのメリット・デメリット

RSpecを導入することで、以下のようなメリットがあります。

**メリット**
- テストのコストが抑えられる  
例）エクセルにテスト仕様書を書き出す場合、、、  
<img width="727" alt="スクリーンショット 2022-03-13 17 35 51" src="https://user-images.githubusercontent.com/52161269/158051703-e4f4a393-bbfe-49ab-962e-fb40a1774bc3.png">
RSpecを利用すると、テスト実施はRSpecが行う

**(グレーアウトしている工程は削減できる工程)**
<img width="779" alt="スクリーンショット 2022-03-27 19 09 14" src="https://user-images.githubusercontent.com/52161269/160276619-ecfa6be6-b2a8-436c-aa80-8de191d39665.png">


- テスト実施や仕様書の工数を削減できる（中長期なメリット）  
一度テストコードを書くと、その後の追加開発時で既存プログラムへのテスト工数が以下のように削減できる  
**(グレーアウトしている工程は削減できる工程)**
<img width="777" alt="スクリーンショット 2022-03-27 19 10 45" src="https://user-images.githubusercontent.com/52161269/160276657-de387b65-0fa9-4200-ae5d-4e5c835b58e9.png">


- 追加開発時のデグレ発生を防止できる（中長期なメリット）
- 開発者は品質を保ちながらも開発が行える（開発しながらテストを実行することができるので、バグやデグレに気付きやすくなる為）
- 誰がやっても同じテストが実施される為、テストのQAが安定する


**デメリット**
- RSpecの学習コストが必要
- 開発時にテストコードも書くので、開発自体の工数が増加する


## RSpecでできること

- 単体テストができる
- 結合テストができる
- システムテストができる





