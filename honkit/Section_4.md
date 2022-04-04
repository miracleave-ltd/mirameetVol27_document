# 結合テスト（IT）
## 本セクションで行うこと
- 今回取り上げる結合テストについて
- Request specの解説


## 今回取り上げる結合テストについて

結合テストは、Integration Test(統合テスト)とも呼ばれ、システム開発において単体レベルで開発した機能が組み合わさって動作した時、期待される動作をするかどうかをテストします。  
結合テストは、実施する対象によって内部結合・外部結合に分かれます。  

**内部結合**・・・ システム内部の機能同士を組み合わせてテストを行う  
**外部結合**・・・ システム内部の機能とシステム外部の機能を組み合わせてテストを行う

今回取り上げる結合テストは「内部結合」になります
![スクリーンショット 2022-04-04 22 13 44](https://user-images.githubusercontent.com/52161269/161551626-58658fa8-95e5-4152-a220-d5a096aa8bd0.png)


今回の投稿アプリでは、以下３つの機能があります。
1. ユーザー機能(users_controller)  
  ユーザー作成、ユーザー認証（ログイン・ログアウト）
2. 投稿機能(posts_controller)  
  投稿の新規作成、投稿の編集、投稿の一覧表示、投稿削除
3. コメント機能(comments_controller)  
  対象の投稿に対してコメントの作成

内部結合では、３つの機能の間の連携部分が期待される動きをするか？というテストをします。
具体的な観点は以下です。
- コントローラーへのリクエストが通ること
- 期待されるレスポンスが返却されること


## Request specの解説

Request specを使うと、以下のテストができるようになります。

- HTTPリクエストのテストができる
- 複数コントローラーの複数のリクエストがテストできる
- 複数のセッションでリクエストを指定できる


以下、Request specファイルの生成コマンド
```
rails g rspec:request ファイル名
```

投稿機能の結合テスト(posts_spec.rb)を例に実際のテストコードを確認していきます。  

```
require 'rails_helper'

RSpec.describe 'Posts', type: :request do
~~~~~~~~~~~①~~~~~~~~~~~~
  let(:user) { create(:user, nickname: 'Takashi') }
  let(:post_instance) { create(:post, user: user, text: 'PostRequestTest', image: 'https://example_image_url') }
~~~~~~~~~~~①~~~~~~~~~~~~

  describe '投稿一覧' do
~~~~~~~~~~~②~~~~~~~~~~~~
    subject { get posts_url }
~~~~~~~~~~~②~~~~~~~~~~~~

    context 'ログインしている場合' do
~~~~~~~~~~~③~~~~~~~~~~~~
      before do
        sign_in user
        post_instance
      end
~~~~~~~~~~~③~~~~~~~~~~~~

~~~~~~~~~~~④~~~~~~~~~~~~
      it_behaves_like 'return_response_status', 200
~~~~~~~~~~~④~~~~~~~~~~~~
      

      it 'レスポンスに適切な投稿内容を含むこと' do
        subject
        aggregate_failures do
          expect(response.body).to include 'PostRequestTest :Takashiの投稿'
          expect(response.body).to include '<span>投稿者</span>Takashi'
        end
      end
    end


  ~~~~省略~~~~~
```
***
##### 3.1 ユーザーがログインしている場合は投稿一覧のリクエストが成功すること

①テストデータとして、ログインユーザーと投稿を定義  
②subject(テスト対象のオブジェクト)は、投稿一覧のリクエスト処理  
　Request specでは直接ルーティング名を指定することができます。  
③テスト前にテストデータのユーザーでログイン処理とlet（post_instance）を呼び出しておく  
④リクエストが通ることを確認するので、HTTPレスポンスのステータスが200であることが期待値  
`it_behaves_like`は、新しいコンテキスト(context 'xxx'do ... endのこと)を自動生成して、そこにテストケースを埋め込みます。  
`return_response_status`は、同じテストコードの重複を避けるために`shared_examples`を使ってまとめています。  
中身のテストコードは、別ファイルで定義しています。  
`mirameetVol27/spec/support/examples/return_response_status.rb`  
```
RSpec.shared_examples 'return_response_status' do |status_no|
  it "#{status_no}レスポンスを返すこと" do
    subject
    expect(response.status).to eq status_no
  end
end
```

`shared_examples`なしのコード：
```
context 'behaves like a return_response_status' do
    it "200レスポンスを返すこと" do
        expect(response.status).to eq 200
    end
end
```

***
##### 3.2 投稿一覧のレスポンスに適切な投稿内容を含んでいること

```
require 'rails_helper'

RSpec.describe 'Posts', type: :request do
  let(:user) { create(:user, nickname: 'Takashi') }
  let(:post_instance) { create(:post, user: user, text: 'PostRequestTest', image: 'https://example_image_url') }

  describe '投稿一覧' do
    subject { get posts_url }

    context 'ログインしている場合' do
      before do
        sign_in user
        post_instance
      end

      it_behaves_like 'return_response_status', 200
      

      it 'レスポンスに適切な投稿内容を含むこと' do
~~~~~~~~~~~①~~~~~~~~~~~~
        subject
~~~~~~~~~~~①~~~~~~~~~~~~
        aggregate_failures do
~~~~~~~~~~~②~~~~~~~~~~~~
          expect(response.body).to include 'PostRequestTest :Takashiの投稿'
          expect(response.body).to include '<span>投稿者</span>Takashi'
~~~~~~~~~~~②~~~~~~~~~~~~
        end
      end
    end


  ~~~~省略~~~~~
```

①subject(テスト対象のオブジェクト)は先ほどと同じ、投稿一覧のリクエスト処理  
②返却されたレスポンスのボディに投稿の内容とhtmlを含んでいるか確認します。  
response.bodyの内容を検証しています。






