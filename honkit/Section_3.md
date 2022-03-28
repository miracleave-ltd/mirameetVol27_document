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


#### テストデータの作成方法
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

するとspec/factories/user.rbというファイルが作成されます。
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

つまり、今回のケースでは`spec/factories/user.rb`で`factory :user do`と定義されているため、
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

※詳細は割愛しますが今回のアプリではFactorybotでデータ作成する際に、併せてFakerというライブラリを使用しています。
上記では毎回同じテストデータになってしまいますが、Fakerを使用するとランダムな値を簡単に作成することができたり、
email形式やurl形式のデータを簡単に作成してくれるため、非常に便利です。
詳細は下記GitHubをご覧ください。

https://github.com/faker-ruby/faker

テストデータの作成方法(Factorybot、Fackerの導入)
外部APIテスト
vcrの使用方法

