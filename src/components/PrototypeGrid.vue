<template>
  <svg :width="canvasWidth" :height="canvasHeight">
    <rect v-for="(type, rectIndex) in positions" :key="rectIndex"
          v-bind="type"
          @mouseenter="tileHover($event, rectIndex)"
          @mouseleave="tileLeave($event, rectIndex)"
    >
    </rect>
    <rect v-if="mirrorX"
          v-bind="GetMirrorXData()">
    </rect>
    <rect v-if="mirrorY"
          v-bind="GetMirrorYData()">
    </rect>
  </svg>
</template>

<script>
export default {
  name: "PrototypeGrid",
  props: {
    slotsX: {type: Number, default: 30},
    slotsY: {type: Number, default: 30},
    gap: {type: Number, default: 1},
    canvasWidth: {type: Number, default: 300},
    canvasHeight: {type: Number, default: 300},
    posData: Array,
    typeRef: Object,
    hoverPosition: {type: Number, default: -1},
    mirrorX: Boolean,
    mirrorY: Boolean
  },
  data() {return {
    isDrawing: false
  }},
  methods: {
    GetMirrorXData() {
      if(!this.mirrorX) return {}
      let tileSize = this.tileSize;

      return {
        x: (tileSize + this.gap) * (this.slotsX / 2.0) - (tileSize * 0.2 / 2),
        y: 0,
        fill: "rgba(50,50,255,0.3)",
        width: tileSize * 0.2,
        height: (this.slotsY + 1) * tileSize
      }
    },
    GetMirrorYData() {
      if(!this.mirrorY) return {}
      let tileSize = this.tileSize;

      return {
        y: (tileSize + this.gap) * (this.slotsY / 2.0) - (tileSize * 0.2 / 2),
        x: 0,
        fill: "rgba(50,255,50,0.3)",
        height: tileSize * 0.2,
        width: (this.slotsX + 1) * tileSize
      }
    },
    KeyToCoordinate(key, countX, countY) {
      let posX = key%countX;
      let posY = (key - posX) / countX;
      return {x: posX, y: posY}
    },
    tileLeave(ev, slot) {
      if(this.hoverPosition !== slot) return;
      this.$emit("slot:leave", [slot]);
    },
    tileHover(ev, slot) {
      if(this.hoverPosition === slot) return;
      this.$emit("slot:hover", [slot]);
    },
    isHovered(pos) {
      let hoveredPosition = this.KeyToCoordinate(this.hoverPosition);
        return this.hoverPosition > -1 && hoveredPosition.x === pos.x && hoveredPosition.y === pos.y
    }
    ,
    TypeToColor(type, hover) {
      let colors = {
        "-1": [0,0,0],
        "0": [0,0,0],
        "1": [128,128,128],
        "2": [166,107,0],
        "3": [192,192,192],
        "C": [0,172,255],
        "T": [255,128,0],
      }

      let pickedColor = colors[type];

      if(typeof pickedColor === "undefined") {
        pickedColor = colors["-1"];
      }

      if(hover) {
        let additional = [30,30,30];
        for(let i = 0; i < pickedColor.length; i++) {
          pickedColor[i] = Math.min(pickedColor[i] + additional[i], 255);
        }
      }

      return "rgba(" + pickedColor.join(',') +",1)";

    },
    TypeToBorder(type) {
      switch(type) {
        case -1: return "black";
        case '0': return "0px black";
        case '1': return "2px solid rgba(30,30,30,0.5)";
        case '2': return "2px solid rgba(50,50,50,0.3)";
        case '3': return "0px black";
        case 'C': return "0px black";
        case 'T': return "0px black";
        default:
          return "rgba(" + [Math.min(255,33*type),Math.min(255,33*type),Math.min(255,33*type),1].join(",") + ")"
      }
    }
  },
  computed: {
    refData() {
      if(this.typeRef !== undefined && this.typeRef !== null) return this.typeRef.typeData;
      return [3];
    },
    tileSize() {
      return Math.min((this.canvasWidth-this.slotsX*3) / this.slotsX, (this.canvasHeight - this.slotsY*3) / this.slotsY);
    },
    positions() {
      let pos = [];
      let tileSize = this.tileSize;
      let maxSlots = this.slotsX * this.slotsY;
      for(let i = 0; i < Math.min(this.refData.length, maxSlots); i++) {
        let coord = this.KeyToCoordinate(i, this.slotsX, this.slotsY);
        let hover = false;
        if(this.isHovered(coord)) {
          hover= true;
        }

        if(hover) {
          console.log("got some hover, baby");
        }

        pos.push({
          x: coord.x * (tileSize+ this.gap),
          y: coord.y * (tileSize+ this.gap),
          width: tileSize,
          height: tileSize,
          fill: this.TypeToColor(this.refData[i], hover),
          style: {"border":this.TypeToBorder(this.refData[i])},
        })
      }
      return pos;
    }
  }
}
</script>

<style scoped>

</style>
