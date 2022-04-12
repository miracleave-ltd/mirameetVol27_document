# システムテスト(ST)
## 本セクションで行うこと
- システムテストについて
- System spec解説


## システムテストについて

システムテスト(System Test)は、要件定義書に記載した要件をクリアしているか?  
つまりアプリ全体が一つのシステムとして期待される動きをするのか?を確認するテストになります。  
これまで説明した単体テスト、結合テストとは、詳細設計、基本設計で記載した要件をクリアしているかどうかを確認していました。  
このような形のテストモデルを「V字モデル」と呼びます。

<img width="648" alt="スクリーンショット 2022-03-21 15 34 10" src="https://user-images.githubusercontent.com/52161269/159214576-b1c97d34-531e-42b1-a467-2461a0dc8bfe.png">

では、実際にシステムテストを行う際はどのようにテストケースを実施すれば良いでしょうか?  
システムテストの手法としていくつかありますが、今回はシナリオテストを行います。

シナリオテストは、実際のユーザーの行動をシナリオ化してテストを行います。  
投稿機能を例に考えてみましょう。  

```
投稿機能におけるシステムテスト：
1. ユーザーは投稿する
2. ユーザーは投稿の編集をする
3. ユーザーは投稿の削除をする
4. ユーザーは自分の投稿にコメントをする
5. ユーザーは他人の投稿を閲覧し、コメントをする
6. ...以下省略...
```

## System spec解説

RSpecでは、システムテストとして行うテストをSystem specと呼んでいます。
System specでは、ユーザーの動きをコードに起こしてテストする為に、「Capybara」と言うgem(ライブラリ）を使います。  
Capybaraは、Rails5.1以降では標準インストールされています。  
Capybaraは、指定したページにアクセスしたり、フィールドに入力したり、ボタンをクリックしたり、ブラウザの操作を可能にします。


先ほどの投稿機能のシステムテストケースを使って、実際のコードを確認していきましょう！

```ruby
require 'rails_helper'

RSpec.feature 'Posts', type: :system do
  background do
    driven_by(:rack_test)
  end
~~~~~~~~~~~①~~~~~~~~~~~~
  let(:user_a) { create(:user, nickname: 'Takashi') }
~~~~~~~~~~~①~~~~~~~~~~~~
  let(:user_b) { create(:user, nickname: 'Satoshi') }
  let!(:post) { create(:post, text: 'hello world', user: user_a)}
~~~~~~~~~~~②~~~~~~~~~~~~
  before do
    sign_in_as user_a
  end
~~~~~~~~~~~②~~~~~~~~~~~~

  describe '投稿の作成' do
    scenario 'ユーザーは新しい投稿を作成する' do
      ActiveJob::Base.queue_adapter = :test

      expect {
~~~~~~~~~~~③~~~~~~~~~~~~
        click_link_or_button '投稿する'
        fill_in 'post_text', with: 'sample_text'
        click_button 'commit'

        post = Post.find_by_text_and_user_id('sample_text :Takashiの投稿', user_a.id)
~~~~~~~~~~~③~~~~~~~~~~~~

~~~~~~~~~~~④~~~~~~~~~~~~
        aggregate_failures do
          expect(current_path).to eq posts_path
          expect(page).to have_content 'sample_text'
          expect {
            SlackSyncJob.perform_later(post)
          }.to have_enqueued_job.with(post)
        end
~~~~~~~~~~~④~~~~~~~~~~~~
      }.to change(user_a.posts, :count).by(1)
    end
    
~~以下省略~~
```

##### 3.1. ユーザーは投稿する

①投稿する為にユーザーを定義

②テスト実施前に、ユーザーのログイン処理を行う

③テスト対象は`ユーザーは投稿する`なので、Capybaraのメソッドでブラウザ操作して投稿処理をする  

上記では、「投稿する」ボタンを押下して遷移先で、"sanple_text"と投稿欄に入力して、投稿ボタンを押下しています。

**③で使用しているCapybaraメソッド**  
`click_link_or_button　'対象要素のIDやテキスト'`  
　・・・指定したボタンもしくはリンクを探してクリックする。  
 
`fill_in '対象要素のnameやID', with: 'フィールドに入力する文字'`  
　・・・指定した要素を探して、with以降で指定した文字を入力する。  

`click_button　'対象要素のIDやname'`  
　・・・指定したボタンを探してクリックする。  

④テスト対象（③）に対して、遷移先のパスが合っているか、投稿した内容を表示しているか、スラック通知処理はできているかを確認


**④で使用しているCapybaraメソッド**  
`current_path`・・・現在表示しているページのパスを返す。  
`page`・・・現在表示しているページのオブジェクト。


補足：`aggregate_failures do...end`は、これで囲まれたexpectは、expectの評価で失敗しても次のexpectを実行することができるようになります。これまでのRSpecだと、一つのexampleにつき複数のexpectで書くと失敗した時にそれ以降のexpectが実行されませんでしたが、これにより一つのexpamleに複数のexpectを書くことが出来ます。













