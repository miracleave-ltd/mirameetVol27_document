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
モデルスペックでは以下のような観点で実装をします。

- 有効な属性で初期化した場合、モデルが有効であることを検証する。
- 無効な属性で初期化した場合、モデルが有効ではないことを検証する。
- クラスメソッドとインスタンスメソッドが定義されている場合は期待通りに動作すること。

### ファイルの作成
まずはUserモデルを例にファイル作成をしていきます。

`ターミナル`
```ruby
rails g rspec:model user
```

`spec/models/user_spec.rb`というファイルが作成されていればOKです。

### Userモデルの要件
ではモデルスペックの観点を参考に、Userモデルでのテスト要件をまとめます。
先ほど作成した`user_spec.rb`ファイルに必要なテストを以下のように記載しました。

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

このようにテストを書き始める際はまずはどのような観点が必要なのか洗い出してexampleとしてまとめると実装がしやすいです。

上記の文法を説明していきます。

#### describe
期待する結果をまとめます。上記では `describe User`としていて、これがUserモデルのテストであると明示しています。
これはネストすることが可能で、`describe`の中に`describe`を実装することもできます。

#### it
実際のテストを実行するexampleを定義しています。
- 基本的にexample一つにつき一つの結果を期待しています。
- exampleは明示的に記載します。省略することもできますが、可読性が落ちるため、基本的には記述することが望ましいです。
- exampleの説明は動詞で始まります。例えば、`nicknameがnilの場合、無効であること`を英語に置き換えると`is invalid if nickname is nil`となり、動詞から始まっていることがわかります。


### テストを実装する
では、実際にテストを実装していきます。まずは、「有効な属性の場合のテスト」から実装しましょう。

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

まず、`it ~ do end`でexampleのブロックを作成ます。今回は有効な属性のテストのため、exampleとして`it "nickname, email, password, password_confirmationがあれば有効であること" do end`としています。こうすることで、このテストが何をテストするのか非常にわかりやすくなります。

続いて、`it`の中身を見てみましょう。最初にUserモデルのインスタンスを作成しています。
今回は、すべての属性が有効である場合のテストのため、全ての属性が有効であるインスタンスを作成します。
```ruby
user = User.new(
  nickname: 'Takashi',
  email: 'tester@example.com',
  password: 'p@ssword!!',
  password_confirmation: 'p@ssword!!',
)
```

最後に、本当にこのインスタンスが有効であるかどうかをチェックします。

```ruby
expect(user).to be_valid
```

上記が実際のテストの実行構文です。
まずはこのテストが通るかどうか実行してみます。

`ターミナル`
```ruby
bundle exec rspec spec/models/user_spec.rb
```

下記のようにターミナルに出力されればテスト成功です！

```ruby
User
  nickname, email, password, password_confirmationがあれば有効であること

Finished in 0.29163 seconds (files took 6.71 seconds to load)
1 example, 0 failures
```

では、`expect(user).to be_valid`が実際に何をやっているか一つずつ見ていきましょう。

#### expectメソッドとマッチャ
テストには`expect`メソッドを使用すします。expectとは日本語で「〜を期待する」という意味で、「テスト結果が〜になることを期待する」ということを表しています。
`expect`は引数をとり、その引数がどのような状態になっていることを期待するのかによってさまざまなマッチャと組み合わせて使用します。

マッチャとは、期待値と実際の値を比較して、一致したかもしくは一致していないかを返すオブジェクトのことです。
今回のケースでは`be_valid`というマッチャを使用していて、このマッチャは「〜は有効である」ことを示しています。
つまり、`expect(user).to be_valid`は「userインスタンスが有効であることを期待する」テストであるということです。

このようにRSpecのテストでは基本的に`expect`とマッチャを使用して、期待した結果が得られるかどうかテストしていきます。

ではここで、失敗テストも実行してみましょう。
「userインスタンスが有効であることを期待する」テストなので、失敗するテストとは、ここでは「userインスタンスが有効でないことを期待する」テストのこととします。
次のようにソースを変更します。

