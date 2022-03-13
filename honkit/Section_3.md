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
  it "nicknameが既に保存されている場合、無効であること"
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
- exampleの説明は動詞で始まる。例えば、`nicknameがnilの場合、無効であること`を英語に置き換えると`is invalid if nickname is nil`となる。


### テストを実装する
#### 有効な属性の場合のテスト

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  # 有効な属性の場合のテスト
  it "nickname, email, password, password_confirmationがあれば有効であること" do
    user = User.new(
      nickname: 'Takashi',
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
    expect(user).to be_valid
  end
end

```

`it ~ do end`でexampleのブロックを作成する。今回は有効な属性のテストのため、exampleとして`it "nickname, email, password, password_confirmationがあれば有効であること" do end`とする。

すべての属性が有効である場合のテストのため、全ての属性が有効であるインスタンスを作成する。
```ruby
user = User.new(
  nickname: 'Takashi',
  email: 'tester@example.com',
  password: 'p@ssword!!',
  password_confirmation: 'p@ssword!!',
)
```

そして、本当にこのインスタンスが有効であるかどうかをチェックする。
```ruby
expect(user).to be_valid
```

このテストでは`be_valid`というマッチャを使用して`user`が有効であるかどうかをテストしている。
下記コマンドでテストを実行してみる。

`ターミナル`
```ruby
bundle exec rspec spec/models/user_spec.rb
```

下記のようにターミナルに出力されればテスト成功
```ruby
User
  nickname, email, password, password_confirmationがあれば有効であること

Finished in 0.29163 seconds (files took 6.71 seconds to load)
1 example, 0 failures
```

#### expectメソッドとマッチャ
テストには`expect`メソッドを使用する。expectとは日本語で「〜を期待する」という意味で、「テスト結果が〜になることを期待する」ということ。
`expect`は引数をとり、その引数がどのような状態になっていること期待するのかによってさまざまなマッチャと組み合わせて使用する。

マッチャとは、期待値と実際の値を比較して、一致したかもしくは一致していないかを返すオブジェクトのこと。
今回のケースでは`be_valid`というマッチャを使用していて、このマッチャは「〜は有効である」ことを示している。
つまり、`expect(user).to be_valid`は「userインスタンスが有効であることを期待する」テストであるということ。

ここで、失敗テストを実行してみる。
「userインスタンスが有効であることを期待する」テストのため、失敗するテストとは、ここでは「userインスタンスが有効でないことを期待する」テストのこと。
次のようにソースを変更する。

```ruby
expect(user).to_not be_valid
```

この状態でもう一度テストを実行する

`ターミナル`
```ruby
bundle exec rspec spec/models/user_spec.rb
```

下記のようにターミナルに出力されれば期待通りのテスト結果。
```ruby
User
  nickname, email, password, password_confirmationがあれば有効であること (FAILED - 1)

Failures:

  1) User nickname, email, password, password_confirmationがあれば有効であること
     Failure/Error: expect(user).to_not be_valid
       expected #<User id: nil, email: "tester@example.com", nickname: "Takashi", created_at: nil, updated_at: nil> not to be valid
     # ./spec/models/user_spec.rb:11:in `block (2 levels) in <top (required)>'

