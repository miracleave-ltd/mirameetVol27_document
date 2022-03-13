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
  it "nickname, email, password, password_confirmationがあれば有効であること"
  it "postモデルとのアソシエーションが有効であること"
  it "commentモデルとのアソシエーションが有効であること"
  it "nicknameがnilの場合、無効であること"
  it "nicknameが空文字の場合、無効であること"
  it "nicknameが10文字以内の場合、有効であること"
  it "nicknameが11文字以上の場合、無効であること"


  describe 'email' do
    it { is_expected.to validate_presence_of :email }
    it { is_expected.to validate_uniqueness_of(:email).case_insensitive }
    it 'emailの形式ではない場合、無効な状態であること' do
      user = build(:user, email: 'example_no_email')
      user.valid?
      expect(user.errors[:email]).to include("は不正な値です") #invalid
    end

    it 'emailは全角文字を使用する場合、無効な状態であること' do
      user = build(:user, email: 'ｅｘａｐｌｅ@gmail.com')
      user.valid?
      expect(user.errors[:email]).to include("は不正な値です") #invalid
    end
  end

  describe 'password' do
    it { is_expected.to validate_presence_of :password }
    it { is_expected.to validate_length_of(:password).is_at_least(6).is_at_most(128) }
    it { is_expected.to validate_presence_of :password_confirmation }

    it 'passwordとpassword_confirmationが不一致の場合、無効な状態であること' do
      user = build(:user, password: 'password', password_confirmation: 'password_confirmation')
      user.valid?
      expect(user.errors[:password_confirmation]).to include("とパスワードの入力が一致しません") #confirmation
    end
  end
end

```
exampleを書く it '~' do end
実装
テストデータの作成方法(Factorybot、Fackerの導入)
DRYに書く(before, subject, letなど)
外部APIテスト
vcrの使用方法

