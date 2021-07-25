<template>
  <div class="planner-host">
    <div class="planner-panel">
      <h3>I bimz, dat Prototype!</h3>
      <div v-if="modalShown" class="modal">
        <div class="card">
          <textarea ref="codeArea" @click="$refs.codeArea.select()" readonly>{{ generatedCode }}</textarea>
          <button @click="saveToFile(generatedCode)">Speichern</button>
          <button @click="modalShown = false">Schließen</button>
        </div>
      </div>
      <div class="planner-input">
        <div><label>slotsX</label><input type="number" min="1" max="100" step="1" v-model="slotsX"></div>
        <div><label>slotsY</label><input type="number" min="1" max="100" step="1" v-model="slotsY"></div>
        <div><label>radius</label><input type="number" min="1" max="100" step="1" v-model="radius"></div>
        <div><label>renderSize</label><input type="number" min="1" max="100" step="1" v-model="renderSize"></div>
        <div><label>renderGap</label><input type="number" min="1" max="100" step="1" v-model="renderGap"></div>
      </div>
      <div class="planner-editor-section">
        <div class="planner-tile-list">
          <button type="button" @click="(currentValue = '0')">0 empty</button>
          <button type="button" @click="(currentValue = '1')">1 armor/wall</button>
          <button type="button" @click="(currentValue = '2')">2 door</button>
          <button type="button" @click="(currentValue = '3')">3 floor</button>
          <button type="button" @click="(currentValue = 'C')">C canon</button>
          <button type="button" @click="(currentValue = 'T')">T thruster</button>
        </div>
        <div class="planner-editor-preview"
        @mousedown="onMouseDown($event)"
        @mouseup="onMouseUp($event)"
        >
          <div><small>Selected Value: {{ currentValue }}</small></div>
          <PrototypeGrid :key="GridKey" :canvas-height="400" :slots-x="SlotsX" :slots-y="SlotsY" :canvas-width="600" :type-ref="typeRef"
                          :mirror-x="mirrorX"
                          :mirror-y="mirrorY"
                         :hover-position="mouseOver"
                         @slot:leave="onSlotLeave($event)"
                         @slot:hover="onSlotHover($event)"
                         @slot:changed="onSlotChanged($event)"
          ></PrototypeGrid>
        </div>
        <div class="planner-tile-list">
          <button type="button" @click="(mirrorX = !mirrorX)">Mirror X</button>
          <button type="button" @click="(mirrorY = !mirrorY)">Mirror Y</button>
        </div>
      </div>
      <div class="planner-footer">
        <button type="button" @click="showCode($event)">Convert</button>
      </div>
    </div>

  </div>
</template>

