---
title: ViteとEleventyで静的サイト構築
description: 
createdAt: 2022-03-04
tags:
  - Vite
  - Eleventy
---

先日発表された[The State of JavaScript 2021](https://2021.stateofjs.com/en-US/)では98%の満足度を獲得しエンジニアからの高い注目度がうかがえる[Vite](https://ja.vitejs.dev/)ですが、ビルドツールのため静的なページの生成には別途仕組みが必要になるかと思います。

公式のテンプレートにはVueやReactなどがサポートされていますが、これらは学習コストやプロジェクトの要件にもより気軽に導入できるものではないかもしれません。

そこで今回はページの生成に静的サイトジェネレーターの[Eleventy](https://www.11ty.dev/)（以下、11ty）を利用し、Viteの高速な開発体験の恩恵を受けつつ11tyでページを生成するシンプルな静的サイトの開発環境を作ってみたいと思います。

※11tyについての記事は[こちら](https://www.evoworx.co.jp/blog/11ty-static-site-generator/) （バージョンが記事公開時より更新されていますので利用の際は[公式ドキュメント](https://www.11ty.dev/docs/getting-started/)をご確認ください）

## 環境
- Node.js: v16.12.0 ※Viteの利用には12.2.0以上のバージョンが必要です。
- @11ty/eleventy: ^1.0.0
- vite: ^2.8.6
- ディレクトリ構成

```shell
├── .eleventy.js
├── _site: 出力
│   ├── index.html
│   └── main.js
├── package-lock.json
├── package.json
├── src: 入力
│   ├── _data
│   │   └── build.js
│   ├── client
│   │   └── main.js
│   └── index.njk
└── vite.config.js
```

## インストール~開発コマンド作成
### 11tyのインストール
プロジェクトディレクトリに移動したら ```package.json``` を作成し11tyをインストールします。
```shell
npm init -y
npm install --save-dev @11ty/eleventy
```
テンプレートには弊社でもよく利用している [Nunjucks](https://mozilla.github.io/nunjucks/) を利用します。 ```/src/index.njk``` を作成します。
```html[/src/index.njk]
<!DOCTYPE html>
<html lang="ja">
<head>
  ...
</head>
<body>
  <p>hello from 11ty</p>
</body>
</html>
```
次に11tyの設定ファイルを作成します。プロジェクトルートに ```.eleventy.js``` を作成し下記のように設定します。
```js[/.eleventy.js]
module.exports = function (eleventyConfig) {
  return {
    templateFormats: ['njk', 'html'],
    htmlTemplateEngine: 'njk',
    dir: {
      input: 'src',
      output: '_site'
    }
  }
}
```
ここまで作成したら11tyを実行します。
```shell
npx @11ty/eleventy
```
これで ```src``` ディレクトリのテンプレートが ```_site``` ディレクトリにコンパイルされます。

ローカルWebサーバーを起動し、生成されたファイルを確認してみます。
```shell
npx @11ty/eleventy --serve
```
11tyのホットリロードローカルWebサーバーが起動しソースファイルに変更を加えるとブラウザが自動でリロードされるのが確認できます。（portは ```:8080```）

### Viteのインストール
次にViteをインストールしていきます。
```shell
npm install --save-dev vite@latest
```
```/src/client/main.js``` ←このようにディレクトリとjsファイルを作成し、テスト用のコードを書いておきます。
```js[/src/client/main.js]
function init() {
  const successNode = document.createElement('p')
  document.body.appendChild(successNode)
  successNode.innerText = 'hello from vite'
}
init()
```
```index.njk``` に作成したjsを読み込む記述を追加します。
```html[/src/index.njk]
...
  <p>hello from 11ty</p>
  <script type="module" src="http://localhost:3000/src/client/main.js"></script>
</body>
```

### 11tyとViteを並列に実行
開発の際は11tyとViteを並列に起動し**11tyのローカルサーバー**でファイルを確認します。コマンドは ```npx @11ty/eleventy --serve & npx vite``` のようにつなげても動作しますが、記述が煩雑になりやすいのとOS環境への依存を避けるためツールを利用します。[concurrently](https://www.npmjs.com/package/concurrently)という複数のコマンドをまとめて実行できるパッケージがありますのでこちらをプロジェクトに追加します。
```shell
npm install --save-dev concurrently
```
```package.json``` の ```scripts``` に開発コマンドをまとめます。
```json[/package.json]
...
"scripts": {
    "dev": "concurrently npm:dev:*",
    "dev:eleventy": "eleventy --serve",
    "dev:vite": "vite"
  },
```
これで下記コマンドを実行し ```localhost:8080```（11tyの方。Viteはデフォルトで ```localhost:3000``` ）で開発を行います。
```shell
npm run dev
```

## プロダクションビルド
### ビルドタスクの作成
開発時の設定はできたので、最後にビルドタスクを作成します。プロジェクトルートに ```vite.config.js``` を作成しオプションを下記のように設定します。
```js[/vite.config.js]
import { defineConfig } from 'vite'
export default defineConfig({
  build: {
    outDir: '_site',
	rollupOptions: {
      input: '/src/client/main.js',
      output: {
        entryFileNames: 'main.js',
      }
    }
  }
})
```
Viteはビルド時のバンドルに[rollup.js](https://rollupjs.org/guide/en/)を採用しており ```rollupOptions``` を変更することでファイル名など出力時の設定がカスタムできます。詳しくは[こちら](https://rollupjs.org/guide/en/#big-list-of-options)。

また、 ```package.json``` にビルド用の ```scripts``` を追記し
```json[/package.json]
...
"scripts": {
    "dev": "concurrently npm:dev:*",
    "dev:eleventy": "eleventy --serve",
    "dev:vite": "vite",
	"build": "npm run build:vite && npm run build:eleventy",
    "build:eleventy": " eleventy",
    "build:vite": "vite build"
  },
```
下記コマンドでビルドします。
```shell
npm run build
```
これで ```_site``` ディレクトリにビルドされた ```main.js``` が出力されます。

### 開発時とビルド時でコードを分岐
現状のコードではjsをソースファイルから直接読み込んでいますが（開発時にViteが高速なのはこのためです）、サイトのデプロイ時にはバンドルされたアセットを読み込む必要があります。そのため開発時とビルド時で環境変数を利用しコードを分岐させます。

```/src/_data/build.js``` ←このようにディレクトリとjsファイルを作成し、環境変数を格納しておきます。
```js[/src/_data/build.js]
module.exports = {
  env: process.env.NODE_ENV
}
```
次に ```index.njk``` のjs読み込み箇所をif文（Nunjucksテンプレートタグ）で分岐します。
```html[/src/index.njk]
...
  <p>hello from 11ty</p>
  {% if build.env === "production" %}
  <script type="module" src="./main.js"></script>
  {% else %}
  <script type="module" src="http://localhost:3000/src/client/main.js"></script>
  {% endif %}
</body>
```
```package.json``` のビルドコマンドに環境変数を追記します。
```json[/package.json]
...
"scripts": {
    "dev": "concurrently npm:dev:*",
    "dev:eleventy": "eleventy --serve",
    "dev:vite": "vite",
	"build": "npm run build:vite && npm run build:eleventy",
    "build:eleventy": "NODE_ENV=production eleventy",
    "build:vite": "NODE_ENV=production vite build"
  },
```
これで再度 ```npm run build``` を行うと、jsの読み込み箇所がプロダクション用に書き換わっているのが確認できるかと思います。

<!-- ### ブラウザ確認用のプロダクションビルドコマンドの作成
最後に、ビルドしたサイトをブラウザで確認するプロダクションビルド用のコマンドを作成します。
開発時と同様に11tyのローカルサーバーを利用してもいいのですが、ローカルサーバーを立てつつportを毎回ランダムに変更できる ```serve``` というパッケージがあるのでこちらを利用します。
```shell
npm install --save-dev serve
```
```package.json``` にプロダクションビルドのコマンドを追記します。
```json[/package.json]
...
"scripts": {
    "dev": "concurrently npm:dev:*",
    "dev:eleventy": "eleventy --serve",
    "dev:vite": "vite",
	"build": "npm run build:vite && npm run build:eleventy",
    "build:eleventy": "NODE_ENV=production eleventy",
    "build:vite": "NODE_ENV=production vite build",
	"prod": "NODE_ENV=production npm run build && serve -l 0 _site"
  },
```
※ ```serve``` は引数でportが指定でき、番号を0にするとOSで使用していないportがランダムに割り当てられます。 -->

## 終わりに
この記事では取り扱っていませんが、ViteはSassもパッケージを追加してscssをjsにインポートするだけで利用できます。公式にも記載がありますので[こちら](https://ja.vitejs.dev/guide/features.html#css-%E3%83%97%E3%83%AA%E3%83%97%E3%83%AD%E3%82%BB%E3%83%83%E3%82%B5)を参考にしてみてください。

## 参考
- [fpapado/eleventy-with-vite](https://github.com/fpapado/eleventy-with-vite)
- [Clean SASS and JS with Eleventy in 2021 (Using Vite)](https://medium.com/@SimonEast/clean-sass-and-js-with-eleventy-in-2021-using-vite-98747500d8f8)
