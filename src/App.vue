<template>
  <v-app id="inspire">
    <v-app-bar
        app
        dark
        flat
    >
      <v-toolbar-title>הטבות כרטיסי אשראי</v-toolbar-title>

      <v-spacer></v-spacer>
      <v-responsive max-width="260">
        <v-text-field
            v-model="search"
            clearable
            flat
            dense
            solo-inverted
            hide-details
            rounded
            prepend-inner-icon="mdi-magnify"
            label="חיפוש"
        ></v-text-field>
      </v-responsive>
    </v-app-bar>

    <v-main class="grey lighten-3">
      <v-container>
        <v-row>
          <v-col cols="3">
            <v-sheet rounded="lg" class="my-2">
              <br />
              <h2 class="mx-5">כרטיסי אשראי</h2>
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
                        <v-list-item-title>{{ provider }}</v-list-item-title>
                      </v-list-item-content>
                    </template>
                  </v-list-item>
                </v-list-item-group>
              </v-list>
              <v-btn text link @click="selectedProviders = Object.values(providers).map((val, index) => index)">בחר הכל</v-btn>
              <v-btn text link @click="selectedProviders = []">אפס הכל</v-btn>
              <br />
              <br />
              <v-divider />
              <br />

              <h2 class="mx-5">קטגוריות</h2>
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
                        <v-list-item-title >{{ category }}</v-list-item-title>
                      </v-list-item-content>
                    </template>
                  </v-list-item>
                </v-list-item-group>
              </v-list>
              <v-btn text link @click="selectedCategories = Object.values(categories).map((val, index) => index)">בחר הכל</v-btn>
              <v-btn text link @click="selectedCategories = []">אפס הכל</v-btn>
              <br />
              <br />
            </v-sheet>
          </v-col>

          <v-col cols="9">
            <v-sheet
                class="my-5 px-3"
                min-height="70vh"
                rounded="lg"
            >
              <v-alert v-if="search">מציג תוצאות חיפות עבור <strong>{{search}}</strong></v-alert>
              <v-data-iterator
                  :items="items"
                  :items-per-page.sync="itemsPerPage"
                  :page.sync="page"
                  :search="search"
                  :sort-by="sortBy.toLowerCase()"
                  :sort-desc="sortDesc"
                  hide-default-footer
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
                            {{ item.provider }}
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
                      class="mt-2 mb-5"
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

                    <span
                        class="mr-4
            grey--text"
                    >
            עמוד {{ page }} מתוך {{ numberOfPages }}
          </span>
                    <v-btn
                        fab
                        dark
                        color="blue darken-3"
                        class="mr-1"
                        @click="formerPage"
                    >
                      <v-icon>mdi-chevron-right</v-icon>
                    </v-btn>
                    <v-btn
                        fab
                        dark
                        color="blue darken-3"
                        class="ml-1"
                        @click="nextPage"
                    >
                      <v-icon>mdi-chevron-left</v-icon>
                    </v-btn>
                  </v-row>
                </template>
              </v-data-iterator>
            </v-sheet>
          </v-col>
        </v-row>
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

  data () {
    // shuffle array
    for (let i = data.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [data[i], data[j]] = [data[j], data[i]];
    }


    return {
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
    nextPage () {
      if (this.page + 1 <= this.numberOfPages) this.page += 1
    },
    formerPage () {
      if (this.page - 1 >= 1) this.page -= 1
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
      this.items = this.items.filter(item => selectedCategories.find(category => category === item.category))
    },
  },
  watch: {
    selectedCategories: {
      handler(selectedCategoriesIndexes, oldValue) {
        this.applyFilters()
      },
      immediate: true
    },
    selectedProviders: {
      handler(selectedProvidersIndexes, oldValue) {
        this.applyFilters()
      },
      immediate: true
    }
  },
});
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Assistant&display=swap');

html, body {
  font-family: 'Assistant', sans-serif;
}

#inspire {
  font-family: 'Assistant', sans-serif;
}
</style>
