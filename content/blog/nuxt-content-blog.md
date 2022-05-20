---
title: nuxt/contentでMarkdownのブログを作る
description: Generate blog from Markdown with nuxt/content
createdAt: 2021-09-05
tags:
  - Nuxt.js
---

[nuxt/content](https://content.nuxtjs.org/ja)という[NuxtJS](https://ja.nuxtjs.org/)のモジュールでMarkdownからブログが作れるらしいということで、試しに作ってみました。


## nuxt/contentとは
公式サイトには
> `nuxt/content` モジュールを使ってNuxtJSアプリケーションを強化します。`content/` ディレクトリに書き込むことで、MongoDBのようなAPIを使ってMarkdown、JSON、YAML、XML、CSVファイルを取得します。これは**GitベースのヘッドレスCMS**として動作します。

とあります。

今回はざっくりですが　Markdownで記事を作成 → 記事一覧・個別記事ページの生成 → それらを静的ファイルとして出力　までを行います。

記事を書くのはMarkdownですがタグやカテゴリー分け、ソート、ページネーションなどブログに必要な機能はひととおり実装することができ、使用感としてはとにかくホットリロードによる高速な動作でMarkdownを編集すると変更が即座にブラウザへ反映されるので執筆作業が鬼のように楽です。

また、恐ろしいことにdevelopmentモードでアプリケーションを実行している間ブラウザで記事をダブルクリックするとブラウザ上で直接記事の編集ができ、フォーカスを外すとその変更がMarkdownにも反映されます。


## 開発環境
* Node.js: v14.16.0
* npm: v6.14.11
* Nuxt: ^2.15.7
* @nuxt/content: ^1.14.0


## インストール
あらかじめ作成したnuxtのプロジェクトに追加することもできますが、今回は初めから使うことが決まっているので `create nuxt-app` でまとめてインストールします。nuxtのプロジェクトを始める前提条件は[公式ドキュメント](https://ja.nuxtjs.org/docs/2.x/get-started/installation)をご確認ください。
  
プロジェクトを作成したいディレクトリに移動し、`npx create nuxt-app <project-name>` を実行します。
いくつか質問があるので、作成したい仕様に合わせて回答します。今回はこの設定で作成しました。  
```
? Project name: blog
? Programming language: JavaScript
? Package manager: Npm
? UI framework: Vuetify.js
? Nuxt.js modules: Axios - Promise based HTTP client, Content - Git-based headless CMS
? Linting tools: ESLint, Prettier
? Testing framework: None
? Rendering mode: Universal (SSR / SSG)
? Deployment target: Static (Static/Jamstack hosting)
? Development tools: jsconfig.json
? Continuous integration: None
? Version control system: Git
```

インストールが完了したら生成されたプロジェクトのディレクトリに移動しアプリケーションを立ち上げます。
```
cd <project-name>
npm run dev
```


## サイトの構成

```
├── content
│   └── blog
│       ├── blog-1.md
│       ├── blog-2.md
│       └── blog-3.md
~
├── pages
│   ├── blog
│   │   └── _slug.vue
│   └── index.vue
```

TOPページに記事一覧を表示し、クリックすると各記事（`/blog/記事スラッグ`）へ遷移するようにします。Markdownの記事は`content/blog/` ディレクトリに格納し、これを各ページで取得します。  
  
nuxt/contentではデフォルトでプロジェクト直下の `content/` ディレクトリ配下に記事のファイルを作成するルールになっています。ディレクトリ名は何でも構いません。


## 記事を作成する
Markdownで記事を作成します。マークダウンファイルにはフロントマターというYAML形式のブロックを挿入することができ、ここで定義した変数を取得できます。フロントマターは必ずファイルの最初に記述します。
```markdown[blog-1.md]
---
title: nuxt/contentでMarkdownのブログを作る
description: Generate blog from Markdown with nuxt/content
---

[nuxt/content](https://content.nuxtjs.org/ja)という[NuxtJS](https://ja.nuxtjs.org/)のモジュールでMarkdownからブログが作れるらしいということで、試しに作ってみました。
```


## 記事一覧の表示
```vue[pages/index.vue]
<template>
  <ul>
    <li v-for="article in blog" :key="article.slug">
      <nuxt-link :to="'/blog/' + article.slug">
        <time :datetime="article.createdAt">
          {{ $dateFns.format(new Date(article.createdAt), 'yyyy/MM/dd') }}
        </time>
        <p>{{ article.title }}</p>
      </nuxt-link>
    </li>
  </ul>
</template>

<script>
export default {
  async asyncData ({ $content } {
    // 記事を全て取得（作成日で降順にソート）
    const blog = await $content('blog').sortBy('createdAt', 'desc').fetch()
    return {
      blog
    }
  }
}
</script>
```

`asyncData` で記事データを取得し `v-for` でループを回します。
日付の出力に用いている `item.createdAt` はファイルが作成された日付を返します。


## 記事詳細の表示
```vue[pages/blog/_slug.vue]
<template>
  <article>
    <h1>{{ article.title }}</h1>
    <NuxtContent :document="article" />
  </article>
</template>

<script>
export default {
  async asyncData ({ $content, params }) {
    const article = await $content('blog', params.slug).fetch()
    return {
      article
    }
  }
}
</script>
```

`$content` は引数を複数渡すことで以下のように変換されます。  
`$content('blog', 'blog-1')` → `/blog/blog-1`  
ファイル名を `_slug.vue` としているため `params.slug` には `$content` のスラッグ（ファイル名）が入ります。

## DEMO
以上の構成でブログを生成し静的出力したサンプルがこちらになります。（※UIフレームワークに[ Vuetify](https://vuetifyjs.com/ja/)を利用しています。）


## おわりに
今回は導入だけでしたが、公式サイトの実装例にも紹介されている通り記事の検索や目次の追加などといった機能も実装できるようですのでこれから記事も増やしつついろいろ試してみたいと思います。


## 参考
* [nuxt/content](https://content.nuxtjs.org/ja)
* [nuxt/contentでブログを作る](https://www.suzu6.net/posts/264-nuxt-content/)
* [Nuxt ContentをNuxt.jsのプロジェクトに導入する(TypeScriptにも対応)](https://tech-broccoli.life/articles/engineer/add-nuxt-content-for-nuxt/)
