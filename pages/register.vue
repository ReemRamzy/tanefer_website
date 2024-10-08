<template>
  <div class="page-wrapper">
    <div class="background-color" />
    <div class="auth-page-wrapper">
      <div class="auth-wrapper">
        <h2>Register</h2>
        <form @submit.prevent="register">
          <div class="input-row">
            <input v-model="username" type="text" placeholder="Username" required>
            <input v-model="email" type="email" placeholder="Email" required>
          </div>
          <div class="input-row">
            <MobileInput @update="assignPhone" />
            <input v-model="password" type="password" placeholder="Password" required>
            <input v-model="password_confirmation" type="password" placeholder="Confirm Password" required>
          </div>
          <div class="terms-checkbox">
            <input
              id="terms"
              v-model="agreedToTerms"
              type="checkbox"
              required
            >
            <label for="terms">
              I agree to the
              <router-link to="/termsAndConditions">terms and conditions</router-link>.
            </label>
          </div>
          <button :disabled="!agreedToTerms" type="submit">
            Register
          </button>
          <div v-if="message" :class="{'success-message': isSuccess, 'error-message': !isSuccess}" class="message-box">
            {{ message }}
          </div>
        </form>
        <p v-if="Object.keys(validationErrors).length">
          <span v-for="(messages, field) in validationErrors" :key="field">
            <strong>{{ field }}:</strong> {{ messages.join(', ') }}
          </span>
        </p>
        <p class="no-account">
          Already have an account? <router-link to="/login">
            Login
          </router-link>
        </p>
        <div class="social-login">
          <!-- Buttons are invisible but still in the DOM -->
          <button class="facebook-btn" style="display: none;">
            <i class="fab fa-facebook-f" /> Register with Facebook
          </button>
          <button class="google-btn" style="display: none;">
            <i class="fab fa-google" /> Register with Google
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import clientAPI from '../services/axiosConfig'

export default {
  data () {
    return {
      username: '',
      email: '',
      password: '',
      phone: '',
      password_confirmation: '',
      validationErrors: {},
      message: '',
      isSuccess: true,
      agreedToTerms: false
    }
  },

  head () {
    return {
      title: 'Register Page',
      meta: [
        { hid: 'description', name: 'description', content: 'Register a new account' }
      ]
    }
  },

  methods: {
    async register () {
      try {
        // Register the user
        const response = await clientAPI('https://api.tanefer.com/api/v2/auth').post('/register', {
          username: this.username,
          email: this.email,
          phone: this.phone,
          password: this.password,
          password_confirmation: this.password_confirmation
        })

        const token = response.data.token

        // Store the token in localStorage
        localStorage.setItem('authToken', token)

        // Set the token for future requests
        clientAPI.defaults.headers.common.Authorization = `Bearer ${token}`

        // Update the message and success status
        this.message = 'Registered successfully!'
        this.isSuccess = true

        // Optionally, update your Vuex store if you have one
        this.$store.commit('auth/setToken', token)
        this.$store.dispatch('auth/fetchUser')

        // Redirect to the dashboard or home page
        this.$router.push('/') // Replace '/dashboard' with your desired route
      } catch (error) {
        if (error.response) {
          if (error.response.status === 422) {
            this.validationErrors = error.response.data.errors || {}
            this.message = error.response.data.message || 'Please check your input.'
          } else if (error.response.status === 404) {
            this.message = 'Endpoint not found. Please try again later.'
          } else {
            this.message = 'Registration failed. Please try again.'
          }
        } else {
          this.message = 'Network error. Please try again later.'
        }
        this.isSuccess = false
      }
    },

    assignPhone (phone) {
      this.phone = phone
    }
  }
}
</script>

<style scoped>
.page-wrapper {
  position: relative;
  min-height: 100vh;
}

.background-color {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #CFB9A1;
  z-index: -1;
}

.auth-page-wrapper {
  position: relative;
  padding: 80px 10px 10px;
}

.auth-wrapper {
  max-width: 400px;
  margin: 20px auto;
  padding: 20px;
  text-align: center;
}

h2 {
  margin-bottom: 20px;
}

.input-row {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 20px;
}

.input-row input {
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background: #fff;
  font-size: 16px;
  color: #333;
}

.input-row input::placeholder {
  color: #aaa;
}

button {
  padding: 15px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  background-color: #764322;
  color: #fff;
  font-size: 16px;
  margin-top: 10px;
}

button:hover {
  color: black;
  background-color: #cfb0a1;
}

.social-login {
  margin-top: 20px;
}

.social-login button {
  width: 100%;
  margin-top: 10px;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  display: none;
}

.facebook-btn {
  background-color: #3b5998;
  color: #fff;
}

.google-btn {
  background-color: #db4a39;
  color: #fff;
}

.facebook-btn:hover,
.google-btn:hover {
  color: black;
}

.fab {
  font-size: 20px;
}

.message-box {
  margin-top: 10px;
  padding: 10px;
  border-radius: 5px;
  font-size: 14px;
}

.success-message {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.error-message {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}

.terms-checkbox {
  margin: 20px 0;
  text-align: left;
  display: flex;
  align-items: center;
  gap: 10px;
}

.terms-checkbox input {
  margin: 0;
}

.terms-checkbox label {
  font-size: 14px;
}

.no-account{
  margin-top: revert;
}

</style>
