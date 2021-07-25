<template>
  <svg class="judas2" xself="https://www.w3.org/2000/svg" width="200" height="200">
    <rect v-for="(type, index) in rects"
          :width="rectSize"
          :height="rectSize"
          :x="positions[index].x * (rectSize + gapSize)"
          :y="positions[index].y * (rectSize + gapSize)"
          v-bind:key="index"></rect>
  </svg>

</template>
<script lang="ts">
import {defineComponent, h, PropType, ref, Ref} from "vue";
import {VNode} from "@vue/runtime-core";

type Grid = number[][]
type Position = {x: number,y:number}

const ShipSvg = defineComponent({
  name: "ShipSvg",
  props: {
    grid: Object as PropType<Grid>,
    gap: Number,
    size: Number,
    slotsX: Number,
    slotsY: Number
  },
  setup(props) {
    let grid = props.grid || [];
    let gapSize = ref(props.gap || 0);
    let rectSize = ref(props.gap || 20);
    let slotsX = ref(props.slotsX || 22);
    let slotsY = ref(props.slotsY || 22);

    console.log("SETUP", "gapSize",gapSize, "rectSize", rectSize, "slotsX", slotsX, "slotsY", slotsY, "grid", grid);

    let rects: Ref<number[]> = ref([]);
    let positions: Ref<Position[]> = ref([]);


    let x,y;
    for(y = 0; y < slotsY.value; y++) {
      if(grid.length <= y) continue;
      for(x = 0; x < slotsX.value; x++) {
        if(grid[y].length <= x) continue;
        let key = y * slotsY.value + x;
        rects.value.push(grid[y][x])
        positions.value.push({x:x,y:y});
        console.log("Push it to me baby, aha aha yeah!")
      }
    }

    const tintColor = (type:number):string => {

      switch(type) {
        case 0: return "rgba(33,33,33,1)";
        case 1: return "rgba(66,66,66,1)";
        case 2: return "rgba(99,99,99,1)";
        case 3: return "rgba(132,132,132,1)";
        case 4: return "rgba(165,165,165,1)";
        case 5: return "rgba(195,195,195,1)";
        case 6: return "rgba(225,225,225,1)";
        case 7: return "rgba(255,255,255,1)";
        default: return "rgba(200,50,200,1)";
      }
    }
    const posInRect = (x:number,y:number) => {
      return x >= 0 &&
          y >= 0 &&
          x <= slotsX.value &&
          y <= slotsY.value ;
    }
    const keyForPos = (x:number,y:number) => {
      return y * slotsX.value + x
    }

    const posForKey = (key: number) => {
      return [key % slotsX.value, Math.floor(key / slotsX.value)];
    }

    const rndPos = () => {
      return {x:Math.round(Math.random() * slotsX.value),y: Math.round(Math.random() * slotsY.value)};
    }
//    const timer = setInterval(()=>{
//      let pos = rndPos();
//      let key = keyForPos(pos.x, pos.y);
//
//      if(posInRect(pos.x,pos.y) && rects.value.length >= key) {
//        rects.value[key] = Math.floor(Math.random()*8);
//      }
//    }, 500);

    let children:VNode[] = [];

    for(let key = 0; key < rects.value.length; key++) {
      let pos = posForKey(key);
      //console.log("key", key, "result in pos", pos);
      if(posInRect(pos[0],pos[1])) {
    //    children.push(h("rect", {x: pos[0] * (rectSize.value + gapSize.value), y: pos[1] * (rectSize.value + gapSize.value),
    //      width: rectSize.value,
    //      height: rectSize.value,
    //      fill: tintColor(rects.value[key]),
    //    }))
      }
    }
    return {
      rects,
      positions,
      rectSize,
      gapSize
    }
  //  return ()=> h("svg", {class: "judas",xself: "https://www.w3.org/2000/svg", width: 200, height: 200}, children)
  }
})
export default ShipSvg;
</script>

<style scoped>
.judas {
  border: 10px groove pink;
}
</style>
