<template>
  <div class="navigation">
    <router-link class="navigation-link" v-for="(link, index) in links" :key="index" :to="link.path" >{{ link.name }}</router-link>
  </div>
</template>
<script lang="ts">
import {Options, Vue} from "vue-class-component";
import {defineComponent, h, PropType, ref, reactive} from "vue";
import RouterLink from "vue-router";
import {VNode} from "@vue/runtime-core";
import {MainMenuLink} from "@/MainMenuLink"

const MainMenuLinksPanel = defineComponent({
  components: { RouterLink },
  props: {
    links: {
      type: Object as PropType<MainMenuLink[]>,
      required: true
    }
  },
  setup(props) {
      const links = reactive(props.links)

    const addLink=(link:MainMenuLink) => h('router-link', {path: link.path, name: link.name});

      // return the render function
      return h(
              /** @ts-ignore */
              "div",
              {

  //              onClick: increment
              },
              links.map(x=>addLink)
          )
  }
})

export default MainMenuLinksPanel;
</script>

<style lang="scss" scoped>
.navigation {
  .navigation-link {padding-left: 0.9rem;margin-left: 0.1rem; padding-right: 0.9rem;margin-right: 0.1rem; border-left: 2px solid red; border-right: 2px solid red;}
  .navigation-link:first-child {padding-left: .5rem; padding-right: 0.9rem;margin-right: 0.1rem;}
  .navigation-link:last-child {padding-left: 0.9rem;margin-left: 0.1rem; padding-right: 0.5rem;}
}
</style>
