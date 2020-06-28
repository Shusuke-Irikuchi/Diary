# GraphQL

## 調査前😑
- スキーマ言語?ってやつ
- BFFに登場するイメージ

## オレオレ系調査開始🕵️‍♀️

### 背景
[GraphQLパターン](https://speakerdeck.com/twkiiim/huo-yong-patandexue-bugraphql?slide=26)を参考にさせていただきました.
以下のようなAPIを考える
```
GET /api/users
GET /api/users?page_num3 //ページネーション
```
要件が増加して, モバイルとウェブとタブレットを考慮することとなると
```
GET /api/users?page_num3&devise=mobile 
GET /api/users?page_num3&devise=web 
GET /api/users?page_num3&devise=tablet
```
これならいいが、画面構成によってレスポンスデータの種類を変えるとかさらに他のクライアントデバイスを考慮しようとするとAPI側の負荷がかかり, 重複したコードが多くなり効率が悪くなる.
ちゃんと設計してなからじゃないのかとツッコミを入れたくなったが**OSFA問題**という物がすでにあったらしい.これはユーズケースに対して最適化されるように設計されたが、APIが次第に大きくなり
多くのユーズケースに答えるようになってメンテナンスや拡張が難しくなってくること...... **ここでGraphQL登場!!!!!**    
GraphQLはスキーマで定義することができて, query(取得), mutation(更新), subscription(サーバからのイベント通知)3つのクエリから構成されてる（素直にシンプルでTSと相性がいいと思った)

## 仕様・実装🏋️‍♀️

<img width="434" alt="スクリーンショット 2020-06-27 23 34 58" src="https://user-images.githubusercontent.com/56505469/85924726-0cc1f400-b8cf-11ea-8535-0a1d9697066e.png">
背景でも述べたようにGraphQlの登場前はAPI側が全てのユースケースを考慮していた.それならユースケースごとのAPIを分ければいいじゃないかと思うかもしれないが,NetFlixを考えて欲しい.
パソコン, スマートフォン, タブレット, カーナビ, ゲーム機,などこれ以外にもさまざなデバイスに対応している.この様子だと将来,デバイスが増えること考慮するとあんまりよろしくない.
<img width="509" alt="スクリーンショット 2020-06-27 23 41 52" src="https://user-images.githubusercontent.com/56505469/85924848-cfaa3180-b8cf-11ea-986f-1dd6f907f0bc.png">
そこでGraphQLのようなBFFサーバを置いてあげることでAPI側の無駄な工数を減らせるだけでなく,クライアント側でフィールドを選んでリクエストできるのでデバイスにあったリクエストが可能となる.
