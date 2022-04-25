<template>
  <div class="home">
    <section class="hero is-medium is-dark mb-1">
        <div class="has-text-centered mb-6">
            <img src="../assets/vienamgale_logo.svg" width="500" height="600">

            <p class="subtitle">
                Žvejybos reikmenų parduotuvė
            </p>

        </div>
    </section>

    <div class="columns is-multiline">
      <div class="column is-12">
          <h2 class="is-size-2 has-text-centered">Naujausios prekės</h2>
      </div>

      <ProductBox
        v-for = 'product in latestProducts'
        v-bind:key = 'product.id'
        v-bind:product = 'product'/>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import ProductBox from '@/components/ProductBox'
export default {
  name: 'Home',
  data() {
    return {
      latestProducts: []
    }
  },
  components: {
    ProductBox
  },
  mounted() {
    this.getLatestProducts()
    document.title = 'Vienam gale kablys'

  },
  methods: {
    getLatestProducts() {
      axios
        .get('/api/v1/latest-products/')
        .then(response => {
          this.latestProducts = response.data
        })
        .catch(error => {
          console.log(error)
        })

    }
  }
}
</script>
