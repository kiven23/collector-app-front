<template>
  <nav>
    <v-app-bar
      app
      clipped-left
      class="app-bar"
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
        src="/collection.png"
        max-width="40"
        max-height="40"
        contain
        class="ml-2 mr-3 app-logo"
      ></v-img>

      <v-toolbar-title class="app-title">
        <span class="font-weight-light">CUSTOMER</span><strong class="font-weight-bold">TRACKER</strong>
      </v-toolbar-title>

      <v-spacer></v-spacer>

      <v-chip class="mr-4 branch-chip" color="#1a1a1a" text-color="primary" label>
        <v-icon left small>mdi-map-marker</v-icon>
        <strong v-if="currentUser.branch && currentUser.branch.name">{{ currentUser.branch.name }}</strong>
        <span v-else>No Branch Selected</span>
      </v-chip>

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
      this.$store.state.drawer = !this.$store.state.drawer;
    },
    logout() {
      this.$store.commit("logout");
      this.$router.push("/login");
    },
    goDark() {
      this.$store.state.goDark = !this.$store.state.goDark;
    },
  },
  created() {
    this.$store.dispatch("fetchDatabase");
  },
  computed: {
    ...mapGetters(['currentUser']),
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
.app-bar {
  background: linear-gradient(to right, #1a1a1a, #2c2c2c) !important;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
  border-radius: 0 !important;
}

.app-logo {
  border-radius: 5px;
  object-fit: contain;
}

.app-title {
  color: white;
  font-size: 1.6rem;
  letter-spacing: 1px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.4);
  margin-left: 5px;
}

.app-title strong {
  font-weight: 900;
  color: #673ab7; /* Deep purple accent */
}

.branch-chip {
  font-weight: 600;
  letter-spacing: 0.5px;
  color: #673ab7 !important;
}

.v-btn--icon:hover {
  background-color: rgba(255, 255, 255, 0.1) !important;
  border-radius: 50%;
}
</style>