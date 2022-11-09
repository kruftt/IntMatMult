<script setup lang="ts">
import {
  computed,
  ref,
  reactive,
  onMounted
} from 'vue'

type Vector = { 0: number, 1: number }

const v_x = ref<SVGGElement|null>(null)

const a11 = ref(3)
const a12 = ref(1)
const a21 = ref(-1)
const a22 = ref(2)

const r1 = reactive({ 0: a11, 1: a12})
const r2 = reactive({ 0: a21, 1: a22})
const c1 = reactive({ 0: a11, 1: a21})
const c2 = reactive({ 0: a12, 1: a22})

const A = reactive([r1, r2]) // by rows
const x = reactive([2,2])
const b = reactive([8,2])

const xc1 = computed(() => [x[0]*c1[0], x[0]*c1[1]])
const xc2 = computed(() => [x[1]*c2[0], x[1]*c2[1]])
const hp1 = computed(() => getLineCoords(r1[0], r1[1], b[0]))
const hp2 = computed(() => getLineCoords(r2[0], r2[1], b[1]))

function getLineCoords(x: number, y: number, b: number) {
  if (y == 0) {
    if (x == 0) {
      return [0, 0, 0, 0]
    } else {
      // vertical line intercept b/x
      const i = b/x
      return [i, 10, i, -10]
    }
  } else {
    if (x == 0) {
      // horizontal line
      const i = b/y
      return [-10, i, 10, i]
    } else {
      if (Math.abs(x) < Math.abs(y)) {
        // todo optimize multiplications
        const m = -x/y
        const i = b/y // y intercept
        const dy = 10*m
        return [-10, i-dy, 10, i+dy]
      } else {
        const m = -y/x
        const i = b/x
        const dx = 10*m
        return [i-dx ,-10, i+dx, 10]
      }
    }
  }
}

function compute_b() {
  b[0] = r1[0]*x[0] + r1[1]*x[1]
  b[1] = r2[0]*x[0] + r2[1]*x[1]
}

function compute_x() {
  const inv_det = 1 / (A[0][0]*A[1][1] - A[1][0]*A[0][1])
  x[0] = inv_det * (A[1][1]*b[0] - A[0][1]*b[1])
  x[1] = inv_det * (A[0][0]*b[1] - A[1][0]*b[0])
}

const _deg = 180/Math.PI
const vTransform = (x: number, y: number, cx=x, cy=y) => `rotate(${_deg * Math.atan2(y, x) - 90}, ${cx}, ${cy})`


const drag_pt = new DOMPoint()
let activeVector: Vector
let activeSVG: SVGSVGElement
let coordMatrix: DOMMatrix
let rox: boolean
function startVectorDrag(e: PointerEvent, v: Vector, rowOrX: boolean) {
  console.log('start')
  activeVector = v
  
  activeSVG = (e.target! as SVGElement).ownerSVGElement!
  coordMatrix = activeSVG.getScreenCTM()!.inverse()
  rox = rowOrX

  activeSVG.addEventListener('pointermove', onVectorDrag)
  activeSVG.addEventListener('pointerup', endVectorDrag)
  activeSVG.addEventListener('pointerleave', endVectorDrag)
}

function onVectorDrag(e: PointerEvent) {
  drag_pt.x = e.clientX
  drag_pt.y = e.clientY
  const coords = drag_pt.matrixTransform(coordMatrix)
  activeVector[0] = coords.x
  activeVector[1] = coords.y
  if (rox) compute_b()
  else compute_x()
}

function endVectorDrag(e: PointerEvent) {
  console.log('end')
  activeSVG.removeEventListener('pointermove', onVectorDrag)
  activeSVG.removeEventListener('pointerup', endVectorDrag)
  activeSVG.removeEventListener('pointerleave', endVectorDrag)
}

function printCoords(e: PointerEvent) {
  const svg = e.target as SVGSVGElement
  drag_pt.x = e.clientX
  drag_pt.y = e.clientY
  console.log(drag_pt.matrixTransform(svg.getScreenCTM()!.inverse()))
}