```ruby
expect(user).to_not be_valid
```

先ほどと比べると`to`が`to_not`となっています。
`to`が「〜となることを期待する」を示しているのに対し、`to_not`は「〜とならないことを期待する」を示しています。

この状態でもう一度テストを実行してみましょう。

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

上記の`Failure/Error: expect(user).to_not be_valid`のように、`expect(user).to_not be_valid`がエラーとなって、テストが失敗しています。


#### validationのテスト
続いてvalidationのテストを実装していきます。

今回のUserモデルでは下記のようなvalidationを実装しています。

```ruby
validates :nickname, presence: true, uniqueness: true, length: { maximum: 10 }
validates :password_confirmation, presence: true
```

また、このほかにユーザーの認証にはdeviseを使用しているため、deviseが設定しているvalidationが内部的に存在しています。
（例えばemailがnil or 空文字だと無効など)

ここではnicknameのvalidationテストを実装してみます。

nicknameのvalidationは下記の通り.

```ruby
validates :nickname, presence: true, uniqueness: true, length: { maximum: 10 }
```

- nilまたは空文字の場合は無効 (presence: true)
- 同じnicknameは保存できない (uniqueness: true)
- 文字数は最大で10文字

以上のことから、テストすべき観点は下記が想定されます。

```ruby
it "nicknameがnilの場合、無効であること"
it "nicknameが空文字の場合、無効であること"
it "nicknameが既に保存されている場合、無効であること"
it "nicknameが10文字以内の場合、有効であること"
it "nicknameが11文字以上の場合、無効であること"
```

では実際にテストを実装してみましょう。

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

テストを実行してみます。

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

となっていればテスト成功です。

まず、何をテストするかを明示的に示すため、`describe`を使用します。

```ruby
describe 'nickname' do
```

これでこのブロックはnicknameのテストを実装しているんだな、と理解できるようになります。
続いてexampleについて、'nilの場合、無効であること'を例に解説していきます。

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

以前のテスト同様に、まずはuserインスタンスを作成しますが、今回は「nicknameがnilの場合」をテストしたいため、
nicknameをnilの状態でインスタンスを作成します。

```ruby
user = User.new(
  nickname: nil,
  email: 'tester@example.com',
  password: 'p@ssword!!',
  password_confirmation: 'p@ssword!!',
)
```

そして`valid?`メソッドを使用してuserインスタンスが有効かどうかを判定します。

```ruby
user.valid?
```

`valid?`メソッドは対象のオブジェクトが有効な場合はtrue、無効な場合はfalseを返し、falseの場合はerrorsの中にエラー内容を格納します。
そのエラー内容が意図したものになっているかテストをしています。

```ruby
expect(user.errors[:nickname]).to include("を入力してください")
```

※今回のアプリはエラーを日本語化しているため、「を入力してください」という少しわかりづらいメッセージになっています。
英語の場合は「can't be blank」となります。

このテストで使用しているマッチャは`include`で、引数に取ったものがexpectの引数のものに含まれているか検証しています。

ここで一つ疑問が発生します。
> user.valid?でuserが無効な場合、falseを返すのであれば、「user.valid?がfalseであることを期待する」というテストではダメなのか？

下記のようなテストでも良いように見えますね。

```ruby
it '11文字以上の場合、無効であること' do
  user = User.new(
    nickname: 'TakashiKaii',     # 11文字
    email: 'tester@example.com',
    password: 'p@ssword!!',
    password_confirmation: 'p@ssword!!',
  )
  expect(user.valid?).to eq(false)
end
```

`expect`の引数に`user.valid?`をとり、`eq`マッチャを使用して`user.valid?`が`false`であることを期待するというテストです。
こちらの方が先ほどのテストの方よりも一行短くなります。しかも、このテストは通ります。

では下記の場合はどうなるか考えてみましょう。

