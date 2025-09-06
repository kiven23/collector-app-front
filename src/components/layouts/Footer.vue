<template>
  <v-footer app class="app-footer-unique py-0">
    <v-container fluid class="pa-0">
      <v-row no-gutters class="footer-row">
        <v-col cols="12" md="6" class="footer-left pa-6 d-flex align-center justify-center justify-md-start">
          <div class="text-center text-md-left">
            <h3 class="white--text text-h6 font-weight-bold mb-2">
              Customer Collection Tracker
            </h3>
            <p class="grey--text text--lighten-2 text-caption mb-0">
              &copy; {{ new Date().getFullYear() }} All rights reserved.
            </p>
          </div>
        </v-col>

        <v-col cols="12" md="6" class="footer-right pa-6 d-flex align-center justify-center justify-md-end">
          <div class="text-center text-md-right">
            <p class="white--text text-subtitle-2 font-weight-medium mb-1">
              Urdaneta City, Ilocos Region, Philippines
            </p>
            <p class="grey--text text--lighten-2 text-caption mb-0">
              <v-icon small color="primary" class="mr-1">mdi-phone</v-icon> +63 9XX XXX XXXX |
              <v-icon small color="primary" class="ml-2 mr-1">mdi-email</v-icon> info@yourcompany.com
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

---

### **Styling for Unique Footer**

```css
<style scoped>
.app-footer-unique {
  background-color: #121212 !important; /* Main dark background */
  box-shadow: 0 -4px 15px rgba(0, 0, 0, 0.5); /* Stronger shadow for depth */
}

.footer-row {
  min-height: 80px; /* Set a minimum height for the footer */
}

.footer-left {
  background-color: #1a1a1a; /* Slightly lighter dark for the left section */
  border-right: 1px solid #2c2c2c; /* Subtle divider */
}

.footer-right {
  background-color: #1a1a1a; /* Consistent with left section */
}

/* Responsive adjustments for smaller screens */
@media (max-width: 959px) {
  .footer-left, .footer-right {
    border-right: none; /* Remove border on mobile */
    border-bottom: 1px solid #2c2c2c; /* Add bottom border between sections on mobile */
  }
  .footer-right {
    border-bottom: none; /* No bottom border on the last section */
  }
}

.white--text {
  color: #ffffff !important;
}

.grey--text.text--lighten-2 {
  color: #b0b0b0 !important;
}

.primary--text {
  color: #673ab7 !important; /* Our deep purple accent */
}
</style>