<script>
import PrototypeGrid from "../components/PrototypeGrid";
export default {
  name: "PrototypePlannerPage",
  components: {PrototypeGrid},
  data(){return{
    mirrorX: false,
    mirrorY: false,
    currentValue: 0,
    slotsX: 12,
    slotsY: 8,
    radius: 1,
    renderSize: 10,
    renderGap: 1,
    rects: [
        ['0','0','0','0','0','1','1','1','0','0','0','0','0'],
        ['0','0','0','0','0','1','1','1','0','0','0','0','0'],
        ['0','0','0','0','1','1','3','1','1','0','0','0','0'],
        ['0','0','0','1','1','3','3','3','1','1','0','0','0'],
        ['0','0','0','T','3','3','3','3','3','T','0','0','0'],
        ['0','0','1','1','1','1','2','1','1','1','1','0','0'],
        ['0','1','1','3','3','1','3','1','3','3','1','1','0'],
        ['0','C','3','3','3','2','3','2','3','3','3','C','0'],
        ['0','1','1','1','1','1','3','1','1','1','1','1','0'],
        ['0','0','1','3','3','2','3','2','3','3','1','0','0'],
        ['0','0','1','1','1','1','3','1','1','1','1','0','0'],
        ['0','1','1','3','3','1','3','1','3','3','1','1','0'],
        ['0','C','3','3','3','2','3','2','3','3','3','C','0'],
        ['0','1','1','1','1','1','2','1','1','1','1','1','0'],
        ['0','0','1','1','3','3','3','3','3','1','1','0','0'],
        ['0','0','0','1','3','3','3','3','3','1','0','0','0'],
        ['0','0','0','1','T','1','T','1','T','1','0','0','0'],
        ['0','0','0','1','T','1','T','1','T','1','0','0','0'],
    ],
    iteration: 0,
    typeData: [],
    typeRef: {typeData: []},
    posData: [],
    mouseOver: -1,
    mousePressed: false,
    modalShown: false,
    generatedCode: "",
    format: {
      prefix: "private static readonly string shipString = new [] {",
      postfix: "};",
      rowStart: ' "',
      rowEnd: '",',
      lastRowEnd: '"'
    }
  }},
  computed: {
    GeneratedCode() {},
    TypeData() {return this.typeData;},
    GridKey() {return [
      this.SlotsX,
      this.SlotsY,
      this.Radius,
      this.RectSize,
      this.GapSize,
        this.Iteration,
    ].join(',')},
    Iteration() {return this.iteration||0;},
    SlotsX(){return parseInt(this.slotsX, 10);},
    SlotsY(){return parseInt(this.slotsY, 10);},
    GapSize(){return parseInt(this.renderGap, 10);},
    RectSize(){return parseInt(this.renderSize, 10);},
    Radius(){return parseInt(this.radius, 10);},
  },
  watch: {
    slotsX(nu, ol) {this.shiftData(ol, nu); this.prepareData()},
    slotsY(nu, ol) {this.prepareData()},
    renderGap(nu, ol) {this.prepareData()},
    renderSize(nu, ol) {this.prepareData()},
  },
  created() {
    let tmpTypeStack = [];
    this.slotsY = this.rects.length;
    this.slotsX = this.rects[0].length;

    for(let y = 0; y < this.rects.length; y++) {
      for(let x = 0; x < this.rects[y].length; x++) {
        tmpTypeStack.push(this.rects[y][x])
      }
    }
    this.typeRef.typeData = tmpTypeStack

    this.prepareData();
  },
  methods: {
    saveToFile(userInput) {
      const data = userInput;
      const blob = new Blob([data], {type: 'text/plain'})
      const e = document.createEvent('MouseEvents'),
          a = document.createElement('a');
      a.download = "ship.txt";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ['text/html', a.download, a.href].join(':');
      e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
      a.dispatchEvent(e);
    },
    KeyToCoordinate(key, countX, countY) {
      let posX = key%countX;
      let posY = (key - posX) / countX;
      return {x: posX, y: posY}
    },
    CoordinateToKey(coord, countX, countY) {
      return coord.y * countX + coord.x;
    },
    showRawEdit(ev) {

    },
    showCode(ev) {
      let lines = []
      let dataset = this.typeRef.typeData;
      let maxSize = this.SlotsX * this.SlotsY;
      lines.push(this.SlotsX +"|" + this.SlotsY);
      for(let y = 0; y < maxSize; y+= this.SlotsX) {
        lines.push(dataset.slice(y, y+ this.SlotsX ).map(x => x === '0' || x < 0 || x === "-1" ? 0 : x).map(x => !!x ? x : " ").join(''));
      }
      this.generatedCode = lines.join("\r\n");
      this.modalShown = true;
    },
    getMouseOverSlots(hover) {
      let hovered = [hover];
      if(this.mirrorX) {
        hovered = this.mirrorOnX(hovered)
      }
      if(this.mirrorY) {
        hovered = this.mirrorOnY(hovered)
      }

      return hovered;
    },
    mirrorOnX(slots) {
      let newSlots = [].concat(slots)
      for(let i = 0; i < slots.length; i++) {
        let pos = this.KeyToCoordinate(slots[i], this.SlotsX, this.SlotsY);
        let mirrorPos = {x: (this.SlotsX-1) - pos.x, y: pos.y}
        newSlots.push(this.CoordinateToKey(mirrorPos, this.SlotsX, this.SlotsY));
      }
      return newSlots;
    },
    mirrorOnY(slots) {
      let newSlots = [].concat(slots)
      for(let i = 0; i < slots.length; i++) {
        let pos = this.KeyToCoordinate(slots[i], this.SlotsX, this.SlotsY);
        let mirrorPos = {x: pos.x, y:(this.SlotsY-1) -  pos.y}
        newSlots.push(this.CoordinateToKey(mirrorPos, this.SlotsX, this.SlotsY));
      }
      return newSlots;
    },

    drawTile() {
      if(this.mouseOver === -1 || !this.mousePressed) return;
      let slots = this.getMouseOverSlots(this.mouseOver);
      console.log("mouse over", this.mouseOver, "slots", slots)
      for(let i = 0; i<slots.length; i++) {
        if(slots[i] < 0 || slots[i] >= this.SlotsX * this.SlotsY) continue;
        if(this.typeRef.typeData[slots[i]] === this.currentValue) continue;
        this.typeRef.typeData[slots[i]] = this.currentValue
      }

    },
    GetTile(pos) {
        return (pos >= 0 && pos < this.SlotsX * this.SlotsY) ? this.typeRef.typeData[pos] : -1;
    },
    onMouseDown(ev) {
      if(ev.button !== 0) return;
      if(ev.ctrlKey) {
        this.floodFill(this.mouseOver, this.GetTile(this.mouseOver), this.currentValue);
        this.drawTile();
        return;
      }
      console.log(ev);
      this.mousePressed = true
      this.drawTile();
    },
    floodFill(pos, expectedValue, overwriteValue) {
      if(pos >= 0 && pos < this.SlotsX * this.SlotsY) {
        if(this.typeRef.typeData[pos] === expectedValue || (expectedValue === 0 && this.typeRef.typeData[pos] <= 0)) {
          this.typeRef.typeData[pos] = overwriteValue;
          let dirs = [{x: -1, y:0}, {x: 1, y:0}, {x: 0, y: -1}, {x: 0, y:1}]
          let currPos = this.KeyToCoordinate(pos, this.SlotsX, this.SlotsY);
          let otherKey = dirs.map(dirPos => this.CoordinateToKey({x: dirPos.x + currPos.x, y: dirPos.y+currPos.y}, this.SlotsX, this.SlotsY))

          for(let i = 0; i < otherKey.length; i++) {
            this.floodFill(otherKey[i], expectedValue, overwriteValue)
          }
        }
      }
    },
    onMouseUp(ev) {
      if(ev.button !== 0) return;
      this.mousePressed = false},
    onSlotHover(ev) {
      if(this.mouseOver === ev[0]) return;
      this.mouseOver = ev[0]
      this.drawTile();
      this.prepareData()
    },
    onSlotLeave(ev) {
      if(this.mouseOver === ev[0]) {
        this.mouseOver = -1;
      }
    },
    onSlotChanged(ev) {
      console.log("planner prototype: Got change for", ev)
      if(this.typeRef.typeData.length >= ev[0] && ev[0] >= 0) {
        this.typeRef.typeData[ev[0]] = this.currentValue;
     //   this.typeData[ev[0]] = this.currentValue;

        console.log("planner prototype: set index", ev[0], "to value", this.currentValue)
        this.prepareData()
      }
    },
    shiftData(fromWidth, toWidth) {
      let y;
      let newGrid = [];
      let tmpRow = [];
      for(y = 0; y < this.SlotsY; y++) {
        tmpRow = this.typeRef.typeData.slice(y * fromWidth, (y+1) * fromWidth +1).slice(0, toWidth)
        newGrid = newGrid.concat(tmpRow)
      }
      this.typeRef.typeData = newGrid;
      console.log("newGrid", newGrid);
    },
    prepareData() {
      let types = Array.apply([], this.typeRef.typeData).slice(0, this.SlotsX * this.SlotsY);
      let positions = [];
      let x,y;
      for(y = 0; y < this.SlotsY; y++) {
        for(x = 0; x < this.SlotsX; x++) {
          let key = x * this.SlotsX + y;
          if(this.typeData.length >= key) {
            types.push(this.typeData[key])
          }
         // else if(this.typeRef.typeData.length >= key) {
         //   types.push(this.typeRef.typeData[key])
         // }
          else {
            types.push(-1);
          }
          positions.push({x: x*(this.GapSize + this.RectSize), y: y * (this.GapSize + this.RectSize),
            width: this.RectSize,
            height: this.RectSize})
        }
      }

      //this.typeData = types;
      this.typeRef.typeData = types;
//      console.log("Set types data", types);
      this.posData = positions;
      this.iteration++;
    },
    toggleRects(ev) {

    }
  }
}
</script>