```ruby
it '11文字以上の場合、無効であること' do
  user = User.new(
    nickname: 'TakashiKai',      #間違えて10文字
    email: '',　　                  #間違えてブランク
    password: 'p@ssword!!',
    password_confirmation: 'p@ssword!!',
  )
  expect(user.valid?).to eq(false)
end
```

（極端なケースですが）上記のテストはnicknameは11文字以上である場合をテストしたいのに10文字になっていますが、
emailが無効な値のため、`user.valid?`は`false`になり、テストが通ってしまいます。

これでは「11文字以上の場合、無効であること」をテストしたことにはなりません。

今回のアプリのUserモデルのようなテストであれば記述量は少ないため、発生しにくいですが、アプリが巨大化するにつれてこのようなテスト自体のミスは必ず増えてきます。
そのときに、テストしようと思ったことが実はテストできておらず、しかもテストが通ってしまうという状況は品質低下につながるため要注意です。

今回の例では適切にエラーを検証するとテストが失敗するため、すぐに異変に気づくことができます。

```ruby
it '11文字以上の場合、無効であること' do
  user = User.new(
    nickname: 'TakashiKai',      #間違えて10文字
    email: '',　　                  #間違えてブランク
    password: 'p@ssword!!',
    password_confirmation: 'p@ssword!!',
  )
  user.valid?
  expect(user.errors[:nickname]).to include("は10文字以内で入力してください")
end
```

テストを実行すると下記のように失敗します。

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

`expected [] to include "は10文字以内で入力してください"`とあるように`user.errors[:nickname]`の中身が空のためテストが失敗しています。
（emailでエラーになっているため、`user.errors[:nickname]`は空で`user.errors[:email]`の中にエラーが格納されています）


### DRYに書く
ここまでのテストを見てきて「冗長だな」と感じた方もいらっしゃると思います。
ではRSpecをDRYに書く方法をいくつか紹介していきます。

#### before
今回のテストではexampleの中で何度も`User.new`が登場します。
これを共有化して使用するためには`before`を使用します。

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
`before`は処理を定義すると、その処理が各exampleの実行前に実行されます。
今回の場合はUserインスタンスを作成して`@user`に代入しています。
テストを実行すると、各exampleの実行前に`@user`にインスタンスが代入されるため、example内で`@user`を呼び出すことができます。
あとは各exampleに適したテストを実行するだけです。

`before`のスコープはdescribeやまだ出てきていませんがcontextの中になります。
今回は最上位のdescribe内で定義しているため、すべてのexampleから`@user`にアクセスできますが、

これを下記のように書き換えてみます。

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

このように`describe 'nickname' do`内に`before`を定義するとこの`describe`内からしかアクセスできなくなります。
そのため、下記テストは失敗します。

```ruby
it "nickname, email, password, password_confirmationがあれば有効であること" do
  expect(@user).to be_valid
end
```

#### let
続いて`let`です。変数を作成するメソッドになりますが、変数を作成するタイミングが特徴的です。
宣言した時には作成せず、その変数を呼び出した時に初めて作成されます。

