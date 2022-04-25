<template>
  <div class = "page-search">
    <div class = "column is-multiline">
      <div class = "column is-12">
        <h1 class = "title"> Paieškos rezultatai frazei: "{{ query }}" </h1>
      </div>
      
        <ProductBox
          v-for = 'product in products'
          v-bind:key = 'product.id'
          v-bind:product = 'product'/>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import ProductBox from '@/components/ProductBox.vue'

export default {
  name: 'Search',
  components: {
    ProductBox
  },
  data() {
    return {
      products: [],
      query: ''
    }
  },
  mounted() {
    document.title = 'Paieška | Vienam gale kablys'
    let uri = window.location.search.substring(1)
    let params = new URLSearchParams(uri)

    if (params.get('query')) {
      this.query = params.get('query')
      this.performSearch()
    }
  },
  methods: {
    performSearch() {
      axios
        .post('/api/v1/products/search/', {'query': this.query})
        .then(response => {
          this.products = response.data
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>
