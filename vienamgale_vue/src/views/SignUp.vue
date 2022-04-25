<template>
  <div class = "page-sign-up">
    <div class = "columns">
      <div class = "column is-4 is-offset-4">
        <h1 class = "title has-text-centered"> Registracija </h1>
        <form @submit.prevent = "submitForm">

          <div class = "field">
            <label> Vartotojo vardas </label>
            <div class="control has-icons-left">
              <input type = "text" class = "input" placeholder = "Vartotojo vardas" v-model = "username">
              <span class="icon is-small is-left">
                <i class="fas fa-user"></i>
              </span>
            </div>
          </div>

          <div class = "field">
            <label> El. paštas </label>
            <div class="control has-icons-left">
              <input type = "email" class = "input" placeholder = "El. Paštas" v-model = "email">
              <span class="icon is-small is-left">
                <i class="fas fa-envelope"></i>
              </span>
            </div>
          </div>

          <div class = "field">
            <label> Slaptažodis </label>
            <div class = "control has-icons-left">
              <input type = "password" class = "input" placeholder = "Slaptažodis" v-model = "password">
              <span class="icon is-small is-left">
                <i class="fas fa-lock"></i>
              </span>
            </div>
          </div>

          <div class = "field">
            <label> Pakartoti slaptažodį </label>
            <div class = "control has-icons-left">
              <input type = "password" class = "input" placeholder = "Pakartoti slaptažodį" v-model = "password2">
              <span class="icon is-small is-left">
                <i class="fas fa-lock"></i>
              </span>
            </div>
          </div>

          <div class = "notification is-danger has-text-centered" v-if = "errors.length">
            <p v-for = "error in errors" v-bind:key = "error" > {{error}} </p>
          </div>

          <div class = "field has-text-centered">
            <div class = "control">
              <button class = "button is-dark"> Registruotis </button>
            </div>
          </div>

          <hr>

          Arba <router-link to= '/log-in' >spauskite čia</router-link>, kad prisijungti.

        </form>
      </div>
    </div>
  </div>
</template>


<script>
import axios from'axios'
import { toast } from 'bulma-toast'

export default {
  name: 'SignUp',
  data() {
    return {
      username: '',
      email: '',
      password: '',
      password2: '',
      errors: []
    }
  },
  mounted(){
    document.title = "Registracija | Vienam gale kablys"
  },
  methods: {
    submitForm() {
      this.errors = []
      if (this.username === '') {
        this.errors.push('Nepateiktas vartotojo vardas')
      }
      if (this.password === '') {
        this.errors.push('Neįvedėte slaptažodžio')
      }
      if (this.password !== this.password2) {
        this.errors.push('Slaptažodžiai nesutampa')
      }
      if (!this.errors.length) {
        const formData = {
          username: this.username,
          email: this.email,
          password: this.password
        }
        axios
          .post("/api/v1/users/", formData)
          .then(response => {
            toast({
              message: 'Paskyra sukurta, prisijunkite.',
              type: 'is-success',
              dismissible: true,
              pauseOnHover: true,
              duration: 2000,
              position: 'bottom-right'
            })
            this.$router.push('/log-in')
          })
          .catch(error => {
            if (error.response) {
                for (const property in error.response.data) {
                    this.errors.push(`${property}: ${error.response.data[property]}`)
                }
                console.log(JSON.stringify(error.response.data))
            } else if (error.message) {
                this.errors.push('Kažkas ne taip, bandykite dar kartą.')

                console.log(JSON.stringify(error))
            }
          })
      }
    }
  }
}
</script>