onMounted(() => {})
</script>


<!-- sqrt(3) = 1.732 -->
<template lang="pug">
div(id="matrix_mult")
  div(id="graphs")
    div
      h4 Domain
      svg(class="graph" id="domain" width="30em" height="30em" viewBox="-10 -10 20 20" transform="scale(1,-1)")
        symbol(id="arrowhead" width="0.8" height="0.8" viewBox="-1 -1 2 2")
          polygon(points="-0.3,-1 0.3,-1 0,0")

        g(stroke="#555" stroke-width="0.1")
          line(x1="0" y1="-10" x2="0" y2="10")
          line(x1="-10" y1="0" x2="10" y2="0")
        
        g(id="hp1" class="hp row1")
          line(:x1="hp1[0]" :y1="hp1[1]" :x2="hp1[2]" :y2="hp1[3]")
          //- line(x1="-6" y1="-10" x2="10" y2="6")
        
        g(id="hp2" class="hp row2")
          line(:x1="hp2[0]" :y1="hp2[1]" :x2="hp2[2]" :y2="hp2[3]")
          //- line(x1="10" y1="-6" x2="-6" y2="10")
        
        g(id="row1" class="vec row1")
          line(x1="0" y1="0" :x2="r1[0]" :y2="r1[1]")
          use(href="#arrowhead" class="arrowhead" :x="r1[0]-0.4" :y="r1[1]-0.4" :transform="vTransform(r1[0], r1[1])" @pointerdown="(e) => startVectorDrag(e, r1, true)")
        
        g(id="row2" class="vec row2")
          line(x1="0" y1="0" :x2="r2[0]" :y2="r2[1]")
          use(href="#arrowhead" class="arrowhead" :x="r2[0]-0.4" :y="r2[1]-0.4" :transform="vTransform(r2[0], r2[1])" @pointerdown="(e) => startVectorDrag(e, r2, true)")
        
        g(ref="v_x" class="vec x")
          line(x1="0" y1="0" :x2="x[0]" :y2="x[1]")
          use(href="#arrowhead" class="arrowhead" :x="x[0]-0.4" :y="x[1]-0.4" :transform="vTransform(x[0], x[1])" @pointerdown="(e) => startVectorDrag(e, x, true)")

    div
      h4 Codomain
      svg(class="graph" id="codomain" width="30em" height="30em" viewBox="-10, -10, 20, 20" transform="scale(1,-1)")
        g(stroke="#555" stroke-width="0.1")
            line(x1="0" y1="-10" x2="0" y2="10")
            line(x1="-10" y1="0" x2="10" y2="0")

        g(class="vec col1 xcol")
          line(x1="0" y1="0" :x2="xc1[0]" :y2="xc1[1]")
          use(href="#arrowhead" class="arrowhead" :x="xc1[0]-0.4" :y="xc1[1]-0.4" :transform="vTransform(xc1[0], xc1[1])")
        
        g(class="vec col2 xcol")
          line(:x1="xc1[0]" :y1="xc1[1]" :x2="xc1[0]+xc2[0]" :y2="xc1[1]+xc2[1]")
          use(href="#arrowhead" class="arrowhead" :x="b[0]-0.4" :y="b[1]-0.4" :transform="vTransform(xc2[0], xc2[1], b[0], b[1])")

        g(id="col1" class="vec col1")
          line(x1="0" y1="0" :x2="c1[0]" :y2="c1[1]")
          use(href="#arrowhead" class="arrowhead" :x="c1[0]-0.4" :y="c1[1]-0.4" :transform="vTransform(c1[0], c1[1])" @pointerdown="(e) => startVectorDrag(e, c1, false)")
        
        g(id="col2" class="vec col2")
          line(x1="0" y1="0" :x2="c2[0]" :y2="c2[1]")
          use(href="#arrowhead" class="arrowhead" :x="c2[0]-0.4" :y="c2[1]-0.4" :transform="vTransform(c2[0], c2[1])"  @pointerdown="(e) => startVectorDrag(e, c2, false)")

        g(id="b" class="vec b")
          line(x1="0" y1="0" :x2="b[0]" :y2="b[1]")
          use(href="#arrowhead" class="arrowhead" :x="b[0]-0.4" :y="b[1]-0.4" :transform="vTransform(b[0], b[1])" @pointerdown="(e) => startVectorDrag(e, b, false)")
        

  div(id="equation")
    svg(class="bracket" width="1em" height="5em" viewBox="0 0 1 5")
      symbol(id="bracket" viewBox="-0.25 0 0.25 5" width="0.5" height="5")
        path(stroke-width="0.1" stroke="#fff" fill="transparent" d="M 0.2 0.5 L -0.2 0.5 L -0.2 4.5 L 0.2 4.5")
      use(href="#bracket")

    table
      tr
        td
          input(type="number" class="num_input" v-model="A[0][0]" @input="compute_b" step="0.1")
        td
          input(type="number" class="num_input" v-model="A[0][1]" @input="compute_b" step="0.1")
      tr
        td
          input(type="number" class="num_input" v-model="A[1][0]" @input="compute_b" step="0.1")
        td
          input(type="number" class="num_input" v-model="A[1][1]" @input="compute_b" step="0.1")
    
    svg(class="bracket" width="1.5em" height="5em" viewBox="0 0 1.5 5")
      use(href="#bracket" transform="scale(-1 1)" x="-0.5" y="0")
      use(href="#bracket" x="1" y="0")
      
    table(class="vtable")
      tr
        td
          input(type="number" class="num_input" v-model="x[0]" @input="compute_b" step="0.1")
      tr
        td
          input(type="number" class="num_input" v-model="x[1]" @input="compute_b" step="0.1")
    
    svg(class="bracket" width="3em" height="5em" viewBox="0 0 3 5")
      use(href="#bracket" transform="scale(-1 1)" x="-0.5" y="0")
      line(x1="1" y1="2.3" x2="2" y2="2.3" stroke="#fff" stroke-width="0.1")
      line(x1="1" y1="2.7" x2="2" y2="2.7" stroke="#fff" stroke-width="0.1")
      use(href="#bracket" x="2.5" y="0")

    table(class="vtable")
      tr
        td
          input(type="number" class="num_input" v-model="b[0]" @input="compute_x" step="0.1")
      tr
        td
          input(type="number" class="num_input" v-model="b[1]" @input="compute_x" step="0.1")
    
    svg(class="bracket" width="1em" height="5em" viewBox="0 0 1 5")
      use(href="#bracket" transform="scale(-1 1)" x="-0.5" y="0")
    
</template>


<style scoped lang="stylus">
body
  background-color #242424
// #matrix_mult
//   width: 100%

h4
  margin 0.5em

#graphs
  display flex
  flex-wrap wrap
  justify-content center
  gap 3em

.graph
  border 2px solid black
  background-color #181818



.arrowhead
  &:hover
    stroke-width 0.6

.hp
  stroke-width 0.08
  opacity 0.25

.vec
  stroke-width 0.1

.row1
  fill #FF0
  stroke #FF0

.row2
  fill #F80
  stroke #F80

.x
  fill #F00
  stroke #F00

.col1
  fill #0F4
  stroke #0F4

.col2
  fill #0DD
  stroke #0DD

.xcol
  opacity 0.25

.b
  fill #A0F
  stroke #A0F

#equation
  display flex
  justify-content center
  align-items center
  gap 0.05em

  // font-size 7em
  color #666
  // text-align center

.num_input
  border none
  background-color #0000
  color #bbb
  text-align center
  width 2.9em
  padding 0.5em 0
  white-space nowrap
  overflow hidden
  text-overflow clip
  &:focus
    outline none

table
  margin 0 0 0 0.1em
  
.vtable
  margin-right -0.8em
</style>
