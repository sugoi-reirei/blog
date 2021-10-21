---
title: Nuxt.jsのcontentでブログ作成してみた。
description: '@nuxtjs/contentを使用したブログを作成してみました。'
---

## NuxtJSのプロジェクト作成
```npx
npx create-nuxt-app <project-name>
```
`<project-name>`には自分のプロジェクト名を入れます。

```npm
create-nuxt-app v3.7.1
✨  Generating Nuxt.js project in my-blog
? Project name: my-blog
? Programming language: JavaScript
? Package manager: Npm
? UI framework: Vuetify.js
? Nuxt.js modules: Content - Git-based headless CMS
? Linting tools: (Press <space> to select, <a> to toggle all, <i> to invert sele
ction)
? Testing framework: None
? Rendering mode: Universal (SSR / SSG)
? Deployment target: Server (Node.js hosting)
? Development tools: (Press <space> to select, <a> to toggle all, <i> to invert 
selection)
```
UIフレームワークはVuetifyを使用します。
Contentモジュールを選択します。
Gitを使用します。


### プロジェクトに移動してアプリケーションを起動
```npm
cd <project-name>
npm run dev
```

ブラウザで [http://localhost:3000](http://localhost:3000) を開いてみましょう。


## サイトの構成
[こちらの記事](https://www.evoworx.co.jp/blog/nuxt-content-blog/)を参考にします。
```language
├── content
│   └── blog
│       ├── blog-1.md
│       ├── blog-2.md
│       └── blog-3.md
~
├── pages
│   ├── blog
│   │   └── _slug.vue
│   └── index.vue
```

## 見た目の調整
`layouts/default.vue`
```vue
<template>
  <v-app dark>
    <v-navigation-drawer
      v-model="drawer"
      
      :clipped="clipped"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar
      :clipped-left="clipped"
      fixed
      app
      dense
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-footer
      :absolute="!fixed"
      app
    >
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Home',
          to: '/'
        }
      ],
      title: 'My Blog'
    }
  }
}
</script>
```
vuetyfyのテーマも`light`に変更。

`nuxt.config.js`
```js
{
  vuetify: {
    customVariables: ['~/assets/variables.scss'],
    theme: {
      light: true, //ここを変更
      themes: {
        light: { //ここを変更
          primary: colors.blue.darken2,
          accent: colors.grey.darken3,
          secondary: colors.amber.darken3,
          info: colors.teal.lighten1,
          warning: colors.amber.base,
          error: colors.deepOrange.accent4,
          success: colors.green.accent3
        }
      }
    }
  }
}
```



## 記事の作成
公式サイトの[nuxt/content](https://content.nuxtjs.org/ja/writing)を参考に記事を作成します。
```md
---
title: Home
---

## Links

<nuxt-link to="/articles">Nuxt Link to Blog</nuxt-link>

<a href="/articles">Html Link to Blog</a>

[Markdown Link to Blog](/articles)

<a href="https://nuxtjs.org">External link html</a>

[External Link markdown](https://nuxtjs.org)
```


## 記事一覧の表示

日付をフォマッとするための`@nuxtjs/date-fns`をインストールしておきます。
```npm
npm install @nuxtjs/date-fns
```

`nuxt.config.js`にモジュールを追加します。
```js
{
  modules: [
    '@nuxt/content',
    '@nuxtjs/date-fns'
  ],
}
```


`pages/index.vue`
```vue
<template>
  <v-container>
  <div class="mb-6">
    <h1>Welcome My Blog</h1>
  </div>
  <v-row>
    <v-col  v-for="article in blog" :key="article.slug"
            class="mx-auto"
    >
      <v-card outlined>
        <v-list-item>
          <v-list-item-content>
            <time class="text-overline mb-4" :datetime="article.createdAt">
              {{ $dateFns.format(new Date(article.createdAt), 'yyyy/MM/dd') }}
            </time>
            <nuxt-link :to="'/blog/' + article.slug">
              <v-list-item-title class="text-h5 mb-1">
                {{ article.title }}
              </v-list-item-title>
            </nuxt-link>
          </v-list-item-content>
        </v-list-item>
      </v-card>
    </v-col>
  </v-row>
  </v-container>
</template>

<script>
export default {
  async asyncData ({ $content }) {
    // 記事を全て取得（作成日で降順にソート）
    const blog = await $content('blog').sortBy('createdAt', 'desc').fetch()

    return {
      blog
    }

  }
}
</script>
```
### おまけ
コードをシンタックスハイライトさせるためのモジュールも追加します。
```npm
npm install prism-themes
```
`nuxt.config.js`のcontentに追加します。
```js
{
  content: {
    markdown: {
      prism: {
        theme: 'prism-themes/themes/prism-material-oceanic.css'
      }
    }
  },
}
```


## 記事詳細の表示
`pages/blog/_slug.vue`
```vue
<template>
  <v-container>
  <article>
    <h1 class="text-h3 font-weight-bold my-6">{{ article.title }}</h1>
    <NuxtContent :document="article" />
  </article>
  <v-row>
    <v-col class="d-flex justify-space-between mb-6">
      <v-list-item >
        <v-list-item-content>
          <div  v-if="prev" class="caption mb-1">
            Prev
          </div>
          <NuxtLink v-if="prev" :to="{ name: 'blog-slug', params: { slug: prev.slug } }">
            
            <v-list-item-title class="subtitle-2 mb-1">{{ prev.title }}</v-list-item-title>
          </NuxtLink>
        </v-list-item-content>
      </v-list-item>
      
      <v-spacer />

      <v-list-item>
        <v-list-item-content>
          <div v-if="next" class="caption mb-1" tag="div">
            Next
          </div>
          <NuxtLink v-if="next" :to="{ name: 'blog-slug', params: { slug: next.slug } }">
            
            <v-list-item-title class="subtitle-2 mb-1">{{ next.title }}</v-list-item-title>
          </NuxtLink>
        </v-list-item-content>
      </v-list-item>
    </v-col>
  </v-row>
  <v-btn link nuxt to="/">
    Back to home
  </v-btn>
  </v-container>
</template>
 
<script>
export default {
  async asyncData ({ $content, params }) {
    const article = await $content('blog', params.slug).fetch()

    //前後の記事を表示
    const [prev, next] = await $content('blog')
      .only(['title', 'slug'])
      .sortBy('createdAt', 'asc')
      .surround(params.slug)
      .fetch()

    return {
      article,
      prev,
      next
    }

  }
}
</script>
<style>
  /* vuetyfy側で背景色が設定されているのでリセット */
  .nuxt-content-highlight code {
    background-color: rgba(255, 255, 255, 0)!important;
  }
</style>
```

出来上がったらこのサイトと同じになるはず

vercelにデプロイしようとしたらクラッシュして500エラー。

ルートディレクトリに`vercel.json`を作成。
```json
{
  "builds": [
    {
      "src": "nuxt.config.js",
      "use": "@nuxtjs/vercel-builder",
      "config": {}
    }
  ]
}
```
あと`package.json`の変更。
```json
{
  "scripts": {
    ...
    "build": "nuxt generate"
  }
}
```

あと`git push`しても毎回`everything up-to-date`になってしまうので以下のコードで対処。
```zsh
% git add -A
% git commit -m "recommit" -v
% git push
```


## ダメだから新たにプロジェクト作成...
今回は以下の設定。

```npm
create-nuxt-app v3.7.1
✨  Generating Nuxt.js project in blog
? Project name: blog
? Programming language: JavaScript
? Package manager: Npm
? UI framework: Vuetify.js
? Nuxt.js modules: Axios - Promise based HTTP client, Content - Git-based headle
ss CMS
? Linting tools: ESLint, Prettier
? Testing framework: None
? Rendering mode: Universal (SSR / SSG)
? Deployment target: Static (Static/Jamstack hosting)
? Development tools: (Press <space> to select, <a> to toggle all, <i> to invert 
selection)
? Continuous integration: None
? Version control system: Git
```

中身は同じでVercelにデプロイ！
