<template>
  <v-container>
    <v-row justify="center" align="center">
      <v-col cols="12" md="8">
        <h1 class="text-h4 font-weight-bold">
          {{ article.title }}
        </h1>
        <!-- <v-chip-group column>
          <v-chip
            v-for="tag in article.tags"
            :key="tag"
            small
            color="indigo darken-1"
            class="white--text"
            :to="'/blog/' + tag + '/'"
          >
            # {{ tag }}
          </v-chip>
        </v-chip-group> -->
      </v-col>
    </v-row>
    <v-row justify="center" align="center">
      <v-col cols="12" md="8">
        <NuxtContent :document="article" />
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default{
  async asyncData ({ $content, params }) {
    const article = await $content('blog', params.slug).fetch()
    return {
      article
    }
  },
  head () {
    return {
      title: this.article.title
    }
  }
}

// import { Context } from '@nuxt/types'
// import Vue from 'vue'

// export default  Vue.extend({
//   async asyncData ({ $content, params }: Context) {
//     const article = await $content('article', params.slug).fetch()
//     return {
//       article
//     }
//   },
//   head () {
//     return {
//       title: this.article.title
//     }
//   }
// })
</script>

<style lang="scss" scoped>
  .container,
  .container .text-h4 {
    font-family: YuGothic, Roboto, sans-serif !important;
  }
  .container .text-h4 {
    line-height: 1.5;
  }
</style>

<style lang="scss">
  .nuxt-content {
    padding-top: 2.5rem;
    padding-bottom: 5rem;

    h2 {
      position: relative;
      font-size: 1.75rem;
      font-weight: 600;
      display: table;
      margin-top: 2rem;
      padding-top: 2rem;
      margin-bottom: 1.5rem;
    }

    h3 {
      position: relative;
      font-size: 1.4rem;
      font-weight: 600;
      display: table;
      margin-top: 1.5rem;
      padding-top: 1rem;
      margin-bottom: 1.5rem;
    }

    p,
    li {
      line-height: 2;
    }

    a {
      text-decoration: none;
      font-weight: bold;
    }

    pre {
      border-radius: .375rem;
    }

    blockquote {
      font-weight: 400;
      font-style: normal;
      color: #718096;
      border-left-width: .25rem;
      border-left-color: #e2e8f0;
      quotes: "\201C""\201D""\2018""\2019";
      margin-top: 1.6em;
      margin-bottom: 1.6em;
      padding-left: 1em;
    }

    .nuxt-content-highlight {
      position: relative;
      > .filename {
        position: absolute;
        right: 0;
        top: 0;
        --text-opacity: 1;
        color: #f7fafc;
        color: rgba(247,250,252,var(--text-opacity));
        z-index: 10;
        font-family: DM Mono,Menlo,Monaco,Consolas,"Liberation Mono","Courier New",monospace;
        font-size: .875rem;
        line-height: 1.25rem;
        letter-spacing: -.025em;
        line-height: 1;
        margin-right: 1rem;
        margin-top: .75rem;
      }
    }

    code[class*=language-],
    pre[class*=language-] {
      margin: 1.5em 0;
      text-align: left;
      white-space: pre;
      word-spacing: normal;
      word-break: normal;
      word-wrap: normal;
      color: #c3cee3;
      background: #263238;
      font-family: Roboto Mono,monospace;
      font-size: 1em;
      line-height: 1.5em;
      -moz-tab-size: 4;
      -o-tab-size: 4;
      tab-size: 4;
      -webkit-hyphens: none;
      -ms-hyphens: none;
      hyphens: none;
    }
  }
</style>
