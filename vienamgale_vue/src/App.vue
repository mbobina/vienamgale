<template>
  <div id="wrapper">
    <nav class="navbar is-dark">
      <div class="navbar-brand">
        <router-link to="/" class="navbar-item"><img src="./assets/kablys_logo.svg"></router-link>
        <a class="navbar-burger" aria-label="menu" aria-expanded="false" data-target="navbar-menu" @click="showMobileMenu = !showMobileMenu">
          <span aria-hidden="true"></span>
          <span aria-hidden="true"></span>
          <span aria-hidden="true"></span>
        </a>
      </div>

      <div class="navbar-menu" id="navbar-menu" v-bind:class="{'is-active': showMobileMenu}">
        <div class="navbar-start">
          <div
            @mouseover="toggleDropdown(true)"
            @mouseleave="toggleDropdown(false)"
            class="navbar-item has-dropdown is-hoverable"
          >
            <a class="navbar-link">Prekių meniu</a>

            <div class="navbar-dropdown is-size-6" :style="{display: showDropdown ? 'block' : 'none' }">

              <router-link
              v-for="category in categories"
              v-bind:to ="category.get_absolute_url"
              class="navbar-item">
              {{category.name}}
              </router-link>
            </div>
          </div>

          <div class="navbar-item">
            <form method="get" action="/search">
              <div class="field has-addons">
                <div class="control">
                  <input type="text" class="input" placeholder="Paieška" name="query">
                </div>

                <div class="control">
                  <button class="button is-grey">
                    <span class="icon">
                      <i class="fas fa-search"></i>
                    </span>
                  </button>
                </div>

              </div>
            </form>
          </div>
        </div>

        <div class="navbar-end">
          <router-link to="/about" class="navbar-item">Apie</router-link>
          <router-link to="/contacts" class="navbar-item">Kontaktai</router-link>

          <div class="navbar-item">
            <div class="buttons">
              <template v-if = "$store.state.isAuthenticated">
               <router-link to="/myaccount" class="button is-light">Paskyra</router-link>
              </template>
              <template v-else>
               <router-link to="/log-in" class="button is-light">Prisijungti</router-link>
              </template>

              <router-link to="/cart" class="button is-success">
                <span class="icon"><i class="fas fa-shopping-cart"></i></span>
                <span>Krepšelis ({{ cartTotalLength }})</span>
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </nav>

    <section class="section">
      <router-view/>
    </section>

    <footer class="footer">
        <p class="has-text-centered"><router-link to="/about">Apie</router-link></p>
        <p class="has-text-centered"><router-link to="/contacts">Kontaktai</router-link></p>
        <p class="has-text-centered"> Vienam gale kablys (c) 2022</p>
    </footer>


  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      routeChange: false,
      showDropdown: false,
      showMobileMenu: false,
      category: {
        products: []
      },
      cart: {
        items: []
      }
    }
  },
  watch: {
    $route() {
      this.routeChange = true;
      this.showDropdown = false;
    }
  },
  beforeCreate() {
    this.$store.commit('initializeStore')
    const token = this.$store.state.token
    if (token) {
        axios.defaults.headers.common['Authorization'] = "Token " + token
    } else {
        axios.defaults.headers.common['Authorization'] = ""
    }
  },
  mounted() {
    this.cart = this.$store.state.cart,
    this.getCategoryList()
  },
  methods: {
    toggleDropdown(payload) {
      if (this.showDropdown !== payload) this.routeChange = false;
      if (!this.routeChange) {
        this.showDropdown = payload;
      }
    },
    getCategoryList() {
      const categorySlug = this.$route.params.category_slug
      axios
        .get(`/api/v1/products/categories/`)
        .then(response => {
          this.categories = response.data
        })
        .catch(error => {
          console.log(error)
        })
    }
  },
  computed: {
      cartTotalLength() {
          let totalLength = 0
          for (let i = 0; i < this.cart.items.length; i++) {
              totalLength += parseInt(this.cart.items[i].quantity)
          }
          return totalLength
      }
  }
}
</script>


<style lang="scss">
@import '../node_modules/bulma';
</style>
