<template>
  <div class = 'page-login'>
    <div class = "columns">
      <div class = "column is-4 is-offset-4">
        <h1 class = "title has-text-centered"> Prisijungimas </h1>
        <form @submit.prevent = "submitForm">

          <div class = "field">
            <label> El. paštas </label>
            <div class="control has-icons-left">
              <input type = "text" class = "input" placeholder = "El. Paštas" v-model = "username">
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
          
          <div class = "notification is-danger has-text-centered" v-if = "errors.length">
            <p v-for = "error in errors" v-bind:key = "error" > {{error}} </p>
          </div>

          <div class = "field has-text-centered">
            <div class = "control">
              <button class = "button is-dark"> Prisijungti </button>
            </div>
          </div>

          <hr>

          Arba <router-link to= '/sign-up' >spauskite čia</router-link>, kad užsiregistruoti.

        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Login',
  data() {
    return{
      username:'',
      password:'',
      errors: []
    }
  },
  mounted() {
    document.title = "Prisijungimas | Vienam gale kablys"
  },
  methods: {
    submitForm() {
      this.errors = []
      axios.defaults.headers.common["Authorization"] = ""
      localStorage.removeItem("token")
      const formData = {
        username: this.username,
        password: this.password
      }
      axios
        .post("/api/v1/token/login/", formData)
        .then(response => {
            const token = response.data.auth_token
            this.$store.commit('setToken', token)

            axios.defaults.headers.common["Authorization"] = "Token " + token
            localStorage.setItem("token", token)
            const toPath = this.$route.query.to || '/cart'
            this.$router.push(toPath)
        })
        .catch(error => {
            if (error.response) {
                for (const property in error.response.data) {
                    this.errors.push(`${property}: ${error.response.data[property]}`)
                }
            } else {
                this.errors.push('Kažkas ne taip. Bandykite dar kartą')

                console.log(JSON.stringify(error))
            }
        })
    }
  }
}
</script>
