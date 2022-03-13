# 単体テスト
## 本セクションで行うこと
1. RSpecの導入方法
2. Modelの単体テストの実装
3. APIの単体テスト

## 1. RSpecの導入方法
まずはRSpecをinstallしていきます。Gemfileに追加します。

`Gemfile`
```ruby
group :development, :test do
 # 省略

  gem 'rspec-rails'  #←追加
end
```
Gemfileに`gem 'rspec-rails'`を追加したらbundle installしていきます。

`ターミナル`
```ruby
bundle install
```
終了したら、ジェネレーターを使用してRSpecのinstallを完了させます。

`ターミナル`
```ruby
rails g rspec:install
```

下記のようにディレクトリ・ファイルが作成されれば完了です。
create  .rspec
create  spec
create  spec/spec_helper.rb
create  spec/rails_helper.rb

※RSpecの設定は各種ありますが、今回は割愛します。


## 2. Modelの単体テストの実装

### モデルスペックの観点
- 有効な属性で初期化した場合、モデルが有効であることを検証する。
- 無効な属性で初期化した場合、モデルが有効ではないことを検証する。
- クラスメソッドとインスタンスメソッドが定義されている場合は期待通りに動作すること。

ファイルの作成　ジェネレーター
モデルスペックの観点
exampleを書く it '~' do end
実装
テストデータの作成方法(Factorybot、Fackerの導入)
DRYに書く(before, subject, letなど)
外部APIテスト
vcrの使用方法

