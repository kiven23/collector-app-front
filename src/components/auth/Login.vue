<template>
  <v-container fluid fill-height class="login-background">
    <div class="background-glow"></div>
    <v-layout align-center justify-center>
      <v-flex xs12 sm8 md5>
        <v-card class="dark-card pa-10 rounded-xl elevation-20 animate__animated animate__zoomIn">
          
          <div class="text-center mb-6">
            <v-avatar size="100" class="logo-glow mb-4">
              <v-img src="/collection.png"></v-img>
            </v-avatar>
            <h1 class="text-h4 font-weight-bold primary--text mb-2">
              Customer Collection System
            </h1>
            <p class="white--text text-subtitle-1 font-weight-light">
              Log in to your account
            </p>
          </div>

          <v-form>
            <v-text-field
              class="mb-4 dark-input"
              prepend-inner-icon="mdi-account"
              v-model="form.email"
              label="Username"
              type="text"
              solo
              flat
              rounded
              dark
            ></v-text-field>

            <v-text-field
              class="dark-input"
              prepend-inner-icon="mdi-lock"
              v-model="form.password"
              label="Password"
              type="password"
              solo
              flat
              rounded
              dark
              @keydown.enter="authenticate"
            ></v-text-field>
          </v-form>

          <v-btn
            block
            color="primary"
            x-large
            rounded
            @click="authenticate"
            :loading="loadingStatus"
            class="login-button mt-6 font-weight-bold text-capitalize"
          >
            Sign In
            <v-icon right>mdi-arrow-right</v-icon>
          </v-btn>

          <div class="text-center mt-6">
            <a href="#" class="primary--text text-decoration-none font-weight-medium">Forgot Password?</a>
          </div>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
// Make sure your path is correct
import { login } from "../../helpers/auth"; 

export default {
  name: "Login",
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
/* Main Dark Background */
.login-background {
  height: 100vh;
  width: 100vw;
  background-color: #121212;
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Subtle background glow effect */
.background-glow {
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle at 50% -20%, #4a2373, #121212 50%);
  opacity: 0.6;
}

/* Dark Card with soft shadows */
.dark-card {
  background-color: #1a1a1a !important;
  border: 1px solid #333;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.5), 0 0 40px #673ab7; /* Glow effect */
}

/* Dark Inputs with soft glow */
.dark-input.v-text-field--solo .v-input__control .v-input__slot {
  background-color: #2c2c2c !important;
  box-shadow: none !important;
}

.dark-input.v-input--is-focused .v-input__control .v-input__slot {
  box-shadow: 0 0 5px #673ab7 !important;
}

.dark-input >>> .v-icon,
.dark-input >>> .v-label {
  color: #a3a3a3 !important;
}

.dark-input.v-input--is-focused .v-icon {
  color: #673ab7 !important;
}

.dark-input >>> input {
  color: white !important;
}

/* Login Button with glow */
.login-button {
  transition: all 0.3s ease-in-out;
  background-color: #673ab7 !important; /* A striking purple */
  color: white !important;
  box-shadow: 0 4px 15px rgba(103, 58, 183, 0.4);
}

.login-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(103, 58, 183, 0.6);
}

/* Other Custom Styles */
.primary--text {
  color: #673ab7 !important;
}

.logo-glow {
  box-shadow: 0 0 10px #673ab7, 0 0 20px rgba(103, 58, 183, 0.4);
}
</style>