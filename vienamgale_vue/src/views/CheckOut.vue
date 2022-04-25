<template>
<div class = "page-checkout">
  <div class = "columns is-multiline">

    <div class ="column is-12">
      <h1 class = "title">Atsiskaitymas</h1>
    </div>

    <div class="column is-12 box">
      <table class="table is-fullwidth" v-if="cartTotalLength">
        <thead>
          <tr>
            <th>Prekė</th>
            <th>Kaina</th>
            <th>Kiekis</th>
            <th>Viso</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr
              v-for="item in cart.items"
              v-bind:key="item.product.id"
          >
            <td>{{ item.product.name }}</td>
            <td>{{ item.product.price }} eur</td>
            <td>{{ item.quantity }}</td>
            <td>{{ getItemTotal(item).toFixed(2) }} eur</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="column is-12 box">
                <h2 class="subtitle">Siuntimo duomenys</h2>



                <div class="columns is-multiline">
                    <div class="column is-6">
                        <div class="field">
                            <label>Vardas*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="first_name">
                            </div>
                        </div>

                        <div class="field">
                            <label>Pavardė*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="last_name">
                            </div>
                        </div>

                        <div class="field">
                            <label>El. paštas*</label>
                            <div class="control">
                                <input type="email" class="input" v-model="email">
                            </div>
                        </div>

                        <div class="field">
                            <label>Telefono numeris*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="phone">
                            </div>
                        </div>
                    </div>

                    <div class="column is-6">
                        <div class="field">
                            <label>Adresas*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="address">
                            </div>
                        </div>

                        <div class="field">
                            <label>Pašto kodas*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="zipcode">
                            </div>
                        </div>

                        <div class="field">
                            <label>Vieta*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="place">
                            </div>
                        </div>
                    </div>
                </div>
                <p class="has-text-grey mb-4">*Būtina užpildyti visus laukelius</p>
                <div class="notification is-danger mt-4" v-if="errors.length">
                    <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
                </div>

                <hr>

                <div id="card-element" class="mb-5"></div>

                <template v-if="cartTotalLength">
                    <hr>
                    <button class="button is-success" @click="submitForm">Mokėti</button>
                </template>
            </div>

  </div>
</div>


</template>

<script>
import axios from 'axios'

export default {
    name: 'Checkout',
    data() {
        return {
            cart: {
                items: []
            },
            stripe: {},
            card: {},
            first_name: '',
            last_name: '',
            email: '',
            phone: '',
            address: '',
            zipcode: '',
            place: '',
            errors: []
        }
    },
    mounted() {
        document.title = 'Atsiskaitymas | Vienam gale kablys'
        this.cart = this.$store.state.cart
        if (this.cartTotalLength > 0) {
            this.stripe = Stripe('pk_test_51ELPw0GnhsO4RtFa7HIILjcjS8WMtFTtPk7NkTZJtQS4r4fySmUFsaQdNWS0WvcBrPygfHn2D97wgL1I9Wwnoaek00ATsqQQSM')
            const elements = this.stripe.elements();
            this.card = elements.create('card', { hidePostalCode: true })
            this.card.mount('#card-element')
        }
    },
    methods: {
        getItemTotal(item) {
            return item.quantity * item.product.price
        },
        removeFromCart() {
            this.cart.items = []
        },
        submitForm() {
          this.errors = []
          if (this.first_name === '') {
            this.errors.push('Neįvestas vardas')
          }
          if (this.last_name === '') {
            this.errors.push('Neįvesta pavardė')
          }
          if (this.email === '') {
            this.errors.push('Neįvestas el. pašto adresas')
          }
          if (this.phone === '') {
            this.errors.push('Neįvestas telefono numeris')
          }
          if (this.address === '') {
            this.errors.push('Neįvestas adresas')
          }
          if (this.zipcode === '') {
            this.errors.push('Neįvestas pašto kodas')
          }
          if (this.place === '') {
            this.errors.push('Neįvesta vieta')
          }
          if (!this.errors.length) {

            this.stripe.createToken(this.card).then(result => {
              if (result.error) {
                this.errors.push('Kažkas negerai su Stripe. Bandykite dar kartą')
                console.log(result.error.message)
              } else {
                this.stripeTokenHandler(result.token)
                this.removeFromCart()
              }
            })
          }
        },
        stripeTokenHandler(token) {
            const items = []
            for (let i = 0; i < this.cart.items.length; i++) {
                const item = this.cart.items[i]
                const obj = {
                    product: item.product.id,
                    quantity: item.quantity,
                    price: item.product.price * item.quantity
                }
                items.push(obj)
            }
            const data = {
                'first_name': this.first_name,
                'last_name': this.last_name,
                'email': this.email,
                'address': this.address,
                'zipcode': this.zipcode,
                'place': this.place,
                'phone': this.phone,
                'items': items,
                'stripe_token': token.id
            }
            axios
                .post('/api/v1/checkout/', data)
                .then(response => {
                    this.$store.commit('clearCart')
                    this.$router.push('/cart/success')
                })
                .catch(error => {
                    this.errors.push('Kažkas negerai. Bandykite dar kartą')
                    console.log(error)
                })
        }
    },
    computed: {
        cartTotalPrice() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.product.price * curVal.quantity
            }, 0)
        },
        cartTotalLength() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.quantity
            }, 0)
        }
    }
}
</script>
