<template>
  <div class = "column is-2">
    <div class = "box">
      <router-link v-bind:to = "product.get_absolute_url" class = "image mb-3">
        <img :src="product.get_thumbnail">
      </router-link>

      <h3  class="is-size-4">
          <router-link v-bind:to = "product.get_absolute_url"> {{product.name}} </router-link>
      </h3>
      <p class = "is-size-6 has-text-grey"> {{product.price}} eur </p>
      <div class = "has-text-centered">
        <a @click = "addToCart()" class = "button is-dark mt-4">Pridėti į krepšelį</a>
      </div>
    </div>
  </div>
</template>

<script>
import {toast} from 'bulma-toast'
export default {
  name: 'ProductBox',
  props: {
    product: Object
  },
  methods:{
    addToCart() {
      if (isNaN(this.quantity) || this.quantity < 1) {
        this.quantity = 1
      }
      const item = {
        product: this.product,
        quantity: this.quantity
      }
      this.$store.commit('addToCart', item)

      toast({
        message: 'Prekė įdėta į krepšelį',
        type: 'is-success',
        dismissible: true,
        pauseOnHover: true,
        duration: 2000,
        position: 'bottom-right'
      })
    }
  },
}
</script>

<style scoped>
  .image {
    margin-top: -1rem;
    margin-left: -1rem;
    margin-right:-1rem;
  }
</style>
