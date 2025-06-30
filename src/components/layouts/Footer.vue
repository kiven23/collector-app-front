<!-- <template>
  <footer>
    <v-footer app style="background-color: #2596be; ">
      <v-card class="flex" flat >
        <v-card-actions class="justify-center">
          &copy;&nbsp;{{ new Date().getFullYear() }} —
          <strong>&nbsp;SalesEdge Addessa Corporation</strong>
        </v-card-actions>
      </v-card>
    </v-footer>
  </footer>
</template>

<script>
export default {
  data() {
    return {
      icons: [
        "fab fa-facebook",
        "fab fa-twitter",
        "fab fa-google-plus",
        "fab fa-linkedin",
        "fab fa-instagram"
      ]
    };
  }
};
</script> -->
<template>
  <v-footer app class="sales-edge-footer py-6">
    <v-container>
      <v-row align="center" justify="space-between">
        <v-col cols="12" md="6" class="text-center text-md-left footer-text">
          &copy; {{ new Date().getFullYear() }} —
          <strong class="font-weight-bold">SalesEdge Addessa Corporation</strong>
          <span class="d-block d-md-inline mt-1 mt-md-0"> All rights reserved.</span>
        </v-col>

        <!-- <v-col cols="12" md="6" class="text-center text-md-right">
          <v-btn
            v-for="icon in icons"
            :key="icon"
            icon
            size="large"
            variant="flat"
            class="mx-2 social-icon-button"
            color="white"
            :href="getIconLink(icon)"
            target="_blank"
            rel="noopener noreferrer"
          >
            <v-icon :icon="icon" size="24"></v-icon>
          </v-btn>
        </v-col> -->
      </v-row>
    </v-container>
  </v-footer>
</template>

<script>
export default {
  data() {
    return {
      icons: [
        "fab fa-facebook-f",
        "fab fa-twitter",
        "fab fa-linkedin-in",
        "fab fa-instagram",
        "fab fa-youtube" // Added YouTube for more options
      ]
    };
  },
  methods: {
    getIconLink(icon) {
      const links = {
        "fab fa-facebook-f": "https://facebook.com/yourcompany", // Replace with actual links
        "fab fa-twitter": "https://twitter.com/yourcompany",
        "fab fa-linkedin-in": "https://linkedin.com/company/yourcompany",
        "fab fa-instagram": "https://instagram.com/yourcompany",
        "fab fa-youtube": "https://youtube.com/yourchannel"
      };
      return links[icon] || "#";
    }
  }
};
</script>

---

### **Styling Enhancements**

```css
<style scoped>
.sales-edge-footer {
  background: linear-gradient(to right, #1a237e, #283593); /* Darker, more professional gradient */
  color: #e3f2fd; /* Lighter text color for contrast */
  font-family: 'Roboto', sans-serif; /* A commonly preferred modern font */
  padding-top: 24px;
  padding-bottom: 24px;
}

.footer-text {
  font-size: 0.95rem;
  line-height: 1.5;
  color: #bbdefb; /* Slightly muted text color for less emphasis on secondary text */
}

.footer-text strong {
  color: #ffffff; /* Pure white for the main company name */
}

.social-icon-button {
  transition: all 0.3s ease; /* Smooth transition for hover effects */
  background-color: rgba(255, 255, 255, 0.1) !important; /* Subtle initial background */
  border-radius: 50%; /* Ensure perfect circle for buttons */
}

.social-icon-button:hover {
  background-color: rgba(255, 255, 255, 0.3) !important; /* Brighter background on hover */
  transform: translateY(-3px); /* Lift effect on hover */
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2); /* Soft shadow on hover */
}

.v-icon {
  color: inherit !important; /* Ensures icon color matches button's text color property */
}

/* Responsive adjustments for smaller screens */
@media (max-width: 959px) {
  .sales-edge-footer {
    padding-top: 20px;
    padding-bottom: 20px;
  }
  .footer-text {
    margin-bottom: 15px; /* Add some space between copyright and icons on small screens */
  }
  .social-icon-button {
    margin: 0 8px; /* Adjust margin for social icons on small screens */
  }
}
</style>