<template>
  <v-app id="inspire">
    <v-navigation-drawer
        app
        right
        v-model="drawer"
        clipped
        color="transparent"
        width="300"
    >

      <v-sheet
          class="my-5 px-3 mx-3"
          min-height="70vh"
          rounded="lg"
      >
        <h3 class="mx-5 mt-5 pt-5">חיפוש הטבות</h3>
        <v-text-field
            class="my-3"
            v-model="search"
            clearable
            flat
            dense
            solo-inverted
            hide-details
            prepend-inner-icon="mdi-magnify"
            label="טקסט חופשי"
        ></v-text-field>
        <v-divider />

        <h3 class="mx-5 mt-5">כרטיסי אשראי</h3>
        <v-list color="transparent">
          <v-list-item-group
              v-model="selectedProviders"
              multiple
          >
            <v-list-item v-for="provider in providers" :key="`provider-${provider}`">
              <template v-slot:default="{ active }">
                <v-list-item-action>
                  <v-checkbox :input-value="active"></v-checkbox>
                </v-list-item-action>

                <v-list-item-content>
                  <v-list-item-title>{{ translateProvider(provider) }}</v-list-item-title>
                </v-list-item-content>
              </template>
            </v-list-item>
          </v-list-item-group>
        </v-list>
        <div style="text-align: center">
          <v-btn text link @click="selectedProviders = Object.values(providers).map((val, index) => index)">בחר הכל</v-btn>
          <v-btn text link @click="selectedProviders = []">אפס הכל</v-btn>
        </div>

        <v-divider />

        <h3 class="mx-5 mt-5">קטגוריות</h3>
        <v-list color="transparent">
          <v-list-item-group
              v-model="selectedCategories"
              multiple
              active-class=""
          >
            <v-list-item v-for="category in categories" :key="`category-${category}`">
              <template v-slot:default="{ active }">
                <v-list-item-action>
                  <v-checkbox :input-value="active"></v-checkbox>
                </v-list-item-action>

                <v-list-item-content>
                  <v-list-item-title >{{ translateCategory(category) }}</v-list-item-title>
                </v-list-item-content>
              </template>
            </v-list-item>
          </v-list-item-group>
        </v-list>
        <div style="text-align: center">
          <v-btn text link @click="selectedCategories = Object.values(categories).map((val, index) => index)">בחר הכל</v-btn>
          <v-btn text link @click="selectedCategories = []">אפס הכל</v-btn>
        </div>
      </v-sheet>


    </v-navigation-drawer>
    <v-app-bar
        app
        dark
        flat
        clipped-right
    >
      <v-toolbar-title>הטבות ומבצעים בכרטיסי אשראי</v-toolbar-title>



      <v-spacer></v-spacer>

      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>

    </v-app-bar>
    <v-main class="grey lighten-3">
      <v-container>
        <v-sheet
            class="my-2 px-3 py-3"
            min-height="70vh"
            rounded="lg"
        >
          <v-alert
              border="left"
              close-text="Close Alert"
              color="deep-purple accent-4"
              dark
              dismissible
          >
            האתר נועד לרכז את כל ההטבות והמבצעים שחברות כרטיסי האשראי מציעות.
            כאן תוכלו לחפש הטבה ספציפית, לסנן לפי קטגוריה / כרטיס אשראי,
            ולהשוות הטבות ומבצעים בין כרטיסי אשראי שונים.
          </v-alert>

          <v-alert color="success" v-if="search">
            <div style="color: white">מציג תוצאות חיפות עבור <strong>{{search}}</strong></div>
          </v-alert>
          <v-data-iterator
              :items="items"
              :items-per-page.sync="itemsPerPage"
              :page.sync="page"
              :sort-by="sortBy.toLowerCase()"
              :sort-desc="sortDesc"
              hide-default-footer
              no-data-text="אין תוצאות"
              no-results-text="אין תוצאות"
          >


            <template v-slot:default="props">
              <v-row>
                <v-col
                    v-for="item in props.items"
                    :key="item.url"
                    cols="12"
                    sm="6"
                    md="4"
                    lg="3"
                >
                  <v-card :href="item.url" target="_blank">
                    <v-img
                        height="250"
                        :src="item.image"
                    >

                      <v-chip
                          class="ma-2"
                          :color="getProviderColor(item.provider)"
                          text-color="white"
                          small
                      >
                        {{ translateProvider(item.provider) }}
                      </v-chip>

                      <v-chip
                          class="ma-2"
                          color="yellow"
                          text-color="black"
                          small
                      >
                        {{ translateCategory(item.category) }}
                      </v-chip>
                    </v-img>

                    <v-card-title class="subheading font-weight-bold">
                      {{ item.title }}
                    </v-card-title>

                  </v-card>
                </v-col>
              </v-row>
            </template>

            <template v-slot:footer>
              <v-row
                  v-if="items && items.length > 0"
                  class="px-5 py-5"
                  align="center"
                  justify="center"
              >
                <span class="grey--text">מס' פריטים בעמוד</span>
                <v-menu offset-y>
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn
                        dark
                        text
                        color="primary"
                        class="ml-2"
                        v-bind="attrs"
                        v-on="on"
                    >
                      {{ itemsPerPage }}
                      <v-icon>mdi-chevron-down</v-icon>
                    </v-btn>
                  </template>
                  <v-list>
                    <v-list-item
                        v-for="(number, index) in itemsPerPageArray"
                        :key="index"
                        @click="updateItemsPerPage(number)"
                    >
                      <v-list-item-title>{{ number }}</v-list-item-title>
                    </v-list-item>
                  </v-list>
                </v-menu>

                <v-spacer></v-spacer>

                <span class="ml-4 grey--text">עמוד {{ page }} מתוך {{ numberOfPages }}</span>
                <v-btn
                    fab
                    dark
                    class="ml-1"
                    @click="formerPage"
                >
                  <v-icon>mdi-chevron-right</v-icon>
                </v-btn>
                <v-btn
                    fab
                    dark
                    class="mr-1"
                    @click="nextPage"
                >
                  <v-icon>mdi-chevron-left</v-icon>
                </v-btn>
              </v-row>
            </template>
          </v-data-iterator>
        </v-sheet>

      </v-container>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from 'vue';
