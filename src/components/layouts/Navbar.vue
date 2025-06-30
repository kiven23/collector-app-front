<template>
  <nav>
    <v-navigation-drawer
      :value="drawer"
      app
      clipped
      class="sales-edge-navbar"
      dark
    >
      <v-list-item class="user-profile-section py-4">
        <v-list-item-avatar size="64">
          <v-img src="https://via.placeholder.com/150/FFFFFF/000000?text=JD"></v-img> </v-list-item-avatar>
        <v-list-item-content>
          <v-list-item-title class="user-name text-h6 font-weight-bold">
            {{ currentUser.first_name }}&nbsp;{{ currentUser.last_name }}
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
.sales-edge-navbar {
  background: linear-gradient(to bottom, #1a237e, #283593) !important; /* Darker, more professional gradient */
  box-shadow: 3px 0 10px rgba(0, 0, 0, 0.2); /* Subtle shadow for depth */
}

/* User Profile Section */
.user-profile-section {
  background-color: rgba(255, 255, 255, 0.08); /* Slightly lighter background for the profile */
  margin-bottom: 0; /* Remove default margin */
}

.user-profile-section .v-list-item__avatar {
  border: 2px solid rgba(255, 255, 255, 0.5); /* Light border around avatar */
  margin-right: 15px; /* Space between avatar and text */
}

.user-name {
  color: #ffffff; /* Pure white for the user's name */
  font-size: 1.15rem !important; /* Slightly larger name */
  letter-spacing: 0.5px;
}

.user-position {
  color: #bbdefb; /* Muted light blue for position */
  font-size: 0.85rem !important;
}

.nav-divider {
  border-color: rgba(255, 255, 255, 0.2) !important; /* Lighter divider for contrast */
}

/* General Link Styling */
.navigation-links .v-list-item {
  margin: 5px 10px; /* Spacing between list items */
  border-radius: 8px; /* Rounded corners for list items */
  transition: all 0.2s ease-in-out; /* Smooth transitions */
}

.navigation-links .v-list-item:hover {
  background-color: rgba(255, 255, 255, 0.15) !important; /* Lighter hover effect */
  transform: translateX(3px); /* Slight slide on hover */
}

.navigation-links .v-list-item .v-list-item__title,
.navigation-links .v-list-item .v-icon {
  color: #e3f2fd; /* Light text/icon color for all items */
}

/* Active Link Styling */
.active-nav-link {
  background: linear-gradient(to right, #007bff, #0056b3) !important; /* Matches app bar gradient */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3); /* Subtle shadow for active state */
  border-left: 5px solid #ffeb3b; /* Vibrant yellow border for active item */
  padding-left: 11px !important; /* Adjust padding due to border */
}

.active-nav-link .v-list-item__title,
.active-nav-link .v-icon {
  color: #ffffff !important; /* Pure white for active text/icon */
  font-weight: 600 !important; /* Bolder text for active item */
}

/* Sub-group and Nested Link Adjustments */
.v-list-group.parent-link .v-list-item__content {
  padding-left: 0; /* Remove default padding from parent group title */
}

.sub-parent-link .v-list-item__content {
  padding-left: 16px !important; /* Indent for sub-parent links */
}

.nested-link {
  padding-left: 48px !important; /* Increased indentation for nested links */
}

.child-link {
  padding-left: 32px !important; /* Indentation for direct child links */
}
</style>


<!-- <template>
  <nav>
    <v-navigation-drawer :value="drawer" dark app clipped style="background-color: #2596be; ">
      <v-list dense>
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title class="title">
              {{ currentUser.first_name }}&nbsp;{{ currentUser.last_name }}
            </v-list-item-title>
            <v-list-item-subtitle>
              {{
                currentUser.employment
                  ? currentUser.employment.position
                    ? currentUser.employment.position.name
                    : "None"
                  : "None"
              }}
            </v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>

        <v-divider></v-divider>

        <span v-for="(link, index) in permissions" :key="index">
          <v-list-group
            @click="toRoute(link.route)"
            :append-icon="link.subLinks ? 'keyboard_arrow_down' : ''"
            :prepend-icon="link.icon"
          >
            <template v-slot:activator>
              <v-list-item-title
                v-text="link.text"
              ></v-list-item-title>
            </template>

            <span
              v-for="(subLink, i2) in link.subLinks"
              :key="i2"
              :to="subLink.route"
            >
              <v-list-item v-if="!subLink.links">
                <v-list-item-action></v-list-item-action>
                <v-list-item-title class="text-wrap" v-text="subLink.text"></v-list-item-title>
                <v-list-item-action>
                  <v-icon v-text="subLink.icon"></v-icon>
                </v-list-item-action>
              </v-list-item>
            </span>

            <span v-for="(subLink, i3) in link.subLinks" :key="i3">
              <v-list-group v-if="subLink.links" no-action sub-group>
                <template v-slot:activator>
                  <v-list-item-content>
                    <v-list-item-title class="text-wrap"
                      v-text="subLink.text"
                    ></v-list-item-title>
                  </v-list-item-content>
                </template>

                <v-list-item
                  v-for="(_subLink, i) in subLink.links"
                  :key="i"
                  router
                  :to="_subLink.route"
                  link
                >
 
                   
                 
                  <v-list-item-title  class="text-wrap" v-text="_subLink.text" ></v-list-item-title>
                  <v-list-item-action>
                    <v-icon v-text="_subLink.icon"></v-icon>
                  </v-list-item-action>  
                </v-list-item>
              </v-list-group>
            </span>
          </v-list-group>
        </span>
      </v-list>
     
    </v-navigation-drawer>
  </nav>
</template>

<script>
import { mapGetters } from "vuex";
export default {
  data() {
    return {
      
    };
  },

  computed: {
    ...mapGetters({
      permissions: "userPermissions/getPermissions",
    }),
    drawer() {
      return this.$store.state.drawer;
    },
    currentUser() {
      return this.$store.getters.currentUser;
    },
    getCurrentRoutePath(){
      return this.$route.path;
    }
  },
  created() {
    this.$store.dispatch("userPermissions/fetchPermissions");
 
  },
  methods: {
    toRoute(_path) {
      if(!_path || this.getCurrentRoutePath == _path) {
        return
      }
      this.$router.push(_path);
    },
 
  },

};



</script> -->


