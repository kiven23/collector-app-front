<template>
  <v-footer app dark class="app-footer-unique py-0">
    <v-container fluid class="pa-0">
      <v-row no-gutters class="footer-row">
        <!-- Left Column -->
        <v-col cols="12" md="6" class="footer-left pa-6 d-flex flex-column align-center align-md-start justify-center">
          <div class="text-center text-md-left">
            <h3 class="white--text text-h6 font-weight-bold mb-2">
              Customer Collection Tracker
            </h3>
            <p class="grey--text text--lighten-2 text-caption mb-0">
              &copy; {{ new Date().getFullYear() }} All rights reserved.
            </p>
          </div>
        </v-col>

        <!-- Right Column -->
        <v-col cols="12" md="6" class="footer-right pa-6 d-flex flex-column align-center align-md-end justify-center">
          <div class="text-center text-md-right">
            <p class="white--text text-subtitle-2 font-weight-medium mb-1">
              Urdaneta City, Ilocos Region, Philippines
            </p>
            <p class="grey--text text--lighten-2 text-caption mb-0">
              <v-icon small color="primary" class="mr-1">mdi-phone</v-icon> +63 9237473678 |
              <v-icon small color="primary" class="ml-2 mr-1">mdi-email</v-icon> webdev@addessa.com
            </p>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </v-footer>
</template>

<script>
export default {
  // No script changes needed for this footer
};
</script>