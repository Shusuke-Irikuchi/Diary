# webにおいてコンテンツが画面がされるまでのお話

## レンダリングエンジンについて
まずレンダリングエンジンってなんぞや？と言うお話からしていきます.  
レンダリングエンジンは情報(HTMLとかXML)を解析して, 特定のルールにしたがってそれを適切な表現に変換します.
もっとわかりやすく言うとユーザがこれ頂戴っていったきたコンテンツがあり,サーバから送られてきたものを解析してブラウザに表示すると言うことです
ブラウザごとにレンダリングエンジンを持っており下記にそれらを示しておきます.
- google: Blink
- Sfari: Webkit
- Firefox: Gecko
- Microsoft Edge: EdgeHTML


### フローについて
簡単なフローについて素晴らしい物があったのでそちらを載せておきます。

<img width="629" alt="スクリーンショット 2020-06-28 18 17 48" src="https://user-images.githubusercontent.com/56505469/85943562-c66ea280-b96b-11ea-981d-7ca99962ae71.png">

## construct the DOM tree

ダウンロードしたリソースをパースします.ここでは主に(字句解析,構文解析が行われます)
- HTMLファイル
- CSSファイル
- JavaScriptファイル
- jpeg、png、gif、svg, など

HTMLファイル → DOMツリー  
CSSファイル → CSSOMツリー  
この時にDOMツリー内で宣言されているリソースをさらに取得し読み込み進めていきます.(2回目以降はキャッシュなど)

## Scripting
JsのコードをJavaScriptエンジンに引き渡して実行させます. 
### Javascriptエンジンについて
 名前のとおりこいつがいるおかげでブラウザでJSを実行することができます.これも下記ブラウザでいろいろあります。
- google: V8
- Sfari: Nitro
- Firefox: SpiderMonkey(かっこいい)
- Microsoft Chakra
次の工程のように実装を行っていきます
- 字句解析
- 構文分析
- 抽象構文木の作成
- コンパイル(実行可能な形式に変換)
- 実行

字句解析と構文解析の説明は先と同様ようです.例えば下記のようなJsコードがあった場合に字句解析と構文解析を無事終えた後の抽象構文木は下記の図のようになります.
```
if ( a > 0 ) {
 x = a;
}
else {
 x = -a;
}
```

<img width="670" alt="スクリーンショット 2020-07-04 23 20 10" src="https://user-images.githubusercontent.com/56505469/86514428-fb3b9780-be4c-11ea-85a1-5e7b84a55653.png">  
上記のような構文木ができた後に,JavaScriptエンジン内部のコンパイラによって実行可能な形式にコンパイルされます.
コンパイル終了後に初めて実行されてこのときDOMの変更等があった場合はDOMツリーにその変更が反映されます.

## layout of the render tree
Jsの実行が終了するとスタイルの計算を行い,レイアウトツリーの構築が行われます.
<img width="890" alt="スクリーンショット 2020-07-05 12 47 09" src="https://user-images.githubusercontent.com/56505469/86525104-bdc32280-bebd-11ea-865c-272e1ddf8cf7.png">

