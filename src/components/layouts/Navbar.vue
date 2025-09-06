<template>
  <nav>
    <v-navigation-drawer
      :value="drawer"
      app
      clipped
      dark
      class="app-sidebar"
    >
      <v-list-item class="user-profile-section py-4">
        <v-list-item-avatar size="64">
          <v-img :src="currentUser.avatar || 'https://via.placeholder.com/150/1A1A1A/FFFFFF?text=JD'"></v-img>
        </v-list-item-avatar>
        <v-list-item-content>
          <v-list-item-title class="user-name text-body-1 font-weight-bold">
            {{ currentUser.last_name || 'User' }}
          </v-list-item-title>
          <v-list-item-subtitle class="user-position mt-1">
            {{
              currentUser.employment && currentUser.employment.position
                ? currentUser.employment.position.name
                : "Position Not Set"
            }}
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <v-divider class="my-3 nav-divider"></v-divider>

      <v-list dense nav class="navigation-links">
        <template v-for="(link, index) in permissions">
          <v-list-group
            v-if="link.subLinks && link.subLinks.length > 0"
            :key="`group-${index}`"
            :prepend-icon="link.icon"
            no-action
            color="white"
            class="parent-link"
          >
            <template v-slot:activator>
              <v-list-item-title v-text="link.text"></v-list-item-title>
            </template>

            <template v-for="(subLink, i2) in link.subLinks">
              <v-list-group
                v-if="subLink.links && subLink.links.length > 0"
                :key="`sub-group-${i2}`"
                sub-group
                no-action
                class="sub-parent-link"
                color="white"
              >
                <template v-slot:activator>
                  <v-list-item-content>
                    <v-list-item-title v-text="subLink.text"></v-list-item-title>
                  </v-list-item-content>
                </template>
                <v-list-item
                  v-for="(_subLink, i3) in subLink.links"
                  :key="`nested-link-${i3}`"
                  :to="_subLink.route"
                  link
                  active-class="active-nav-link"
                  class="nested-link"
                >
                  <v-list-item-title v-text="_subLink.text"></v-list-item-title>
                  <v-list-item-icon v-if="_subLink.icon">
                    <v-icon>{{ _subLink.icon }}</v-icon>
                  </v-list-item-icon>
                </v-list-item>
              </v-list-group>

              <v-list-item
                v-else
                :key="`single-sublink-${i2}`"
                :to="subLink.route"
                link
                active-class="active-nav-link"
                class="child-link"
              >
                <v-list-item-title v-text="subLink.text"></v-list-item-title>
                <v-list-item-icon v-if="subLink.icon">
                  <v-icon>{{ subLink.icon }}</v-icon>
                </v-list-item-icon>
              </v-list-item>
            </template>
          </v-list-group>

          <v-list-item
            v-else
            :key="`single-link-${index}`"
            :to="link.route"
            link
            active-class="active-nav-link"
            class="top-level-link"
            @click="toRoute(link.route)"
          >
            <v-list-item-icon>
              <v-icon>{{ link.icon }}</v-icon>
            </v-list-item-icon>
            <v-list-item-title v-text="link.text"></v-list-item-title>
          </v-list-item>
        </template>
      </v-list>
    </v-navigation-drawer>
  </nav>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  data() {
    return {
      // No local data needed here for permissions, as it's from Vuex
    };
  },
  computed: {
    ...mapGetters({
      permissions: "userPermissions/getPermissions",
    }),
    drawer: {
      get() {
        return this.$store.state.drawer;
      },
      set(value) {
        this.$store.commit('setDrawer', value); // Assuming you have a mutation to update drawer state
      }
    },
    currentUser() {
      return this.$store.getters.currentUser;
    },
    getCurrentRoutePath() {
      return this.$route.path;
    }
  },
  created() {
    this.$store.dispatch("userPermissions/fetchPermissions");
  },
  methods: {
    toRoute(_path) {
      if (!_path || this.getCurrentRoutePath === _path) {
        return;
      }
      this.$router.push(_path);
    },
  },
};
</script>

---

### **Styling for the Navigation Bar**

```css
<style scoped>
/* Main Sidebar Styling */
.app-sidebar {
  background-color: #1a1a1a !important; /* Dark background */
  box-shadow: 3px 0 10px rgba(0, 0, 0, 0.4); /* Deeper shadow for depth */
}

/* User Profile Section */
.user-profile-section {
  background-color: #121212 !important; /* Slightly darker background for the profile section */
  margin-bottom: 0;
}

.user-profile-section .v-list-item__avatar {
  border: 2px solid #673ab7; /* Deep purple border around avatar */
  margin-right: 15px;
  box-shadow: 0 0 8px rgba(103, 58, 183, 0.5); /* Glowing effect */
}

.user-name {
  color: #ffffff;
  font-size: 1.1rem !important;
  letter-spacing: 0.5px;
}

.user-position {
  color: #b0b0b0;
  font-size: 0.85rem !important;
}

.nav-divider {
  border-color: #2c2c2c !important; /* Lighter divider for contrast */
}

/* General Link Styling */
.navigation-links .v-list-item {
  margin: 5px 10px;
  border-radius: 8px;
  transition: all 0.2s ease-in-out;
}

.navigation-links .v-list-item:hover {
  background-color: #2c2c2c !important; /* Lighter hover effect */
  transform: translateX(3px);
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.navigation-links .v-list-item .v-list-item__title,
.navigation-links .v-list-item .v-icon {
  color: #e0e0e0; /* Light gray text/icon color */
}

/* Active Link Styling */
.active-nav-link {
  background-color: #383838 !important; /* A slightly lighter background for active link */
  border-left: 5px solid #673ab7; /* Deep purple active border */
  padding-left: 11px !important;
  box-shadow: 0 0 10px rgba(103, 58, 183, 0.5); /* Glowing effect for active state */
}

.active-nav-link .v-list-item__title,
.active-nav-link .v-icon {
  color: #ffffff !important;
  font-weight: 600 !important;
}

/* Sub-group and Nested Link Adjustments */
.nested-link {
  padding-left: 48px !important;
}

.child-link {
  padding-left: 32px !important;
}
</style>