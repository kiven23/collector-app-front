<template>
  <nav>
    <v-app-bar
      app
      clipped-left
      class="sales-edge-app-bar"
      height="64"
      flat
    >
      <v-app-bar-nav-icon
        @click.stop="drawer"
        color="white"
        aria-label="Toggle Navigation"
      >
        <v-icon>{{ toggleDrawer ? "mdi-menu-open" : "mdi-menu" }}</v-icon>
      </v-app-bar-nav-icon>

      <v-img
        src="/logo3.png"
        max-width="40"
        max-height="40"
        contain
        class="ml-2 mr-3 app-logo"
      ></v-img>

      <v-toolbar-title class="app-title">
        <span class="font-weight-light">SALES</span><strong class="font-weight-bold">EDGE</strong>
      </v-toolbar-title>

      <v-spacer></v-spacer>

      <v-chip class="mr-4 branch-chip" color="white" text-color="primary" label>
        <v-icon left small>mdi-map-marker</v-icon>
        <strong v-if="currentUser.branch && currentUser.branch.name">{{ currentUser.branch.name }}</strong>
        <span v-else>No Branch Selected</span>
      </v-chip>

      <!-- <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn icon v-on="on" @click="goDark" color="white" aria-label="Toggle Dark Mode">
            <v-icon>{{ isDark ? 'mdi-brightness-4' : 'mdi-brightness-7' }}</v-icon>
          </v-btn>
        </template>
        <span>{{ isDark ? "Switch to Light Mode" : "Switch to Dark Mode" }}</span>
      </v-tooltip> -->

      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn icon v-on="on" @click="logout" color="white" aria-label="Logout">
            <v-icon>mdi-logout</v-icon>
          </v-btn>
        </template>
        <span>Logout</span>
      </v-tooltip>
    </v-app-bar>
  </nav>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  methods: {
    change(){
      const data = {'id': this.connections.connection}
      this.$store.dispatch("updateDB", data);
    },
    drawer() {
      // Toggle the drawer state
      this.$store.state.drawer = !this.$store.state.drawer;
    },
    logout() {
      this.$store.commit("logout");
      this.$router.push("/login");
    },
    goDark() {
      // Toggle dark mode state
      this.$store.state.goDark = !this.$store.state.goDark;
    },
  },
  created() {
    // console.log(this.$store.getters.currentUser); // Keep for debugging if needed
    this.$store.dispatch("fetchDatabase");
  },
  computed: {
    ...mapGetters(['currentUser']), // More concise way to map getters
    connections(){
      return  this.$store.state.connections;
    },
    loadingStatus() {
      return this.$store.state.loading;
    },
    isDark() {
      return this.$store.state.goDark;
    },
    toggleDrawer() {
      return this.$store.state.drawer;
    },
  },
};
</script>

---

### **Styling for the App Bar**

```css
<style scoped>
.sales-edge-app-bar {
  background: linear-gradient(to right, #1a237e, #283593) !important; /* Professional blue gradient */
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2); /* Enhanced shadow for depth */
  border-radius: 0 !important; /* Remove border-radius for full-width sleekness */
}

/* Specific styling for the app logo */
.app-logo {
  border-radius: 5px; /* Slightly rounded corners for the logo */
  object-fit: contain; /* Ensures the image fits without cropping */
}

/* Styling for the SalesEdge title */
.app-title {
  color: white; /* White text for contrast */
  font-size: 1.6rem; /* Larger font size for prominence */
  letter-spacing: 1px; /* Slightly increased letter spacing */
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2); /* Subtle text shadow */
  margin-left: 5px; /* Space from the logo */
}

.app-title strong {
  font-weight: 900; /* Extra bold for "EDGE" */
}

/* Branch Name Chip Styling */
.branch-chip {
  font-weight: 600; /* Bold text for the branch name */
  letter-spacing: 0.5px;
  /* Add more styling if needed, e.g., different border or shadow */
}

/* Icon button hover effects */
.v-btn--icon:hover {
  background-color: rgba(255, 255, 255, 0.2) !important; /* Lighter hover background */
  border-radius: 50%; /* Ensure it stays round on hover */
}

/* Adjustments for smaller screens */
@media (max-width: 600px) {
  .sales-edge-app-bar {
    height: 56px !important; /* Standard height for mobile app bars */
  }

  .app-title {
    font-size: 1.3rem; /* Smaller title on mobile */
  }

  .branch-chip {
    display: none; /* Hide branch chip on very small screens if space is limited */
  }
}
</style>