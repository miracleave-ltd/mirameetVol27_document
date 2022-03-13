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

### ファイルの作成
`ターミナル`
```ruby
rails g rspec:model user
```

`spec/models/user_spec.rb`というファイルが作成されていればOKです。

### Userモデルの要件
```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  # 有効な属性の場合のテスト
  it "nickname, email, password, password_confirmationがあれば有効であること"
  
  # アソシエーションのテスト
  it "postモデルとのアソシエーションが有効であること"
  it "commentモデルとのアソシエーションが有効であること"
  
  # 各属性の有効・無効の場合のテスト
  it "nicknameがnilの場合、無効であること"
  it "nicknameが空文字の場合、無効であること"
  it "nicknameが10文字以内の場合、有効であること"
  it "nicknameが11文字以上の場合、無効であること"
  it "emailがnilの場合、無効であること"
  it "emailが空文字の場合、無効であること"
  it "emailが既に保存されている場合、無効であること"
  it "emailがemailの形式ではない場合、無効な状態であること"
  it "emailは全角文字を使用する場合、無効な状態であること" do
  it "passwordがnilの場合、無効であること"
  it "passwordが空文字の場合、無効であること"
  it "passwordが5文字以内の場合、無効であること"
  it "passwordが6文字以上の場合、有効であること"
  it "passwordが128文字以内の場合、有効であること"
  it "passwordが129文字以上の場合、無効であること"
  it "password_confirmationがnilの場合、無効であること"
  it "password_confirmationが空文字の場合、無効であること"
  it "passwordとpassword_confirmationが不一致の場合、無効であること"
end

```

#### describe
期待する結果をまとめる。上記では `descrive User`としていて、これがUserモデルのテストであると明示している。

#### it
実際のテストを実行するexampleを定義している。基本的にexample一つにつき一つの結果を期待する。
- exampleは明示的に記載する。省略することもできるが、可読性が落ちるため、基本的には記述する。
- exampleの説明は動詞で始まる。例えば、`nicknameがnilの場合、無効であること`を英語に置き換えると`is Invalid if nickname is nil`となる。



exampleを書く it '~' do end
実装
テストデータの作成方法(Factorybot、Fackerの導入)
DRYに書く(before, subject, letなど)
外部APIテスト
vcrの使用方法

