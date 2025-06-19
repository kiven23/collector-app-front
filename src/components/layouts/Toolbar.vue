<template>
  <nav>
    <v-app-bar app clipped-left  style="background-color: white; border-radius: 10px;">
      <!-- <v-app-bar-nav-icon @click.stop="drawer"></v-app-bar-nav-icon> -->
      <v-btn icon @click.stop="drawer">
        <v-icon>{{ toggleDrawer ? "mdi-menu" : "mdi-menu-open" }}</v-icon>
      </v-btn>
      <v-img
        src="/addessa_logo.png"
        max-width="40"
        max-height="40"
        style="margin: 5px"
      ></v-img>
      <v-toolbar-title class="d-flex align-center">
  <svg xmlns="http://www.w3.org/2000/svg" 
       height="24" viewBox="0 0 24 24" 
       fill="currentColor" class="mr-2">
    <path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04a1.003 1.003 0 0 0 0-1.42l-2.34-2.34a1.003 1.003 0 0 0-1.42 0l-1.83 1.83 3.75 3.75 1.84-1.82z"/>
  </svg>
  <strong>SalesEdge</strong>
</v-toolbar-title>


      <v-spacer></v-spacer>
      <v-spacer></v-spacer>
      <v-spacer></v-spacer>
    
 
                  <!-- <v-select
                      style="margin-top: 15px; margin-right: 15px; width: 10px"
                      :items="connections.databases"
                      label="Select Database"
                      v-model="connections.connection"
                      item-value="id"
                      item-text="dbname"
                      dense
                      :loading="loadingStatus"
                      @change ="change()"
                      :hidden="true"
                 ></v-select> -->

      
    
      <v-tooltip-title >
        <strong style="text-align: center">{{currentUser.branch == null? '': currentUser.branch.name}}</strong>
      </v-tooltip-title>

      <v-tooltip bottom>
       
        <span>{{ isDark ? "Go Dark" : "Go Light" }}</span>
      </v-tooltip>
     
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn icon v-on="on" @click="logout">
            <v-icon>logout</v-icon>
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
    console.log(this.$store.getters.currentUser);
    this.$store.dispatch("fetchDatabase");
   
  },

  computed: {
    connections(){
     return  this.$store.state.connections;
    },
    currentUser() {
      return this.$store.getters.currentUser;
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
