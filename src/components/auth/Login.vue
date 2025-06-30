<template>
  <v-container fluid fill-height class="login-background">
    <v-layout align-center justify-center>
      <v-flex xs12 sm8 md5>
        <v-card class="elevation-12 pa-4 login-card">
          <v-toolbar flat color="transparent" class="login-toolbar">
            <v-img
              src="/logo3.png"
              max-width="120"
              max-height="120"
              class="mx-auto custom-logo"
            ></v-img>
          </v-toolbar>
          <v-card-title class="justify-center primary--text text-h5 font-weight-bold mt-7">
            Reports System
   
          </v-card-title>

          <v-card-text>
            <v-form class="login-form">
              <v-text-field
                class="custom-input mb-6"
                autofocus
                prepend-inner-icon="mdi-account-circle"
                v-model="form.email"
                label="Username"
                type="text"
                outlined
                rounded
                dense
                @keydown.enter="authenticate"
              ></v-text-field>

              <v-text-field
                class="custom-input"
                prepend-inner-icon="mdi-lock"
                v-model="form.password"
                label="Password"
                type="password"
                outlined
                rounded
                dense
                @keydown.enter="authenticate"
              ></v-text-field>
            </v-form>
          </v-card-text>

          <v-card-actions class="px-4 pb-4">
            <v-spacer></v-spacer>
            <v-btn
              color="primary"
              dark
              large
              rounded
              @click="authenticate"
              :loading="loadingStatus"
              class="login-button"
            >
              Login
              <v-icon right>mdi-login</v-icon>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import { login } from "../../helpers/auth";

export default {
  name: "login",
  data() {
    return {
      form: {
        email: "",
        password: "",
      },
    };
  },
  methods: {
    authenticate() {
      this.$store.dispatch("login");

      login(this.$data.form)
        .then((response) => {
          this.$store.commit("loginSuccess", response);
          this.$router.push({ path: "/" });
        })
        .catch((error) => {
          let payload = [
            { status: true, message: error, timeout: 3000 },
          ];
          this.$store.commit("SNACKBAR_STATUS", payload, { root: true });
        });
    },
  },
  computed: {
    loadingStatus() {
      return this.$store.state.loading;
    },
  },
  created() {
    this.$store.commit("LOADING_STATUS", false, { root: true });
  },
};
</script>

<style scoped>
.login-background {
  background: linear-gradient(to right, #4facfe, #00f2fe); /* Softer, modern gradient */
  background-size: cover;
  display: flex; /* Use flexbox for centering */
  align-items: center; /* Center vertically */
  justify-content: center; /* Center horizontally */
}

.login-card {
  background-color: rgba(255, 255, 255, 0.95); /* Slightly less transparent white */
  border-radius: 15px; /* More pronounced rounded corners */
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2); /* Stronger shadow for depth */
  overflow: hidden; /* Ensures content stays within rounded corners */
}

.login-toolbar {
  background-color: transparent !important; /* Make toolbar transparent */
  padding-top: 20px; /* Add some space above the logo */
}

.custom-logo {
  border-radius: 50%; /* Make the logo round */
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1); /* Subtle shadow for the logo */
  border: 3px solid #1976D2; /* Add a border to the logo */
}

.logo-underline {
  display: block; /* Change to block to take full width of parent */
  width: 50px; /* Adjust width as needed */
  height: 4px; /* Thicker underline */
  background-color: #1976D2; /* Primary color */
  margin: 8px auto 0 auto; /* Center the underline and add margin */
  border-radius: 2px;
}

.login-form {
  padding: 20px; /* More padding */
  background: none; /* Remove background from form to let card background show */
  box-shadow: none; /* Remove shadow from form */
}

.custom-input .v-label {
  font-weight: 500; /* Slightly less bold for subtlety */
  color: #3f51b5 !important; /* Deeper blue for labels */
  font-size: 1rem; /* Standard font size */
}

.custom-input input {
  font-size: 1.1rem; /* Slightly larger input text */
  font-weight: 500;
  color: #2c3e50; /* Darker, more professional text color */
  letter-spacing: 0.2px;
}

.custom-input .v-input__control {
  border-radius: 10px; /* More rounded input fields */
  background-color: #f8f9fa; /* Off-white background for inputs */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05); /* Softer shadow */
  padding-left: 10px; /* More padding inside input */
}

.custom-input .v-icon {
  color: #007bff; /* A more vibrant blue for icons */
}

.login-button {
  letter-spacing: 1px;
  font-weight: bold;
  transition: all 0.3s ease-in-out; /* Smooth transition for hover effects */
}

.login-button:hover {
  transform: translateY(-2px); /* Slight lift on hover */
  box-shadow: 0 5px 15px rgba(0, 123, 255, 0.4); /* Blue shadow on hover */
}
</style>