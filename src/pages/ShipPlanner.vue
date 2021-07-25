<template>
  <div>
    <h5>Hallo, ShipPlanner!</h5>
    <span>c====8</span>
    <div>
      <input type="number" min="1" max="100" step="1" v-model="RectSize">
      <input type="number" min="1" max="100" step="1" v-model="Rows">
      <input type="number" min="1" max="100" step="1" v-model="Colums">
      <input type="number" min="1" max="100" step="1" v-model="GridGap">
    </div>
    <div class="svg-area">
      <ship-svg :grid="grid"></ship-svg>
    </div>
  </div>
</template>

<script lang="ts">

import ShipSvg from "@/components/ShipSvg.vue";
import {App, defineComponent, PropType, toRefs, ref} from "vue";

const ShipPlanner = defineComponent({
  name: 'ShipPlanner',
  components: {ShipSvg},
  props: {
  },
  setup(props){
    const RectSize = ref(8);
    const Rows = ref(5);
    const Colums = ref(22);
    const GridGap = ref(2);
    const gridData = ref([
      [0,0,2,0,0,0,0,2,0,0,0,0,2,0,0,2,0,0],
      [0,0,2,0,0,0,0,2,0,0,0,0,2,0,0,2,0,0],
      [0,0,3,0,0,0,0,3,0,0,0,0,3,0,0,3,0,0],
      [0,0,3,0,0,0,0,3,0,0,0,0,3,0,0,3,0,0],
      [0,4,4,4,0,0,4,4,4,0,0,4,4,4,0,4,4,0],
      [0,4,4,4,0,0,4,4,4,0,0,4,4,4,0,4,4,0],
      [0,5,0,5,0,0,5,0,5,0,0,5,0,5,0,0,5,0],
      [0,5,0,5,0,0,5,0,5,0,0,5,0,5,0,0,5,0],
      [7,6,0,6,7,7,6,0,6,7,7,6,0,6,7,0,6,7],
      [7,6,0,6,7,7,6,0,6,7,7,6,0,6,7,0,6,7],
      [0,5,0,5,0,0,5,0,5,0,0,5,0,5,0,0,5,0],
      [7,6,0,6,7,7,6,0,6,7,7,6,0,6,7,0,6,7],
      [7,6,0,6,7,7,6,0,6,7,7,6,0,6,7,0,6,7],
      [0,5,0,5,0,0,5,0,5,0,0,5,0,5,0,0,5,0],
      [7,6,0,6,7,7,6,0,6,7,7,6,0,6,7,0,6,7],
      [7,6,0,6,7,7,6,0,6,7,7,6,0,6,7,0,6,7],
      [7,7,0,7,7,7,7,0,7,7,7,7,0,7,7,0,7,7],
      [7,7,0,7,7,7,7,0,7,7,7,7,0,7,7,0,7,2],
    ])
    return {
      RectSize: RectSize,
      Rows: Rows,
      Colums: Colums,
      GridGap: GridGap,
      grid: gridData,

    }
  }
});

export default ShipPlanner;

</script>

<style scoped>
.svg-area {
  padding: 3rem;
  background-color: rgba(33,33,33,0.3);
  border: 4px groove rgba(33,33,33,0.8);
}
</style>