Finished in 0.09446 seconds (files took 4.89 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/models/user_spec.rb:4 # User nickname, email, password, password_confirmationがあれば有効であること
```

上記の`Failure/Error: expect(user).to_not be_valid`のように、`expect(user).to_not be_valid`がエラーとなって、テストが失敗している。


#### validationのテスト
今回のUserモデルでは下記のようなvalidationを実装している。
```ruby
validates :nickname, presence: true, uniqueness: true, length: { maximum: 10 }
validates :password_confirmation, presence: true
```
また、このほかにユーザーの認証にはdeviseを使用しているため、（詳細は割愛するが）deviseが設定しているvalidationが内部的に存在している。
（例えばemailがnil or 空文字だと無効など)

ここではnicknameのvalidationテストを実装する。

nicknameのvalidationは下記の通り.

```ruby
validates :nickname, presence: true, uniqueness: true, length: { maximum: 10 }
```

- nilまたは空文字の場合は無効 (presence: true)
- 同じnicknameは保存できない (uniqueness: true)
- 文字数は最大で10文字

以上のことから、テストすべき観点は下記の通り。

```ruby
it "nicknameがnilの場合、無効であること"
it "nicknameが空文字の場合、無効であること"
it "nicknameが既に保存されている場合、無効であること"
it "nicknameが10文字以内の場合、有効であること"
it "nicknameが11文字以上の場合、無効であること"
```

テストを実装すると下記の通り。

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  # 有効な属性の場合のテスト

  ## 省略

 describe 'nickname' do
  it 'nilの場合、無効であること' do
    user = User.new(
      nickname: nil,
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
    user.valid?
    expect(user.errors[:nickname]).to include("を入力してください")
  end

  it '空文字の場合、無効であること' do
    user = User.new(
      nickname: "",
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
    user.valid?
    expect(user.errors[:nickname]).to include("を入力してください")
  end

  it 'すでに使用されているnicknameの場合、保存できないこと' do
    User.create(
      nickname: 'Takashi',
      email: 'tester_1@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )

    user = User.new(
      nickname: 'Takashi',
      email: 'tester_2@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
    user.valid?
    expect(user.errors[:nickname]).to include("はすでに存在します")
  end

  it '10文字以内の場合、有効であること' do
    user = User.new(
      nickname: 'TakashiKai',
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
    expect(user).to be_valid
  end

  it '11文字以上の場合、無効であること' do
    user = User.new(
      nickname: 'TakashiKaii',
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
    user.valid?
    expect(user.errors[:nickname]).to include("は10文字以内で入力してください")
  end
 end
end

```

テストを実行する。

`ターミナル`
```ruby
bundle exec rspec spec/models/user_spec.rb
```

```ruby
User
  nickname
    nilの場合、無効であること
    空文字の場合、無効であること
    すでに使用されているnicknameの場合、保存できないこと
    10文字以内の場合、有効であること
    11文字以上の場合、無効であること

Finished in 1.19 seconds (files took 4.51 seconds to load)
5 examples, 0 failures
```

となっていればテスト成功。

まず、何をテストするかを明示的に示すため、`describe`を使用する。

```ruby
describe 'nickname' do
```

これでこのブロックはnicknameのテストを実装しているんだな、と理解できるようになる。
続いてexampleについて、'nilの場合、無効であること'を例に解説する。

```ruby
it 'nilの場合、無効であること' do
  user = User.new(
    nickname: nil,
    email: 'tester@example.com',
    password: 'p@ssword!!',
    password_confirmation: 'p@ssword!!',
  )
  user.valid?
  expect(user.errors[:nickname]).to include("を入力してください")
end
```

以前のテスト同様に、まずはuserインスタンスを作成するが、今回は「nicknameがnilの場合」をテストしたいため、
nicknameをnilの状態でインスタンスを作成する。

```ruby
user = User.new(
  nickname: nil,
  email: 'tester@example.com',
  password: 'p@ssword!!',
  password_confirmation: 'p@ssword!!',
)
```

そして`valid?`メソッドを使用してuserインスタンスが有効かどうかを返している。

```ruby
user.valid?
```

`valid?`メソッドは対象のオブジェクトが有効な場合はtrue、無効な場合はfalseを返し、errorsの中にエラー内容を格納する。
そのエラー内容が意図したものになっているかテストをしている。

```ruby
expect(user.errors[:nickname]).to include("を入力してください")
```

※今回のアプリはエラーを日本語化しているため、「を入力してください」という少しわかりづらいメッセージになっている。
英語の場合は「can't be blank」となる。

このテストで使用しているマッチャは`include`で、引数に取ったものがexpectの引数のものと一致するか検証している。

ここでなぜエラー内容を検証しているか「11文字以上の場合、無効であること」のテストを例に考えてみる。
「無効であること」をテストするのであれば、下記のようなテストでも良いように見える。

```ruby
it '11文字以上の場合、無効であること' do
  user = User.new(
    nickname: 'TakashiKaii',
    email: 'tester@example.com',
    password: 'p@ssword!!',
    password_confirmation: 'p@ssword!!',
  )
  expect(user.valid?).to eq(false)
end
```

`expect`の引数に`user.valid?`をとり、`eq`マッチャを使用して`user.valid?`が`false`であることを期待するというテスト。
実際にこのテストは通る。

では下記の場合はどうなるか考えてみる。

```ruby
it '11文字以上の場合、無効であること' do
  user = User.new(
    nickname: 'TakashiKai',
    email: '',
    password: 'p@ssword!!',
    password_confirmation: 'p@ssword!!',
  )
  expect(user.valid?).to eq(false)
end
```

（極端なケースだが）上記のテストはnicknameは11文字以上である場合をテストしたいのに10文字になっているが、
emailが無効な値のため、`user.valid?`は`false`になり、テストが通ってしまう。

これでは「11文字以上の場合、無効であること」をテストしたことにはならない。

今回のアプリのUserモデルのようなテストであれば記述量は少ないため、発生しにくいが、アプリが巨大化するにつれてこのようなミスは必ず増えてくる。
そのときに、テストしようと思ったことが実はテストできておらず、しかもテストが通ってしまうという状況は品質低下につながるため要注意。

今回の例では適切にエラーを検証するとテストが失敗するため、すぐに異変に気づくことができる。

```ruby
it '11文字以上の場合、無効であること' do
  user = User.new(
    nickname: 'TakashiKai',
    email: '',
    password: 'p@ssword!!',
    password_confirmation: 'p@ssword!!',
  )
  user.valid?
  expect(user.errors[:nickname]).to include("は10文字以内で入力してください")
end
```

テストを実行すると下記のように失敗する。

```ruby
User
  nickname
    11文字以上の場合、無効であること (FAILED - 1)

Failures:

  1) User nickname 11文字以上の場合、無効であること
     Failure/Error: expect(user.errors[:nickname]).to include("は10文字以内で入力してください")
       expected [] to include "は10文字以内で入力してください"
     # ./spec/models/user_spec.rb:66:in `block (3 levels) in <top (required)>'

Finished in 0.67955 seconds (files took 4.55 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/models/user_spec.rb:58 # User nickname 11文字以上の場合、無効であること
```

`expected [] to include "は10文字以内で入力してください"`とあるように`user.errors[:nickname]`の中身が空のためエラーとなっている。
（emailでエラーになっているため、`user.errors[:nickname]`は空で`user.errors[:email]`の中にエラーが格納されている）


### DRYに書く
ここまでのテストを見てきて「冗長だな」と感じた方もいらっしゃると思います。
ではRSpecでDRYに書く方法を紹介していきます。

今回のテストではexampleの中で何度も`User.new`が登場します。
これを共有化して使用するためにはbeforeを使用します。

まずはソースコードをこのように変更します。

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  before do
    @user = User.new(
      nickname: 'Takashi',
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
  end

  it "nickname, email, password, password_confirmationがあれば有効であること" do
    expect(@user).to be_valid
  end

  describe 'nickname' do
    it 'nilの場合、無効であること' do
      @user.nickname = nil
      expect(@user.errors[:nickname]).to include("を入力してください")
    end

    it '空文字の場合、無効であること' do
      @user.nickname = ""
      @user.valid?
      expect(@user.errors[:nickname]).to include("を入力してください")
    end

    it 'すでに使用されているnicknameの場合、保存できないこと' do
      @user.save
      user = User.new(
        nickname: 'Takashi',
        email: 'tester_2@example.com',
        password: 'p@ssword!!',
        password_confirmation: 'p@ssword!!',
      )
      user.valid?
      expect(user.errors[:nickname]).to include("はすでに存在します")
    end

    it '10文字以内の場合、有効であること' do
      @user.nickname = 'TakashiKai'
      expect(@user).to be_valid
    end

    it '11文字以上の場合、無効であること' do
      @user.nickname = 'TakashiKaii'
      @user.valid?
      expect(@user.errors[:nickname]).to include("は10文字以内で入力してください")
    end
  end
end

```
`before`とは処理を定義すると、各exampleの実行前に実行されます。
今回の場合はUserインスタンスを作成して`@user`に代入しています。
テストを実行すると、各exampleの実行前に`@user`にインスタンスが代入されるため、example内で`@user`を呼び出すことができます。
あとは各exampleに適したテストを実行するだけです。

`before`のスコープはdescribeやまだ出てきていませんがcontextの中になります。今回は最上位のdescribe内で定義しているため、
すべてのexampleから`@user`にアクセスできますが、これを

```ruby
## 省略

describe 'nickname' do
  before do
    @user = User.new(
      nickname: 'Takashi',
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
  end
  
## 省略

```

のようにすると`@user`は`describe 'nickname' do`内からしかアクセスできなくなります。
そのため、下記テストは失敗します。

```ruby
it "nickname, email, password, password_confirmationがあれば有効であること" do
  expect(@user).to be_valid
end
```


let let! subject




テストデータの作成方法(Factorybot、Fackerの導入)
外部APIテスト
vcrの使用方法