<style scoped>
.planner-host {position: relative; width: 100vw; height: 100vh; padding: 2rem;
  background-color:grey;
}
.planner-panel {
  padding: 2rem;
  background-color: rgba(255,255,255, .7);
  border-radius: 1rem;
  max-width: 80vw;
}
.planner-panel h3 {border-bottom: 1px solid rgba(33,33,33,0.2)}
.planner-editor-section {
  display: inline-flex;
  max-width: 80vw;
  width: 80%;
  background-color: green;
}
.planner-tile-list {
  display: inline-flex;
  max-width: 20%;
  background-color: pink;
  padding: 1rem;
  flex-direction: column;
}
.planner-input {
  padding: 1rem;
  background-color:rgba(33,33,33,0.2);
  flex-direction: row;
  display: flex;
}

.planner-input label {
  display: block;
}
.planner-editor-preview {
  display: block;
  width: 80%;
  height: 80%;
  background-color: white;
  padding: 2rem;
}
.planner-editor-preview svg {
  border: 2px solid red;
}
.modal {
  display: block;
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(33,33,33, 0.2);
}
.card {
  background-color: white;
  position:relative;
  height: 50vh;
  width: 50vw;
  top: 25vh;
  left: 25vh;
  display: flex;
  flex-direction: column;
}

.card textarea {
  height: 75%;
}
</style>
