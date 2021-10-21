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