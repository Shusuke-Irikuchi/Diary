# BFFについて

## 調べる前
- GraphQLが登場するイメージ
- Backend For Frontendの頭文字でBFFらしい

## オレオレ調査開始🕵️‍♀️

### 背景
2000年代になってAjax通信が浸透しバックエンド(サーバ）はよりAPIに特化し, フロントエンドはよりUI/UXに特化した構築がなされるようになった。**マイクロサービスアーキテクチャー**
そして,あらゆるクライアントデバイスへの対応やUXの向上, リクエストの制限の対策等としてフロントとバックエンドとの間にサーバを用意してあげる.(こいつが**BFF**)

### 仕様
BFFは主なユースケースを代表的なユースケースをgosyoukai
- APIGateWay
- SSR
- ファイルアップロード
- セッション管理
- WebSocket、Server Sent Events、Long Polling(こいつについては今回は言及しない)

#### APIgateWay
例えば、様々なプラットフォームに対応をしたく,それぞれで出力するデータを制限したくなったときのことを考えます.
```
/api/news?platform=web  //web
/api/news?platform=phone //スマホ
/api/news?platform=game  //ゲーム機器
/api/news?platform=tv //テレビ 
```
![IMG_229E0A50B7AB-1](https://user-images.githubusercontent.com/56505469/85938219-366a3200-b946-11ea-8fc5-bb0f66779fc8.jpg)  
これだとAPI側で各プラットフォームを考慮しきれなくなる場合もあり,コードが重複する可能性もあります.それなら,プラットフォームと対になるようにAPIを用意すればいいじゃないかと思うかもしれませんが,
これだと後から機能追加だったりや同様にプラットフォーム間でコード重複する可能性があります.そこでどのようにするかというと,下記のようにします.  
![1](https://user-images.githubusercontent.com/56505469/85938230-53066a00-b946-11ea-9c25-4108a03fa66e.jpg)  
BFFが各プラットフォームにあったレスポンスを返すようにすることでAPI側の開発効率をあげるだけでなく,プラットホームを増やせるようになりました.
しかしこれだとBFFへの負荷が大きくなってしまうので,各プラットフォームにBFFを用意することでBFFへの負荷を分散させます.それにBFFを設けることで単一ページで複数のAPIが必要となったときにフロントエンドからのリクエストを1回で済ませることができます.
またhttp/1.1の場合でもBFFを置くことでリクエスト数を減らすことができ,パフォーマンスをあげることが可能です.

(少しだけhttp/1.1の話)http/1.1の場合,サーバーのTCPコネクションは最大6つまで可能(6つあるけど,規約に同時接続は2つまでにすべきとある)であるがその分サーバーの負荷が増加してします。
そのため、httpパイプラインという物があるがこれはある特定の通信が遅い場合に後ろの通信を待たせてしまう。(Head of Line Blocking)
そのため,BFFを用意することでリクエストを1つにまとめることが可能になりリソースを有効的に活用できる.

#### SSR
単純なHTML/CSS/JacaScriptのみで動くアプリケーションの場合,
- サーバへGET
- サーバはURLに対応したHTMLを返す
- ブラウザはHTMLを評価(Ajax通信等があったら,またサーバへ)

これのよくないところは毎回,HTMLをごそっと返すので共通部分とかがまた更新される(headerとかfooter)は変更する必要がない.
それなら,そのままで行こうや⛹🏻‍♀️ これが**SPA**
- 最初は上と一緒.
- 2回目以降はJsを使って差分のみをリクエストする.

無駄な部分を要求しなくなったので,スイスイ動く.でもSPAの問題点があり, 初回のJSの読み込みに時間がかかってしまい,真っ白ページがただ出るだけみたいなことがある.  
これだと,ユーザは**なんや、壊れとるのか**と思い離脱🏃.さらにSEO的にも問題がありSPAの場合サーバから渡されているHTMLは下記のようになっており.jsが実行されないとコンテンツがレンダリングされない.
これのなにがダメなのかというと動的に更新されたHTML部分をクローラー🏊‍♂️が正しく評価できているかわからない問題発生します.どんな素晴らしいサイトでも検索して出てこなかったら意味がありません.
```
<div id=root></div>
```
どうしようと.頭を抱え夜しか寝られない生活を送ってる皆さんに送りたい.それが,**SSR**  
この子はサーバ側でコンテンツを評価(BFF)した状態でブラウザに返却します。そのため,ブラウザはいちいちhtmlを評価する必要がなく,画面に描画するだけなのでUXの向上と共に
SSRの場合はクローラー🏊‍♂️が正しく評価してくれるのでSEO面も改善されます!!

#### ファイルアップロード
多分、よくある実装が
- ファイルをbase6４でエンコード.
- テキストでサーバへPOST

このときに,変換されたテキストは改行や空白等が含まれ約1.4倍に増加してしまいます.これだと,100MBを送るとなったとき140MBとなってしまいます.
別の方法として、バイナリデータをチャンクして少しづつ送るという方法やmulti-partにして送る方法があるのですが,こうするとJSONベースのAPIサーバでは実装が困難になるらしい(そうなのバックエンドの皆さん??)
そこで登場BFF.こいつがBFFで受けっとったチャンクをS3などのストレージ送って1つのファイルに戻してから、バックエンドのAPIをファイルパスをパラメタにしてリクエストする.

#### セッション管理
簡単に言うと複数のAPIを叩きたい場合に毎回,セッションをチェックしていたら面倒だし時間もったいないねと言うことでBFFのセッションをさせて、OKだったら一気にAPIを叩くと言うことです.
いい説明があったのでこちらをどうぞ[BFFセッション管理]（https://www.atmarkit.co.jp/ait/articles/1805/18/news022.html）

### 添書き
- GraphQLの登場を当人が記憶していたのは次の記事を読んだ覚えがあったらしい.....[フロントエンドに型の秩序を与えるGraphQLとTypeScript](https://www.wantedly.com/companies/wantedly/post_articles/183567)
- BFFという言葉ができる前からNetflixはこのアーキテクチャを導入していたらしい.すごい!
