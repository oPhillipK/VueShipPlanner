<template>
  <div id="nav">
    <main-menu-links-panel :links="menuLinks" />
  </div>
  <router-view/>
</template>

<script lang="ts">
import {Options, Vue} from 'vue-class-component';
import {RouteRecord} from "vue-router";
import {MainMenuLink} from "@/MainMenuLink";
import MainMenuLinksPanel from "@/components/MainMenuLinksPanel.vue";

@Options({
  components: {MainMenuLinksPanel},
  props: {
    msg: String
  }
})
export default class App extends Vue {
  menuLinks:Array<MainMenuLink> = [];

  created() {
    let links:MainMenuLink[] = this.$router.getRoutes().map((route:RouteRecord):MainMenuLink => {return {path: route.path, name: route.name?.toString() || ""}})
    console.log("links", links);
    this.menuLinks = links;
  }
}

</script>
<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
