<template>
  <v-snackbar
    :value="snackbar"
    :timeout="snackbarTimeout"
    bottom
    class="custom-snackbar"
  >
    <div class="snackbar-content">
      <v-icon class="mr-3">{{ snackbarIcon }}</v-icon>
      {{ snackbarText }}
    </div>
    
    <template v-slot:action="{ attrs }">
      <v-btn
        color="primary"
        text
        v-bind="attrs"
        @click="closeSnackbar"
      >
        Dismiss
      </v-btn>
    </template>
  </v-snackbar>
</template>

<script>
export default {
  computed: {
    snackbar() {
      return this.$store.state.snackbar;
    },
    snackbarText() {
      return this.$store.state.snackbarText;
    },
    snackbarTimeout() {
      return this.$store.state.snackbarTimeout;
    },
    snackbarIcon() {
      // You can add logic here to return different icons based on message type (e.g., success, error)
      return 'mdi-check-circle-outline'; // Default icon
    }
  },
  methods: {
    closeSnackbar() {
      // It's better to use a mutation to update state
      this.$store.commit('setSnackbar', false);
    }
  }
};
</script>

<style scoped>
.custom-snackbar.v-snack--active .v-snack__wrapper {
  background-color: #1a1a1a !important;
  color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
}

.custom-snackbar .snackbar-content {
  display: flex;
  align-items: center;
}

.v-btn--text.v-btn--active:before,
.v-btn--text.v-btn:focus:before,
.v-btn--text.v-btn:hover:before {
  background-color: transparent !important;
}
</style>