<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { emailValidator, requiredValidator } from '@/utils/validator'
import { supabase, formActionDefault } from '@/utils/supabase.js'
import AlertNotification from '@/components/common/AlertNotification.vue'

// Utilities
const router = useRouter()

// Form data and state
const formDataDefault = {
  email: '',
  password: '',
}
const formData = ref({ ...formDataDefault })
const formAction = ref({ ...formActionDefault })

const refVForm = ref(null)
const visible = ref(false)

// Toggle password visibility
const toggleVisible = () => {
  visible.value = !visible.value
}

// Form submission
const onSubmit = async () => {
  formAction.value = { ...formActionDefault, formProcess: true }

  const { data, error } = await supabase.auth.signInWithPassword({
    email: formData.value.email,
    password: formData.value.password,
  })

  if (error) {
    formAction.value.formErrorMessage = error.message
    formAction.value.formStatus = error.status
  } else if (data) {
    console.log(data) // user data

    const { user } = data
    // Check if user is blocked
    if (user?.user_metadata?.is_blocked) {
      formAction.value.formErrorMessage = 'Your account has been blocked. Please contact the administrator.'
      formAction.value.formStatus = 'error'
    } else {
      // Check if user is an employer by checking the `is_employer` flag
      if (user?.user_metadata?.is_employer) {
        formAction.value.formSuccessMessage = 'Logged in as employer successfully'
        router.replace('/employerdashboard') // Redirect to employer dashboard
      } else {
        formAction.value.formSuccessMessage = 'Logged in successfully'
        router.replace('/jobdashboard') // Redirect to job dashboard
      }
    }
  }


  refVForm.value?.reset()
  formAction.value.formProcess = false
}

const loginWithGoogle = async () => {
  try {
    const { data, error } = await supabase.auth.signInWithOAuth({
      provider: 'google',
      options: {
        redirectTo: `${window.location.origin}/role`, // Default redirect
      },
    })

    if (error) {
      formAction.value.formErrorMessage = error.message
    } else {
      const user = data.user
      // Check if the user is blocked
      if (user?.user_metadata?.is_blocked) {
        formAction.value.formErrorMessage = 'Your account has been blocked. Please contact the administrator.'
        formAction.value.formStatus = 'error'
      } else {
        console.log('Google login successful', data)
      }
    }
  } catch (error) {
    console.error('Google OAuth login error:', error)
    formAction.value.formErrorMessage = 'Google login failed. Please try again.'
  }
}


// Validate and submit the form
const onFormSubmit = () => {
  refVForm.value?.validate().then(({ valid }) => {
    if (valid) onSubmit()
  })
}


</script>

<template>
  <AlertNotification
    :form-success-message="formAction.formSuccessMessage"
    :form-error-message="formAction.formErrorMessage"
  ></AlertNotification>
  <v-form ref="refVForm" fast-fail @submit.prevent="onFormSubmit">
    <v-text-field
      v-model="formData.email"
      label="Email"
      prepend-inner-icon="mdi-email"
      :rules="[requiredValidator, emailValidator]"
      class="modern-input"
      variant="outlined"
      density="compact"
      rounded
    ></v-text-field>
    <!-- password  -->
    <v-text-field
      v-model="formData.password"
      :append-inner-icon="visible ? 'mdi-eye-off' : 'mdi-eye'"
      :type="visible ? 'text' : 'password'"
      label="Password"
      prepend-inner-icon="mdi-lock-outline"
      @click:append-inner="toggleVisible"
      :rules="[requiredValidator]"
      class="modern-input"
      variant="outlined"
      density="compact"
      rounded
    ></v-text-field>
    <!-- checkbox  -->
    <v-row class="mt-2 align-center">
      <v-col cols="6">
        <v-row>
          <v-col cols="auto">
            <v-checkbox class="small-checkbox" hide-details>
              <template #label>
                <span class="remember-me-text"> Remember Me </span>
              </template>
            </v-checkbox>
          </v-col>
        </v-row>
      </v-col>
      <v-col cols="6" class="text-right">
        <span>Forgot Password?</span>
      </v-col>
    </v-row>

    <v-row class="button-row mt-4">
      <v-col>
        <v-btn
          class="login-button w-100 rounded-pill"
          depressed
          type="submit"
          :disabled="formAction.formProcess"
          :loading="formAction.formProcess"
          block
        >
          Login Now
        </v-btn>
      </v-col>
    </v-row>
    <v-divider></v-divider>
    <v-col>
      <h5>
        don't have an account?<router-link class="link" to="register">
          click here to register</router-link
        >
      </h5>
    </v-col>

    <v-divider class="my-4">Or</v-divider>
    <div class="social-icons d-flex justify-center">
      <v-btn
        prepend-icon="mdi-google"
        class="w-100 ma-10"
        @click="loginWithGoogle"
      >
        Sign In with Google
      </v-btn>
    </div>
  </v-form>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Matemasie&family=Varela+Round&display=swap');

*,
v-btn {
  font-family: 'Varela Round', sans-serif;
  font-weight: 400;
  font-style: normal;
  text-transform: none;
}
.v-btn {
  border-radius: 20px;
}
.small-checkbox .v-label {
  font-size: 0.85rem;
  font-weight: 400;
  color: #6c757d; /* This can be adjusted to match the link color */
}
.small-checkbox .v-input--selection-controls__input {
  transform: scale(0.8);
}
.link {
  text-decoration: none;
  font-size: 0.75rem;
  font-weight: 400;
  color: #6c757d; /* Link color */
}
.modern-input .v-label {
  font-size: 0.9rem; /* Smaller font for modern feel */
  font-weight: 500; /* Medium weight for a sleek look */
  color: #6c757d; /* Soft gray color */
  text-transform: uppercase;
  letter-spacing: 0.5px; /* Adds a bit of space for modern style */
}
.modern-input .v-input__control {
  font-size: 1rem; /* Standard font size for input text */
  font-weight: 400;
  color: #333; /* Darker color for input text */
}
.modern-input .v-input__icon--prepend,
.modern-input .v-input__icon--append {
  color: #2c3e50; /* Primary color for icons */
  opacity: 0.8; /* Slight opacity for subtle effect */
}
.modern-input .v-field__input {
  padding-top: 1rem; /* Extra space for a clean look */
}
.modern-input .v-input--is-dirty .v-label {
  color: #2c3e50; /* Primary color when the input is active */
}
.button-row .login-button {
  background-color: #4caf50; /* Primary green color */
  color: #ffffff;
  font-weight: 100;
  font-size: 0.5rem;
  box-shadow: 0px 4px 10px rgba(76, 175, 80, 0.2); /* Soft shadow */
  transition: all 0.3s ease;
}
.button-row .login-button:hover {
  background-color: #45a049; /* Darker green on hover */
  box-shadow: 0px 6px 12px rgba(76, 175, 80, 0.3); /* More pronounced shadow on hover */
}
.button-row .register-button {
  background-color: #ffffff;
  color: #4caf50; /* Same primary green for the text */
  border-color: #4caf50;
  font-weight: 100;
  font-size: 0.5rem;
  transition: all 0.3s ease;
}
.button-row .register-button:hover {
  background-color: #e8f5e9; /* Light green background on hover */
  color: #45a049; /* Slightly darker green for text on hover */
  color: #ffffff;
}
.remember-me-text {
  font-size: 0.75rem; /* Match size with the link */
  color: #6c757d; /* Match the text color with the link */
  line-height: 1.5; /* Align with the checkbox */
}
</style>