先ほどの`before`を`let`に置き換えてみます。

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  let(:user) {
    User.new(
      nickname: 'Takashi',
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
  }

  it "nickname, email, password, password_confirmationがあれば有効であること" do
    expect(user).to be_valid
  end

  describe 'nickname' do
    it 'nilの場合、無効であること' do
      user.nickname = nil
      expect(user.valid?).to eq(false)
    end

    it '空文字の場合、無効であること' do
      user.nickname = ""
      user.valid?
      expect(user.errors[:nickname]).to include("を入力してください")
    end

    it 'すでに使用されているnicknameの場合、保存できないこと' do
      user.save
      new_user = User.new(
        nickname: 'Takashi',
        email: 'tester_2@example.com',
        password: 'p@ssword!!',
        password_confirmation: 'p@ssword!!',
      )
      new_user.valid?
      expect(new_user.errors[:nickname]).to include("はすでに存在します")
    end

    it '10文字以内の場合、有効であること' do
      user.nickname = 'TakashiKai'
      expect(user).to be_valid
    end

    it '11文字以上の場合、無効であること' do
      user.nickname = 'TakashiKaii'
      user.valid?
      expect(user.errors[:nickname]).to include("は10文字以内で入力してください")
    end
  end
end

```

上記のように置き換えることができました。
では、`before`と`let`の違いはなんでしょうか？

`before`は各exampleの前で必ず実行されます。
対して`let`は宣言している箇所ではまだ処理が実行されず、変数が呼び出されて初めて、処理が実行されます。

```ruby
  let(:user) {
    User.new(
      nickname: 'Takashi',
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
  }
```
上記のように`let`を宣言していますが、このときにはまだ`user`という変数には何も値が入っていません。

```ruby
  it "nickname, email, password, password_confirmationがあれば有効であること" do
    expect(user).to be_valid
  end
```
上記の`expect(user).to be_valid`にて、`user`が呼び出されたタイミングで初めて`let`で宣言した`user`変数に値が格納されます。

つまり、`before`では（スコープはありますが）すべてのexampleの実行前に呼び出されますが、`let`は変数を呼び出して初めて実行されるため、
`let`を使用すると不必要なデータを作成せずに済みます。

`before`で処理する内容はすべてのexampleで必要な処理を定義すべきです。例えば、「ユーザーがログインしている場合」の処理を実行したいのであれば、
そのグループのexampleでは必ずログイン状態を作成する必要があるため、

```ruby
before do
  sign_in user
end
```

のようにすることができます。（詳細は割愛しますが、`sign_in`メソッドはdeviseのメソッドです。テストでも設定ファイルに記述が必要ですが使用することができます。）

#### shoulda-matchers
これはexampleを簡素に記述することができるようになる外部ライブラリです。
まずは導入しましょう。

Gemfile
```ruby
group :test do

  ## 省略
  
  gem 'shoulda-matchers'
end
```

ターミナル
```ruby
bundle install
```

rails_helperに設定を記述します。

spec/rails_helper.rb
```ruby
## 省略

Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end

```

これで準備が整いました。

Userモデルのスペックを例にリファクタリングしてみましょう。

以前、nicknameのvalidationテストは下記のように書いていました。

```ruby
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

こちらを`shoulda-matchers`を使用して記述すると下記のようになります。

```ruby
  describe 'nickname' do
    it { is_expected.to validate_presence_of :nickname }
    it { is_expected.to validate_uniqueness_of :nickname }
    it { is_expected.to validate_length_of(:nickname).is_at_most(10) }
  end
```

なんとexampleがたったの３行になってしまいました！！

`is_expected`でUserのインスタンスを作成しています。
`validate_presence_of`は、引数である`:nickname`がnilまたはブランクだったときに無効であるかをチェックしています。
`validate_uniqueness_of`は、同じくnicknameが同一のデータが作成できないことをチェックしていて、
`validate_length_of(:nickname).is_at_most(10)`は10文字以内は可、11文字以上は作成できないことをチェックしています。

本当にテストできているか確かめるには、`to`を`to_not`に置き換えて実行してみましょう。
例えば`it { is_expected.to validate_length_of(:nickname).is_at_most(10) }`を`it { is_expected.to_not validate_length_of(:nickname).is_at_most(10) }`にして実行すると下記のようなエラーになります。

```ruby
   Expected User not to validate that the length of :nickname is at most
       10, but this could not be proved.
         After setting :nickname to ‹"xxxxxxxxxx"›, the matcher expected the
         User to be invalid, but it was valid instead.
```

`‹"xxxxxxxxxx"›`を見るとマッチャが自動的に10文字の文字列を作成してインスタンスを作成していることがわかります。
この10文字を当てはめて実行したとき、`User to be invalid, but it was valid instead.`とあるように
Userインスタンスは作成できないことを期待していますが、正常に作成できてしまっています。

つまり、`it { is_expected.to validate_length_of(:nickname).is_at_most(10) }`は正常に働いており、正しくテストできていると言えます。
`.is_at_most(10)`の数字を9や11などにしたときにどのように動作するか確かめてもいいかもしれません。

他にもたくさんのマッチャがあるので是非下記を参考にしてみてください。
https://github.com/thoughtbot/shoulda-matchers

では一度ここまでの知識で`user_spec.rb`をリファクタリングしてみましょう。

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  it "nickname, email, password, password_confirmationがあれば有効であること" do
    user = User.new(
      nickname: 'Takashi',
      email: 'tester@example.com',
      password: 'p@ssword!!',
      password_confirmation: 'p@ssword!!',
    )
    expect(user).to be_valid
  end

  describe 'アソシエーション' do
    it { is_expected.to have_many :posts }
    it { is_expected.to have_many :comments }
  end

  describe 'nickname' do
    it { is_expected.to validate_presence_of :nickname }
    it { is_expected.to validate_uniqueness_of :nickname }
    it { is_expected.to validate_length_of(:nickname).is_at_most(10) }
  end

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

非常に簡素にまとまりましたね！！


### テストデータの作成方法
テストを作成するにあたり、テストデータの作成は非常に重要です。
テストが膨大になればなるほどテストデータの作成コストはどんどん膨らんでいきます。
そこで、RailsではFactorybotというライブラリを使用して簡単にテストデータを作成する方法が用意されています。

まずはFactorybotを導入します。

Gemfile
```ruby
group :development, :test do

  ## 省略

  gem 'factory_bot_rails'
end
```

ターミナル
```ruby
bundle install
```

続いてジェネレータを使用してファイルを作成していきます。Userモデルのテストデータを作成したい場合は下記を実行します。

ターミナル
```ruby
rails g factory_bot:model user
```

するとspec/factories/users.rbというファイルが作成されます。
例えば下記のように記述します。

```ruby
FactoryBot.define do
  factory :user do
    nickname { "Takashi" }
    email { "example@gmail.com" }
    password { "password" }
    password_confirmation { password }
  end
end

```

では実際にPostモデルスペックを例にこのテストデータを使用してみます。

```ruby
describe Post do
  let(:user) { create(:user) }

  it 'text, userがあれば有効であること' do
    post = Post.new(
      text: '投稿のテキスト',
      user: user
    )
    expect(user).to be_valid
  end
  
  
  ## 省略
  
end
```

上記のように`create`メソッドを使用するとfactoryのデータを参照して作成することができます。
`create`は引数にシンボルをとり、この値は、`spec/factories`配下のファイル内の`factory :user do`を見つけて作成しています。

つまり、今回のケースでは`spec/factories/users.rb`で`factory :user do`と定義されているため、
`post_spec.rb`にて`create(:user)`とすると`let(:user)`の変数に

```ruby
User.new(
  nickname: "Takashi",
  email: "example@gmail.com"
  password: "password" 
  password_confirmation: "password" 
)
```
が代入されることになります。

また、`create(:user)`とするとspec/factories/users.rbの定義をもとにuserインスタンスを作成しますが、データを上書きすることもできます。
例えばnicknameを違う値にしたい時は

```ruby
let(:user) { create(:user, nickname: "Kato") }
```
とすることで、"Takashi"ではなく"Kato"で作成されます。

※詳細は割愛しますが今回のアプリではFactorybotでデータ作成する際に、併せてFakerというライブラリを使用しています。
上記では毎回同じテストデータになってしまいますが、Fakerを使用するとランダムな値を簡単に作成することができたり、
email形式やurl形式のデータを簡単に作成してくれるため、非常に便利です。
詳細は下記GitHubをご覧ください。

https://github.com/faker-ruby/faker


## 3. 外部APIテスト

今回のアプリではスラックと連携しており、投稿したときに自動的にスラックのチャネルにチャットを投下する機能があります。

- メッセージを投下する(Post.create）
- Postモデルの`after_save :start_slack_sync`が呼ばれてjobが実行される
- jobがPostモデルの`slack_sync!`メソッドを呼び出す。
- `Slack::SendPostService`の`send_post`メソッドが実行されてスラックにメッセージを投下する。

という流れです。
今回は外部APIテストとして`Slack::SendPostService`のテストを実装します。

※詳細は割愛しますが、スラックに連携する際に`slack-notifier`というライブラリを使用しています。詳しく知りたい方は下記を参考にしてください。
https://github.com/slack-notifier/slack-notifier

#### VCRの導入

外部APIの連携テストの際はスタブを使用することが多いと思います。
簡単にスタブを作成できるVCRというライブラリがあるのでご紹介します。

ではまずは導入していきます。

Gemfile
```ruby
group :test do

  ## 省略
  
  gem 'webmock'
  gem 'vcr'
end
```

VCRを使用する際は内部的にwebmockというライブラリも使用するため、一緒にインストールします。

ターミナル
```ruby
bundle install
```

spec_helper.rbを編集して設定を記述していきます。

spec/spec_helper.rb
```ruby
require 'vcr'
require 'webmock'

RSpec.configure do |config|
 ## 省略
end

VCR.configure do |c|
  c.cassette_library_dir = 'spec/vcr'
  c.hook_into :webmock
  c.configure_rspec_metadata!
end
```

これで準備が整いました！

ではテストを実装していきます。

今回テストする処理は`app/servise/slack`配下の`send_post_service.rb`というファイルです。
そのため、テストは`spec/servise/send_post_service_spec.rb`に作成します。

```ruby
require 'rails_helper'

describe Slack::SendPostService do
  describe 'slack alignment' do
    context "#send_post" do
      it 'post to slack' do
      
      ## テストを書く
      
      end
    end
  end
end

```

まずは上記のように外枠を書いてみました。
テストする箇所は`describe Slack::SendPostService do`として明示しています。
テストする内容は`describe 'slack alignment' do`としていて、スラックとの連携テストだよと記載しています。
`context "#send_post" do`では`send_post`メソッドのテストを実装することを示しています。
あるメソッドをテストする時は、慣習的に`#メソッド名`とすることが多いです。
最後にexampleを`it 'post to slack' do`としています。

では実際にexampleの中身を書いていきましょう。
その前に簡単にSendPostServiceの中身を見てみましょう。

```ruby
module  Slack
  class SendPostService

    def initialize(user_name)
      webhook_url = ENV['SLACK_WEBHOOK_URL']
      channel = "##{ENV['CHANNEL_NAME']}"
      @client ||= Slack::Notifier.new(webhook_url, channel: channel, username: user_name)
    end

    def send_post(message)
      @client.ping(message)
    end
  end
end
```

`Slack::SendPostService`をinitializeするときにuserのnicknameを引数に取り、`@client`をを作成します。
これは先ほどの`slack-notifier`ライブラリを使用していますが、`Slack::Notifier.new`インスタンスであり、
引数にスラックのwebhook_url、メッセージを投下するチャンネル名、そしてusernameを指定しています。

そしてこの`@client`に`ping`メソッドを使用すると、該当のスラックのチャンネルにメッセージを投下するという仕組みです。

それを踏まえてテストを実装すると下記のようになります。

```ruby
it 'post to slack', :vcr do
  response = Slack::SendPostService.new('user_nickname').send_post('test_message')
  expect(response.first.message).to eq("OK")
end
```

まず、`response = Slack::SendPostService.new('user_nickname').send_post('test_message')`で`Slack::SendPostService`のインスタンスを作成して、`send_post`メソッドを呼び出しています。これでスラックにメッセージを投下することができます。

投下した後はAPIからレスポンスが返されるため、それを`response`に代入しています。

メッセージの投下が成功すれば`response`の中に"OK"というメッセージが含まれるはずなので`expect(response.first.message).to eq("OK")`として検証をしています。

ではテストを実行してみましょう。
（事前にSlack APIの設定およびenvファイルの作成と環境変数の設定が必要です）

ターミナル
```ruby
 rspec spec/service/send_post_service_spec.rb 
```

正常に終了すれば下記のようなログが出ると思います。
```ruby
Slack::SendPostService
  slack alignment
    #send_post
      post to slack

Finished in 0.52359 seconds (files took 4.99 seconds to load)
1 example, 0 failures
```

お気づきの方もいらっしゃるかもしれませんが、先ほどのexampleを作成したときに、`:vcr`と記述しています。

```ruby
it 'post to slack', :vcr do
  response = Slack::SendPostService.new('user_nickname').send_post('test_message')
  expect(response.first.message).to eq("OK")
end
```

`it 'post to slack', :vcr do`の箇所です。こうすることでvcrを使用することができます。
この状態でテストを実行すると、spec配下に`vcr/Slack_SendPostService/slack_alignment/_send_post/post_to_slack.yml`というファイルが自動生成されると思います。

中身はこのようになっています。
```ruby
---
http_interactions:
- request:
    method: post
    uri: https://hooks.slack.com/services/*************
    body:
      encoding: US-ASCII
      string: payload=***************
    headers:
      Accept-Encoding:
      - gzip;q=1.0,deflate;q=0.6,identity;q=0.3
      Accept:
      - "*/*"
      User-Agent:
      - Ruby
      Content-Type:
      - application/x-www-form-urlencoded
  response:
    status:
      code: 200
      message: OK
    headers:
      Date:
      - Mon, 28 Mar 2022 09:25:14 GMT
      Server:
      - Apache
      X-Powered-By:
      - HHVM/4.153.0
      X-Frame-Options:
      - SAMEORIGIN
      Access-Control-Allow-Origin:
      - "*"
      Referrer-Policy:
      - no-referrer
      X-Slack-Backend:
      - r
      Strict-Transport-Security:
      - max-age=31536000; includeSubDomains; preload
      Vary:
      - Accept-Encoding
      Content-Length:
      - '22'
      Content-Type:
      - text/html
      X-Envoy-Upstream-Service-Time:
      - '192'
      X-Backend:
      - main_normal main_bedrock_normal_with_overflow main_canary_with_overflow main_bedrock_canary_with_overflow
        main_control_with_overflow main_bedrock_control_with_overflow
      X-Server:
      - slack-www-hhvm-main-iad-98d1
      X-Slack-Shared-Secret-Outcome:
      - no-match
      Via:
      - envoy-www-iad-9tfu, envoy-edge-nrt-3w6t
      X-Edge-Backend:
      - envoy-www
      X-Slack-Edge-Shared-Secret-Outcome:
      - no-match
    body:
      encoding: ASCII-8BIT
      string: ok
  recorded_at: Mon, 28 Mar 2022 09:25:14 GMT
recorded_with: VCR 6.0.0

```

これはテストを実行した際に、スラックAPIを実際に叩いて通信をしていて、その時のリクエストとレスポンスをymlファイルとしてまとめたものになります。
このファイルが作成された後にもう一度テストすると、今度はスラックAPIを叩くことなく、このファイルをもとにレスポンスを返してくれます。

つまり、VCRは、初回のみ実際にAPIにリクエストを行い、そのリクエストと、レスポンスをもとにymlファイルを自動生成し、以降のテストはそれをスタブとして値を返すというものです。
そのため、テストのたびに余計なリクエストを投げることがなくなります。

また、自分でスタブを実装する必要がなく、自動で生成してくれるため非常に便利です。

ただし、初回だけ通信が必要になるため、その通信環境が整っていなかったり、何かしらの理由でリクエストが失敗してしまったりすると、その状態でymlファイルを作成してしまうため、
正しくテストができない可能性があります。その場合は従来通りwebmock等を使用したスタブを自身で作成することを検討する必要があります。

何かしらの理由で想定外のレスポンスが返ってきた場合、vcrディレクトリを削除して再度実施すれば、もう一度実際にリクエストをしてファイルを自動生成してくれます。



### まとめ

単体テストの実装をモデル・外部APIを例に実装してみました。またリファクタリングのテクニックについてもご紹介しました。
こちらがRSpecの基礎となっていくので是非抑えていただいて以降のテストに活かしていただければと思います。




