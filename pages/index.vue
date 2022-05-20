<template>
  <v-row justify="center" align="center">
    <v-col cols="12" md="8">
      <v-card flat class="logo py-4 d-flex justify-center">
        <NuxtLogo />
        <VuetifyLogo />
      </v-card>
      <!-- <v-card>
        <v-card-title class="headline">
          Welcome to the Vuetify + Nuxt.js template
        </v-card-title>
        <v-card-text>
          <p>Vuetify is a progressive Material Design component framework for Vue.js. It was designed to empower developers to create amazing applications.</p>
          <p>
            For more information on Vuetify, check out the <a
              href="https://vuetifyjs.com"
              target="_blank"
              rel="noopener noreferrer"
            >
              documentation
            </a>.
          </p>
          <p>
            If you have questions, please join the official <a
              href="https://chat.vuetifyjs.com/"
              target="_blank"
              rel="noopener noreferrer"
              title="chat"
            >
              discord
            </a>.
          </p>
          <p>
            Find a bug? Report it on the github <a
              href="https://github.com/vuetifyjs/vuetify/issues"
              target="_blank"
              rel="noopener noreferrer"
              title="contribute"
            >
              issue board
            </a>.
          </p>
          <p>Thank you for developing with Vuetify and I look forward to bringing more exciting features in the future.</p>
          <div class="text-xs-right">
            <em><small>&mdash; John Leider</small></em>
          </div>
          <hr class="my-3">
          <a
            href="https://nuxtjs.org/"
            target="_blank"
            rel="noopener noreferrer"
          >
            Nuxt Documentation
          </a>
          <br>
          <a
            href="https://github.com/nuxt/nuxt.js"
            target="_blank"
            rel="noopener noreferrer"
          >
            Nuxt GitHub
          </a>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="primary"
            nuxt
            to="/inspire"
          >
            Continue
          </v-btn>
        </v-card-actions>
      </v-card> -->
    </v-col>
    <v-col cols="12" md="8" class="news-container">
      <h2 class="text-h4">
        Latest
      </h2>
      <ul class="news-list">
        <li class="news-list__item">
          <v-card v-for="article in blog" :key="article.slug" outlined class="card">
            <nuxt-link :to="'/blog/' + article.slug">
              <time :datetime="article.createdAt" class="card__date text--primary text-body-2">
                {{ $dateFns.format(new Date(article.createdAt), 'yyyy/MM/dd') }}
              </time>
              <v-card-title class="font-weight-bold black--text">
                {{ article.title }}
              </v-card-title>
              <v-card-text>
                <v-chip-group column>
                  <v-chip
                    v-for="tag in article.tags"
                    :key="tag"
                    small
                    color="indigo darken-1"
                    class="white--text"
                  >
                    # {{ tag }}
                  </v-chip>
                </v-chip-group>
              </v-card-text>
            </nuxt-link>
          </v-card>
        </li>
      </ul>
    </v-col>
  </v-row>
</template>

<script>
export default {
  async asyncData ({ $content }) {
    const blog = await $content('blog').sortBy('createdAt', 'desc').fetch()
    return {
      blog
    }
  },
  head() {
    return {
      title: 'Blog',
      titleTemplate: null
    }
  }
}

// import { Context } from '@nuxt/types'
// import { Vue, Component } from 'nuxt-property-decorator'

// @Component
// export default class IndexComponent extends Vue {
//   head() {
//     return {
//       title: 'blog',
//       titleTemplate: null
//     }
//   }
//   async asyncData ({ $content }: Context) {
//     const blog = await $content('blog').sortBy('createdAt', 'desc').fetch()
//     return {
//       blog
//     }
//   }
// }
</script>

<style lang="scss" scoped>
.news-container {
  margin: 2.5rem 0;
}

.news-list {
  padding: 0;
  list-style: none;
}

.card {
  font-family: YuGothic, Roboto, sans-serif !important;
  margin-top: 1.5rem;
  a {
    display: block;
    padding: 16px;
    text-decoration: none;
  }
  &__date {
    display: block;
    padding: 16px 16px 0;
  }
}
</style>
