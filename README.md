![nextjs-microcms-book](https://github.com/nextjs-microcms-book/nextjs-website-sample/assets/4659294/eea23868-1c43-4833-9cd8-97e4298ff3e4)

「[Next.js+ヘッドレスCMSではじめる！ かんたんモダンWebサイト制作入門](https://www.amazon.co.jp/dp/4798183660/)」のサンプルリポジトリです。

## 本書の動作環境
OSはWindows、Macともに対応していますが、それぞれ本書において動作確認を行なったバージョンは次の通りです。
- Windows：Windows 11 Pro
- Mac：Ventura 13.2

また、本書でコーポレートサイト制作を行うための動作環境は以下の通りです。
- Node.js v20.13.1
- Next.js v14.1.4

## ディレクトリ構成
### appディレクトリ
このディレクトリは、アプリケーションの主要な機能やロジックを含む部分をまとめたものです。
- _actions：Next.jsのServerActions処理を格納します。本書においては、お問い合わせの送信処理が記述されています。
- _components：再利用可能なReactコンポーネントを格納します。UIの構築に使用される小さな部品がここに含まれます。
- _constants：定数を格納します。本書では各ページのデータ取得件数が定義されています。
- _libs：共通のユーティリティ関数やライブラリを格納します。これにはデータフォーマット関数やAPI呼び出しのヘルパー関数などが含まれます。
- contact：お問い合わせページに関するビューやロジックが格納されています。
- members：メンバーページに関するビューやロジックが格納されています。
- news：ニュースページに関するビューやロジックが格納されています。
### contents ディレクトリ
このディレクトリは、ニュースとしてCMSにインポートするためのコンテンツを格納しています。
### public ディレクトリ
このディレクトリは、静的ファイル（画像、フォント、その他のアセット）を格納します。
Next.jsでは、このディレクトリ内のファイルはURLパスで直接アクセス可能です。

## 開発環境の立ち上げ方法
```bash
npm run dev
```
コマンドラインにて上記コマンドを入力後、[http://localhost:3000](http://localhost:3000)にアクセスします。

## メモ
global.cssの役割
サイト全体で適用するもの。CSSカスタムプロパティの設定など書く。

Imageタグを使用する理由
大きい画像だと画面に表示する際に縮小して表示するから。
事前に縮小しておくことでパフォーマンスを維持する

ButtonLinkをdivタグで囲う理由
ボタンの位置や余白はコンポーネントを置く場所によって異なるから

コンポーネントに切り出すタイミング
入れ子になってきた。処理が複雑になってきた。

tsの型が複数ファイルに重複してきたら、libs/microcms.tsのようにまとめるようにする

@/appの@は絶対パスを示す、つまりルート

ページごとに細かな差異があるレイアウトを作る場合、Nesting Layoutを使う

use clientでクライアントコンポーネントになる
ユーザーの操作によってUIを更新したい場合にはクライアントコンポーネント
例：Menuコンポーネントはハンバーガーメニューのクリックで変わるから

Next.jsは外部画像の読み込みを制限している。
そのため、next.config.mjsでホワイトリストの作成が必要。

microCMSのlistのgetAPIはデフォルトで10件までしか返却されない。
そのため、limitパラメータで上限値を増やすか、ページネーションを使う必要がある。
limitを使っても上限は100まで。

microCMSのAPIには、１秒間あたりの呼び出し回数上限やレスポンスサイズの上限が決まってる。
https://document.microcms.io/manual/limitations#h1eb9467502

limit 100を使ってるとレスポンスサイズの上限に到達してしまうため、ページネーションが推奨

Newsの型定義でthumbnail?になっているのは必須属性じゃないから。

microCMSのAPIでページが見つからない場合、次のようにcatch文を使う。
await getCategoryDetail(params.id).catch(notFound)

下書きページの表示
npm run dev
microCMSの画面で下書きページを作成してプレビューボタン

useRouter
Linkコンポーネントを使用せずプログラムの中でNext.jsのルートを変更できる

microCMSはqというクエリパラメータで全文検索ができる

useSearchParamsを使うとクライアントコンポーネントになってしまう。
これの対処としては、useSearchParamsを使うところを関数化して、Suspenseタグで囲ってやる。

VercelはCDNとWebサーバーが組み合わせったサービス

ビルド時のマークの意味
◯：静的レンダリング（SSG)
λ：動的レンダリング（SSR)

メンバーページなど静的コンテンツを編集してもCDNで止まって、microCMSまで見に行かないので反映されない。
反映するにはWebhookの設定が必要。

1. Vercel画面から「Settings」->「Git」->「Deploy Hooks」にてHookの作成。
Hook名はmicroCMSで任意、ブランチはmain
2. 1でコピーしたURLをmicroCMSのメンバーページから「API設定」->「Webhook」で設定

revalidate = 0
キャッシュの保持期間を0にするとキャッシュを使わず、毎回オリジンサーバーにデータを取得する。

Next.jsではページ単位だけでなくデータ単位でもキャッシュの設定ができる。
microcms.tsの次の箇所
revalidate: queries?.draftKey === undefined ? 60 : 0,

一覧ページと詳細ページにおけるキャッシュの保持期間(revalidate)は同じにするか、詳細ページのほうが短くなるようにする。
でないと、一覧ページから詳細ページにうつったときに、NotFoundになる場合がある。

revalidateが親階層と子階層にある場合、値が小さいほう（つまり更新頻度が高いほう）が適用される。

Middleware
リクエストがNext.jsのサーバーサイドに到達する前に何らかの処理を動かせる機構
・リクエストの検証や変更
・認証や認可のチェック
・リダイレクト
・カスタムロギングや監視

Next.js13からはMiddlewareの処理がVercelのエッジサーバー上で行われるようになっているため、レスポンスも高速。
Basic認証もMiddlewareの機能で使える
nextjs-basic-auth-middleware@3.1.0
このコード上では省略。

Server Actionsとは？
主にサーバーサイドで実行される機能（データの取得や処理）を、フロントエンドのコンポーネントから直接呼び出せる

useFormStateは次期バージョンでuseActionStateという名前に変わる。

HubSpotはフォームの内容を記録するDB代わり。

Server Actionsは_actionsディレクトリ配下に書く。
"use server"の記述が必要。

動画
https://youtu.be/wf6rW1-1ZQQ

Hubspotにフォームが届かない原因
Vercelの環境変数を追加した後に、リデプロイしてなかったので反映されてなかった

generateMetadata
メタデータ設定用の関数

パフォーマンス改善
next/imageコンポーネントのpriorityをセットする。
fetchpriority
loading属性をeagerに
srcsetは複数の画像をわたして最適なものを選ばせる


