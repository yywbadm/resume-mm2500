<!-- eslint-disable no-irregular-whitespace -->
<template>
  <div class="container mx-auto pt-8 pl-5 pr-5 text-black dark:text-white transition-colors duration-100">
    <!-- navbar -->
    <nav class="text-sm inline-block">
      <!-- left router link -->
      <div class="inline-block" v-for="(item, key) in nav_router" :key="key">
        <router-link
          :to="item.path"
          >
          {{ t(`Navbar.${item.name}`) }}
        </router-link>
        <p class="inline" v-if="key != nav_router.length - 1">｜</p>
      </div>

      <!-- right outside link -->
      <div class="inline absolute right-0">
        <div class="relative right-10">

          <div v-for="(item, key) in nav_link" :key="key" class="inline">
            <a :href="item.path" target="_blank">
              <font-awesome-icon :icon="item.icon" />
            </a>
            <p class="inline">　</p>
          </div>

          <!-- toggle dark mode & light mode -->
          <div class="inline">
            <button @click="toggleDarkMode()">
              <font-awesome-icon :icon="darkMode? ['fas', 'moon']: ['fas', 'sun']" />
            </button>
            <p class="inline">　</p>
          </div>

          <!-- toggle dark mode & light mode -->
          <!-- <font-awesome-icon :icon="['fas', 'earth-asia']" /> -->
          <select v-model="locale" class="hidden sm:inline bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 p-0.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500">
            <option v-for="(item, key) in lang.availableLocales"
              :key="key"
              class="text-gray-500">
              {{ item }}
            </option>
          </select>
        </div>
      </div>
    </nav>

    <!-- view -->
    <section class="mt-5">
      <!-- <button @click="increase(20)">increce</button>
      <button @click="fail()">failed</button>
      <button @click="finish()">finish</button> -->

      <router-view v-slot="{ Component }">
        <transition appear
          @leave="
            (el, done) => {
              gsap.to(el, {
                opacity: 0,
                duration: 0.5,
                onComplete: done
              })
            }">
          <component :is="Component" />
        </transition>
      </router-view>

      <vue-progress-bar></vue-progress-bar>
    </section>

    <!-- Footer Copyright -->
    <Footer></Footer>
  </div>
</template>

<script>
import Footer from './components/Footer.vue'
import SITECFG from './config/SITECFG.json'
import { useI18n } from 'vue-i18n'
import GSAP from 'gsap'

export default {
  name: 'App',
  components: {
    // eslint-disable-next-line vue/no-reserved-component-names
    Footer
  },
  setup() {
    const { t, locale } = useI18n()
    const lang = useI18n()
    return {
      t,
      locale,
      lang
    }
  },
  data() {
    return {
      SITE_CONFIG: SITECFG,
      darkMode: false,
      gsap: GSAP,
      nav_router: [
        {
          name: 'HomePage',
          path: '/'
        },
        {
          name: 'Works',
          path: '/works'
        },
        {
          name: 'Settings',
          path: '/setting'
        }
      ],
      nav_link: [
        {
          name: 'yyw網站',
          path: 'https://www.yywtw.cf',
          icon: ['fas', 'house']
        },
        {
          name: 'yyw部落格',
          path: 'https://blog.yywtw.cf',
          icon: ['fas', 'pen-to-square']
        },
        {
          name: '聯絡yyw',
          path: 'https://vtuber.group',
          icon: ['fas', 'comment-dots']
        }
      ]
    }
  },
  methods: {

    // 切換 dark mode & light mode
    toggleDarkMode() {
      this.darkMode = !this.darkMode
      localStorage.setItem('darkMode', this.darkMode)
    },

    // progress bar
    start() {
      this.$Progress.start()
    },
    set(num) {
      this.$Progress.set(num)
    },
    increase(num) {
      this.$Progress.increase(num)
    },
    decrease(num) {
      this.$Progress.decrease(num)
    },
    finish() {
      this.$Progress.finish()
    },
    fail() {
      this.$Progress.fail()
    },
    test() {
      // test func
      this.$Progress.start()

      this.$http.jsonp('http://api.rottentomatoes.com/api/public/v1.0/lists/movies/in_theaters.json?apikey=7waqfqbprs7pajbz28mqf6vz')
        .then((response) => {
          this.$Progress.finish()
        }, (response) => {
          this.$Progress.fail()
        })
    }
  },
  watch: {
    darkMode: function(val) {
      if (val) {
        document.documentElement.classList.add('dark')
      } else {
        document.documentElement.classList.remove('dark')
      }
    },
    locale: function(val) {
      localStorage.setItem('locale', val)
    }
  },
  mounted() {
    //  [App.vue specific] When App.vue is finish loading finish the progress bar
    this.$Progress.finish()
  },
  created() {
    if (localStorage.darkMode === 'true') {
      this.darkMode = true
    } else {
      this.darkMode = false
    }

    // progress bar of starting
    this.$Progress.start()
    //  hook the progress bar to start before we move router-view
    this.$router.beforeEach((to, from, next) => {
      // 自動標題(會從 router 裡面的 title 去抓)
      document.title = 'yyw履歷表 | ' + to.name
      // next()
      //  does the page we want to go to have a meta.progress object
      if (to.meta.progress !== undefined) {
        const meta = to.meta.progress
        // parse meta tags
        this.$Progress.parseMeta(meta)
      }
      //  start the progress bar
      this.$Progress.start()
      //  continue to next page
      next()
    })
    //  hook the progress bar to finish after we've finished moving router-view
    this.$router.afterEach((to, from) => {
      //  finish the progress bar
      this.$Progress.finish()
    })
  }
}
</script>
