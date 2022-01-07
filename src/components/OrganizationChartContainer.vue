<template>
  <div v-bind="{ scopedSlots: $scopedSlots }"
    class="orgchart-container"
    @wheel="zoom && zoomHandler($event)"
    @mouseup="pan && panning && panEndHandler($event)"
  >
    <div
      class="orgchart"
      :style="{ transform: transformVal, cursor: cursorVal }"
      @mousedown="pan && panStartHandler($event)"
      @mousemove="pan && panning && panHandler($event)"
    >
      <organization-chart-node :datasource="datasource" :handle-click="handleClick">
        <template v-for="slot in Object.keys($scopedSlots)" :slot="slot" slot-scope="scope">
          <slot :name="slot" v-bind="scope"/>
        </template>
      </organization-chart-node>
    </div>
  </div>
</template>

<script>
import $ from 'jquery'
import OrganizationChartNode from './OrganizationChartNode.vue'

export default {
  name: 'Container',
  props: {
    datasource: {
      type: Object,
      required: true
    },
    pan: {
      type: Boolean,
      required: false
    },
    zoom: {
      type: Boolean,
      required: false
    },
    zoomoutLimit: {
      type: Number,
      required: false,
      default: 0.5
    },
    zoominLimit: {
      type: Number,
      required: false,
      default: 7
    }
  },
  mounted() {
    $(".orgchart .node .popup").hide();//This hides all the pop-ups when page loads

    document.querySelectorAll('.orgchart .node').forEach(node => {
      node.onmouseenter = node.onmouseleave = handler;
    })

    function handler(event) {
      let type = event.type;
      switch (type) {
        case 'mouseenter': 
          const callOver = over.bind(this, event);
          callOver();
          break;
        case 'mousemove': 
          const callMove = move.bind(this, event);
          callMove();
          break;
        case 'mouseleave': 
          const callOut = out.bind(this, event);
          callOut();
          break;
        
        default:
          break;
      }

      return false;
    }

    function over(event) { 
      const selector = "#popup-"+this.id; //creates a value of "#popup1", "#popup2", etc for future use

      const popup = this.querySelector('.popup')
      const adjustX = popup.offsetWidth
      const adjustY = popup.offsetHeight// -30

      const rect = this.getBoundingClientRect()
      const offsetTop = rect.top + window.scrollY
      const offsetLeft = rect.left + window.scrollX

      let my = event.pageY-offsetTop-adjustY; //my = mouse y position with some adjustment relateive to top of box
      let mx = event.pageX-offsetLeft-adjustX; //mx = mouse x position with some adjustment relateive to left of box

      popup.style.left  = (event.pageX <= 250 ? mx * 0.5   : mx)+'px';
      popup.style.top   = (event.pageY <= 80  ? my * -0.5 : my)+'px';
      $(selector).show(); //This reveals the popup inside the hovered box
    }

    function move(event) { 
      const popup = this.querySelector('.popup')
      const adjustX = popup.offsetWidth
      const adjustY = popup.offsetHeight -30

      const rect = this.getBoundingClientRect()
      const offsetTop = rect.top + window.scrollY
      const offsetLeft = rect.left + window.scrollX

      var my = event.pageY-offsetTop-adjustY;//my = mouse y position with some adjustment relateive to top of box
      var mx = event.pageX-offsetLeft-adjustX; //mx = mouse x position with some adjustment relateive to left of box

      popup.style.left = mx+'px';
      popup.style.top = (event.pageY <= 80 ? my * -0.5 : my)+'px';
    }

    function out() { 
      const popup = this.querySelector('.popup')
      popup.style.left = 0;
      popup.style.top = 0;

      const boxHovered = this.id;//Gets the id of the box such as "box1", "box2"
      const selector = "#popup-"+boxHovered;//creates a value of "#popup1", "#popup2", etc for future use
      $(selector).hide();//This hides the popup inside the hovered box
    }
  },
  data () {
    return {
      cursorVal: 'default',
      panning: false,
      startX: 0,
      startY: 0,
      transformVal: ''
    }
  },
  components: {
    OrganizationChartNode
  },
  methods: {
    handleClick (nodeData) {
      this.$emit('node-click', nodeData);
    },
    panEndHandler () {
      this.panning = false
      this.cursorVal = 'default'
    },
    panHandler (e) {
        let newX = 0
        let newY = 0
        if (!e.targetTouches) { // pand on desktop
          newX = e.pageX - this.startX
          newY = e.pageY - this.startY
        } else if (e.targetTouches.length === 1) { // pan on mobile device
          newX = e.targetTouches[0].pageX - this.startX
          newY = e.targetTouches[0].pageY - this.startY
        } else if (e.targetTouches.length > 1) {
          return;
        }
        if (this.transformVal === '') {
          if (this.transformVal.indexOf('3d') === -1) {
            this.transformVal = 'matrix(1,0,0,1,' + newX + ',' + newY + ')'
          } else {
            this.transformVal = 'matrix3d(1,0,0,0,0,1,0,0,0,0,1,0,' + newX + ', ' + newY + ',0,1)'
          }
        } else {
          let matrix = this.transformVal.split(',')
          if (this.transformVal.indexOf('3d') === -1) {
            matrix[4] = newX
            matrix[5] = newY + ')'
          } else {
            matrix[12] = newX
            matrix[13] = newY
          }
          this.transformVal = matrix.join(',')
        }
    },
    panStartHandler (e) {
      if ($(e.target).closest('.node').length) {
        this.panning = false
        return
      } else {
        this.cursorVal = 'move'
        this.panning = true
      }
      let lastX = 0
      let lastY = 0
      if (this.transformVal !== '') {
        let matrix = this.transformVal.split(',')
        if (this.transformVal.indexOf('3d') === -1) {
          lastX = parseInt(matrix[4])
          lastY = parseInt(matrix[5])
        } else {
          lastX = parseInt(matrix[12])
          lastY = parseInt(matrix[13])
        }
      }
      if (!e.targetTouches) { // pand on desktop
        this.startX = e.pageX - lastX
        this.startY = e.pageY - lastY
      } else if (e.targetTouches.length === 1) { // pan on mobile device
        this.startX = e.targetTouches[0].pageX - lastX
        this.startY = e.targetTouches[0].pageY - lastY
      } else if (e.targetTouches.length > 1) {
        return
      }
    },
    setChartScale (newScale) {
      let matrix = ''
      let targetScale = 1
      if (this.transformVal === '') {
        this.transformVal = 'matrix(' + newScale + ', 0, 0, ' + newScale + ', 0, 0)'
      } else {
        matrix = this.transformVal.split(',')
        if (this.transformVal.indexOf('3d') === -1) {
          targetScale = Math.abs(window.parseFloat(matrix[3]) * newScale)
          if (targetScale > this.zoomoutLimit && targetScale < this.zoominLimit) {
            matrix[0] = 'matrix(' + targetScale
            matrix[3] = targetScale
            this.transformVal = matrix.join(',')
          }
        } else {
          targetScale = Math.abs(window.parseFloat(matrix[5]) * newScale)
          if (targetScale > this.zoomoutLimit && targetScale < this.zoominLimit) {
            matrix[0] = 'matrix3d(' + targetScale
            matrix[5] = targetScale
            this.transformVal = matrix.join(',')
          }
        }
      }
    },
    zoomHandler (e) {
      let newScale  = 1 + (e.deltaY > 0 ? -0.2 : 0.2)
      this.setChartScale(newScale)
    },
    uuidv4() {
      return ([1e7]+-1e3+-4e3+-8e3+-1e11).replace(/[018]/g, c =>
        (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
    );
}

  }
};
</script>

<style>
.orgchart-container {
  position: relative;
  display: inline-block;
  height: 800px;
  width: calc(100% - 24px);
  border: 2px dashed #aaa;
  border-radius: 5px;
  overflow: auto;
  text-align: center;
}
.orgchart {
  box-sizing: border-box;
  display: inline-block;
   height: 800px;
  width: 100%;
  min-height: 202px;
  min-width: 202px;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  /* background-image: linear-gradient(
      90deg,
      rgba(200, 0, 0, 0.15) 10%,
      rgba(0, 0, 0, 0) 10%
    ),
    linear-gradient(rgba(200, 0, 0, 0.15) 10%, rgba(0, 0, 0, 0) 10%);
  background-size: 10px 10px; */
  border: 1px dashed rgba(0, 0, 0, 0);
  padding: 20px;
}

.orgchart .hidden,
.orgchart ~ .hidden {
  display: none;
}

.orgchart.b2t {
  transform: rotate(180deg);
}

.orgchart.l2r {
  position: absolute;
  transform: rotate(-90deg) rotateY(180deg);
  transform-origin: left top;
}

.orgchart .verticalNodes ul {
  list-style: none;
  margin: 0;
  padding-left: 18px;
  text-align: left;
}
.orgchart .verticalNodes ul:first-child {
  margin-top: 2px;
}
.orgchart .verticalNodes > td::before {
  content: "";
  border: 1px solid rgba(217, 83, 79, 0.8);
}
.orgchart .verticalNodes > td > ul > li:first-child::before {
  box-sizing: border-box;
  top: -4px;
  height: 30px;
  width: calc(50% - 2px);
  border-width: 2px 0 0 2px;
}
.orgchart .verticalNodes ul > li {
  position: relative;
}
.orgchart .verticalNodes ul > li::before,
.orgchart .verticalNodes ul > li::after {
  box-sizing: border-box;
  content: "";
  position: absolute;
  left: -6px;
  border-color: rgba(217, 83, 79, 0.8);
  border-style: solid;
  border-width: 0 0 2px 2px;
}
.orgchart .verticalNodes ul > li::before {
  top: -4px;
  height: 30px;
  width: 11px;
}
.orgchart .verticalNodes ul > li::after {
  top: 1px;
  height: 100%;
}
.orgchart .verticalNodes ul > li:first-child::after {
  box-sizing: border-box;
  top: 24px;
  width: 11px;
  border-width: 2px 0 0 2px;
}
.orgchart .verticalNodes ul > li:last-child::after {
  box-sizing: border-box;
  border-width: 2px 0 0;
}

.orgchart.r2l {
  position: absolute;
  transform: rotate(90deg);
  transform-origin: left top;
}

.orgchart > .spinner {
  font-size: 100px;
  margin-top: 30px;
  color: rgba(68, 157, 68, 0.8);
}

.orgchart table {
  border-spacing: 0;
  border-collapse: separate;
}

.orgchart > table:first-child {
  margin: 20px auto;
}

.orgchart td {
  text-align: center;
  vertical-align: top;
  padding: 0;
}

.orgchart .lines:nth-child(3) td {
  box-sizing: border-box;
  height: 20px;
}

.orgchart .lines .topLine {
  border-top: 2px solid rgba(217, 83, 79, 0.8);
}

.orgchart .lines .rightLine {
  border-right: 1px solid rgba(217, 83, 79, 0.8);
  float: none;
  border-radius: 0;
}

.orgchart .lines .leftLine {
  border-left: 1px solid rgba(217, 83, 79, 0.8);
  float: none;
  border-radius: 0;
}

.orgchart .lines .downLine {
  background-color: rgba(217, 83, 79, 0.8);
  margin: 0 auto;
  height: 20px;
  width: 2px;
  float: none;
}

/* node styling */
.orgchart .node {
  box-sizing: border-box;
  display: inline-block;
  position: relative;
  margin: 0;
  padding: 3px;
  border: 2px dashed transparent;
  text-align: center;
  cursor: pointer;
  /* width: 130px; */
}
.orgchart .node .tool-tips {
  display: none;
}

.orgchart.l2r .node,
.orgchart.r2l .node {
  width: 50px;
  height: 130px;
}

.orgchart .node > .spinner {
  position: absolute;
  top: calc(50% - 15px);
  left: calc(50% - 15px);
  vertical-align: middle;
  font-size: 30px;
  color: rgba(68, 157, 68, 0.8);
}

.orgchart .node:hover {
  background-color: rgba(238, 217, 54, 0.5);
  transition: 0.5s;
  cursor: default;
  z-index: 20;
}

.orgchart .node .popup {
  position: absolute;/*VERY IMPORTANT*/
  width: 350px;
  min-height: 150px;
  /* background-color: #0C9; */
  background-color: #74EBD5;
  background-image: linear-gradient(270deg, #74EBD5 0%, #9FACE6 100%);

  border: solid 1px teal;
  border-radius: 5px;
  left: 0px;/*This value gets replaced in javascript so ANY number is OK*/
  top: 0px;/*This value gets replaced in javascript so ANY number is OK*/
  animation: fadeInAnimation ease 0.5s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
}

@keyframes fadeInAnimation {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

.orgchart .node .popup .lbl-name{
  font-size: 1rem;
  font-weight: bold;
}

.orgchart .node .popup .lbl-field{
  font-size: 0.8rem;
  font-weight: 700;
}

.orgchart .node .popup .lbl-value{
  font-size: 0.7rem;
}


/* 
.orgchart .node:hover > div.tool-tips {
  position: absoluted;
  display: flex;
  justify-content: space-between; 
} */

.orgchart .node.focused {
  background-color: rgba(238, 217, 54, 0.5);
}

.orgchart .ghost-node {
  position: fixed;
  left: -10000px;
  top: -10000px;
}

.orgchart .ghost-node rect {
  fill: #ffffff;
  stroke: #bf0000;
}

.orgchart .node.allowedDrop {
  border-color: rgba(68, 157, 68, 0.9);
}

.orgchart .node .title {
  text-align: center;
  font-size: 12px;
  font-weight: bold;
  height: 40px;
  min-width: 200px;
  line-height: 20px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  background-color: rgb(59 58 143 / 80%);
  color: #fff;
  border-radius: 4px 4px 0 0;
}

.orgchart.b2t .node .title {
  transform: rotate(-180deg);
  transform-origin: center bottom;
}

.orgchart.l2r .node .title {
  transform: rotate(-90deg) translate(-40px, -40px) rotateY(180deg);
  transform-origin: bottom center;
  width: 120px;
}

.orgchart.r2l .node .title {
  transform: rotate(-90deg) translate(-40px, -40px);
  transform-origin: bottom center;
  width: 120px;
}

.orgchart .node .profile-picture {
  border-radius: 50%;
}

.orgchart .node .title .symbol {
  float: left;
  margin-top: 4px;
  margin-left: 2px;
}

.orgchart .node .content {
  display: flex;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
  width: 100%;
  height: 60px;
  min-width: 200px;
  font-size: 11px;
  line-height: 18px;
  border: 1px solid rgba(217, 83, 79, 0.8);
  border-radius: 0 0 4px 4px;
  text-align: center;
  background-color: #fff;
  color: #333;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.orgchart.b2t .node .content {
  transform: rotate(180deg);
  transform-origin: center top;
}

.orgchart.l2r .node .content {
  transform: rotate(-90deg) translate(-40px, -40px) rotateY(180deg);
  transform-origin: top center;
  width: 120px;
}

.orgchart.r2l .node .content {
  transform: rotate(-90deg) translate(-40px, -40px);
  transform-origin: top center;
  width: 120px;
}

.orgchart .node .edge {
  font-size: 15px;
  position: absolute;
  color: rgba(68, 157, 68, 0.5);
  cursor: default;
  transition: 0.2s;
}

.orgchart.noncollapsable .node .edge {
  display: none;
}

.orgchart .edge:hover {
  color: #449d44;
  cursor: pointer;
}

.orgchart .node .verticalEdge {
  width: calc(100% - 10px);
  width: -webkit-calc(100% - 10px);
  width: -moz-calc(100% - 10px);
  left: 5px;
}

.orgchart .node .topEdge {
  top: -4px;
}

.orgchart .node .bottomEdge {
  bottom: -4px;
}

.orgchart .node .horizontalEdge {
  width: 15px;
  height: calc(100% - 10px);
  height: -webkit-calc(100% - 10px);
  height: -moz-calc(100% - 10px);
  top: 5px;
}

.orgchart .node .rightEdge {
  right: -4px;
}

.orgchart .node .leftEdge {
  left: -4px;
}

.orgchart .node .horizontalEdge::before {
  position: absolute;
  top: calc(50% - 7px);
}

.orgchart .node .rightEdge::before {
  right: 3px;
}

.orgchart .node .leftEdge::before {
  left: 3px;
}

.orgchart .node .toggleBtn {
  position: absolute;
  left: 5px;
  bottom: -2px;
  color: rgba(68, 157, 68, 0.6);
}

.orgchart .node .toggleBtn:hover {
  color: rgba(68, 157, 68, 0.8);
}

.oc-export-btn {
  display: inline-block;
  position: absolute;
  right: 5px;
  top: 5px;
  padding: 6px 12px;
  margin-bottom: 0;
  font-size: 14px;
  font-weight: 400;
  line-height: 1.42857143;
  text-align: center;
  white-space: nowrap;
  vertical-align: middle;
  touch-action: manipulation;
  cursor: pointer;
  user-select: none;
  color: #fff;
  background-color: #5cb85c;
  border: 1px solid transparent;
  border-color: #4cae4c;
  border-radius: 4px;
}

.oc-export-btn[disabled] {
  cursor: not-allowed;
  box-shadow: none;
  opacity: 0.3;
}

.oc-export-btn:hover,
.oc-export-btn:focus,
.oc-export-btn:active {
  background-color: #449d44;
  border-color: #347a34;
}

.orgchart ~ .mask {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 999;
  text-align: center;
  background-color: rgba(0, 0, 0, 0.3);
}

.orgchart ~ .mask .spinner {
  position: absolute;
  top: calc(50% - 54px);
  left: calc(50% - 54px);
  color: rgba(255, 255, 255, 0.8);
  font-size: 108px;
}

.orgchart .node {
  transition: transform 0.3s, opacity 0.3s;
}

.orgchart .slide-down {
  opacity: 0;
  transform: translateY(40px);
}

.orgchart.l2r .node.slide-down,
.orgchart.r2l .node.slide-down {
  transform: translateY(130px);
}

.orgchart .slide-up {
  opacity: 0;
  transform: translateY(-40px);
}

.orgchart.l2r .node.slide-up,
.orgchart.r2l .node.slide-up {
  transform: translateY(-130px);
}

.orgchart .slide-right {
  opacity: 0;
  transform: translateX(130px);
}

.orgchart.l2r .node.slide-right,
.orgchart.r2l .node.slide-right {
  transform: translateX(40px);
}

.orgchart .slide-left {
  opacity: 0;
  transform: translateX(-130px);
}

.orgchart.l2r .node.slide-left,
.orgchart.r2l .node.slide-left {
  transform: translateX(-40px);
}
</style>
