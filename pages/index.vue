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