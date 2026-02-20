<template>
  <div class="container">
    <div class="card">
      <h2>Create Account</h2>

      <form @submit.prevent="register">

        <!-- NAME -->
        <div class="form-group">
          <label>Name</label>
          <input
            v-model="form.name"
            type="text"
            placeholder="Enter your name"
          />
          <small v-if="errors.name" class="error">{{ errors.name }}</small>
        </div>

        <!-- EMAIL -->
        <div class="form-group">
          <label>Email</label>
          <input
            v-model="form.email"
            type="email"
            placeholder="Enter your email"
          />
          <small v-if="errors.email" class="error">{{ errors.email }}</small>
        </div>

        <!-- PASSWORD -->
        <div class="form-group">
          <label>Password</label>
          <input
            v-model="form.password"
            type="password"
            placeholder="Enter strong password"
            @input="checkPasswordStrength"
          />

          <!-- Password strength -->
          <div class="password-strength">
            Strength:
            <span :class="strengthClass">{{ passwordStrength }}</span>
          </div>

          <small class="guide">
            Must contain: 8+ characters, uppercase, lowercase, number
          </small>

          <small v-if="errors.password" class="error">
            {{ errors.password }}
          </small>
        </div>

        <!-- BUTTON -->
        <button :disabled="loading">
          {{ loading ? "Registering..." : "Register" }}
        </button>

        <p v-if="message" class="success">{{ message }}</p>

      </form>
    </div>
  </div>
</template>

<script>
import api from '@/services/api'

export default {
  data() {
    return {
      form: {
        name: '',
        email: '',
        password: ''
      },
      errors: {},
      message: '',
      loading: false,
      passwordStrength: '',
    }
  },

  computed: {
    strengthClass() {
      if (this.passwordStrength === 'Weak') return 'weak'
      if (this.passwordStrength === 'Medium') return 'medium'
      if (this.passwordStrength === 'Strong') return 'strong'
      return ''
    }
  },

  methods: {
    validate() {
      this.errors = {}

      if (!this.form.name) {
        this.errors.name = 'Name is required'
      }

      if (!this.form.email) {
        this.errors.email = 'Email is required'
      } else if (!/\S+@\S+\.\S+/.test(this.form.email)) {
        this.errors.email = 'Invalid email format'
      }

      if (!this.form.password) {
        this.errors.password = 'Password is required'
      } else if (this.form.password.length < 8) {
        this.errors.password = 'Password must be at least 8 characters'
      }

      return Object.keys(this.errors).length === 0
    },

    checkPasswordStrength() {
      const pwd = this.form.password

      const hasUpper = /[A-Z]/.test(pwd)
      const hasLower = /[a-z]/.test(pwd)
      const hasNumber = /[0-9]/.test(pwd)
      const longEnough = pwd.length >= 8

      if (hasUpper && hasLower && hasNumber && longEnough) {
        this.passwordStrength = 'Strong'
      } else if (pwd.length >= 6) {
        this.passwordStrength = 'Medium'
      } else {
        this.passwordStrength = 'Weak'
      }
    },

    async register() {
      this.message = ''

      if (!this.validate()) return

      try {
        this.loading = true

        const response = await api.post('/register', this.form)

        this.message = response.data.message

        this.form.name = ''
        this.form.email = ''
        this.form.password = ''
        this.passwordStrength = ''

      } catch (error) {
        if (error.response && error.response.data.errors) {
          this.errors = error.response.data.errors
        } else {
          this.message = 'Registration failed'
        }
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>
.container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #f4f6f9;
  padding: 20px;
}

.card {
  background: #ffffff;
  width: 100%;
  max-width: 500px;
  padding: 40px;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.08);
}

h2 {
  text-align: center;
  margin-bottom: 30px;
  font-weight: 600;
}

.form-group {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
}

label {
  font-size: 14px;
  font-weight: 500;
}

input {
  padding: 12px;
  border-radius: 8px;
  border: 1px solid #ddd;
  margin-top: 6px;
  font-size: 14px;
  transition: 0.2s;
}

input:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 2px rgba(66, 184, 131, 0.1);
}

button {
  width: 100%;
  padding: 14px;
  border: none;
  border-radius: 8px;
  background-color: #42b883;
  color: white;
  font-weight: 600;
  font-size: 15px;
  cursor: pointer;
  margin-top: 10px;
  transition: 0.2s;
}

button:hover {
  background-color: #369870;
}

button:disabled {
  background-color: #9cd6bf;
  cursor: not-allowed;
}

.error {
  color: #e53935;
  font-size: 12px;
  margin-top: 4px;
}

.success {
  color: #2e7d32;
  text-align: center;
  margin-top: 15px;
}

.password-strength {
  font-size: 12px;
  margin-top: 5px;
}

.weak {
  color: #e53935;
}

.medium {
  color: #fb8c00;
}

.strong {
  color: #2e7d32;
}

.guide {
  font-size: 11px;
  color: #777;
  margin-top: 4px;
}
</style>