import data from '@/assets/data.json'

enum PROVIDER {
  CAL = 'cal',
  MAX = 'max',
  ISRACARD = 'isracard',
}

enum CATEGORY {
  ATTRACTIONS = 'attractions',
  CINEMA = 'cinema',
  PARENTS = 'parents',
  ENTERTAINMENT = 'entertainment',
  STANDUP = 'standup',
  TRAVELS = 'travels',
  FOOD = 'food',
  MISC = 'misc',
}

export default Vue.extend({
  name: 'App',

  beforeDestroy () {
    if (typeof window === 'undefined') return

    window.removeEventListener('resize', this.onResize, { passive: true })
  },

  mounted () {
    this.onResize()

    window.addEventListener('resize', this.onResize, { passive: true })

    this.drawer = !this.isMobile
  },


  data () {
    // shuffle array
    for (let i = data.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [data[i], data[j]] = [data[j], data[i]];
    }


    return {
      drawer: false,
      group: null,
      isMobile: false,
      itemsPerPageArray: [24, 48, 72],
      search: '',
      filter: {},
      sortDesc: false,
      page: 1,
      itemsPerPage: 24,
      sortBy: 'provider',
      keys: [
        'provider',
        'category',
        'title',
        // 'description',
      ],
      items: data,
      originalItems: data,
      links: [
        'Dashboard',
        'Messages',
        'Profile',
        'Updates',
      ],
      categories: Object.values(CATEGORY),
      selectedCategories: Object.values(CATEGORY).map((val, index) => index),
      providers: Object.values(PROVIDER),
      selectedProviders: Object.values(PROVIDER).map((val, index) => index),

    }
  },
  computed: {
    numberOfPages () {
      return Math.ceil((this as any).items.length / (this as any).itemsPerPage)
    },
    filteredKeys () {
      return (this as any).keys.filter(key => key !== 'title')
    },
  },
  methods: {
    onResize () {
      (this as any).isMobile = window.innerWidth < 800
    },
    nextPage () {
      if (this.page + 1 <= this.numberOfPages) this.page += 1
      window.scrollTo({
        top: 0,
        left: 0,
        behavior: 'smooth'
      });

    },
    formerPage () {
      if (this.page - 1 >= 1) this.page -= 1
      window.scrollTo({
        top: 0,
        left: 0,
        behavior: 'smooth'
      });

    },
    updateItemsPerPage (number) {
      this.itemsPerPage = number
    },
    getProviderColor (provider) {
      switch (provider) {
        case 'max':
          return 'black';
        case 'cal':
          return 'blue';
        case 'isracard':
          return 'red'
      }
    },
    applyFilters() {
      console.log('applyFilters this.selectedCategories', this.selectedCategories)
      console.log('applyFilters this.selectedProviders', this.selectedProviders)
      const selectedCategories = this.selectedCategories.map(selectedIndex => Object.values(CATEGORY)[selectedIndex]);
      const selectedProviders = this.selectedProviders.map(selectedIndex => Object.values(PROVIDER)[selectedIndex]);
      this.items = this.originalItems.filter(item => selectedProviders.find(provider => provider === item.provider));
      this.items = this.items.filter(item => selectedCategories.find(category => category === item.category));
      if (this.search && this.search.length > 0) {
        this.items = this.items.filter(item => item.title.indexOf(this.search) > -1);
      }
    },
    translateProvider(provider) {
      switch (provider) {
        case PROVIDER.CAL:
          return 'כאל'
        case PROVIDER.ISRACARD:
          return 'ישראכארט'
        case PROVIDER.MAX:
          return 'מקס'
      }
    },
    translateCategory(category) {
      switch (category) {
        case CATEGORY.CINEMA:
          return 'קולנוע';
        case CATEGORY.FOOD:
          return 'אוכל';
        case CATEGORY.ATTRACTIONS:
          return 'ארקציות';
        case CATEGORY.MISC:
          return 'שונות';
        case CATEGORY.TRAVELS:
          return 'טיולים וחופשות';
        case CATEGORY.ENTERTAINMENT:
          return 'בידור ופנאי';
        case CATEGORY.PARENTS:
          return 'ילדים';
        case CATEGORY.STANDUP:
          return 'סטנד אפ';
      }
    },
  },
  watch: {
    selectedCategories: {
      handler(newValue, oldValue) {
        this.applyFilters()
      },
      immediate: true
    },
    selectedProviders: {
      handler(newValue, oldValue) {
        this.applyFilters()
      },
      immediate: true
    },
    search: {
      handler(newValue, oldValue) {
        this.applyFilters()
      },
      immediate: true
    },
    group () {
      this.drawer = false
    },
  },
});
</script>

<style lang="scss">
html, body {
  font-family: 'Assistant', sans-serif;
}

#inspire {
  font-family: 'Assistant', sans-serif;
}
</style>
