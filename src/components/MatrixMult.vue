<script setup lang="ts">
import { computed, ref, reactive } from 'vue'

type Vector = { 0: number, 1: number }

const a11 = ref(3)
const a12 = ref(1)
const a21 = ref(-1)
const a22 = ref(2)

const r1 = reactive({ 0: a11, 1: a12})
const r2 = reactive({ 0: a21, 1: a22})
const c1 = reactive({ 0: a11, 1: a21})
const c2 = reactive({ 0: a12, 1: a22})

const A = reactive([r1, r2])
const x = reactive([2,2])
const b = reactive([8,2])

const xc1 = computed(() => [x[0]*c1[0], x[0]*c1[1]])
const xc2 = computed(() => [x[1]*c2[0], x[1]*c2[1]])
const hp1 = computed(() => getLineCoords(r1[0], r1[1], b[0]))
const hp2 = computed(() => getLineCoords(r2[0], r2[1], b[1]))

enum SlnType { None, Point, Line }
const slnType = ref(SlnType.Point)

function getLineCoords(x: number, y: number, b: number) {
  if (y == 0) {
    if (x == 0) {
      return [0, 0, 0, 0]
    } else {
      const i = b/x
      return [i, 10, i, -10]
    }
  } else {
    if (x == 0) {
      const i = b/y
      return [-10, i, 10, i]
    } else {
      if (Math.abs(x) < Math.abs(y)) {
        const m = -x/y
        const i = b/y
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
  const det = (A[0][0]*A[1][1] - A[1][0]*A[0][1])
  if (det == 0) {
    slnType.value = SlnType.Line
  } else {
    slnType.value = SlnType.Point
  }
}

function compute_x() {
  const det = (A[0][0]*A[1][1] - A[1][0]*A[0][1])
  if (det == 0) {
    if (b[0]*(c1[1]+c2[1]) != b[1]*(c1[0]+c2[0])) {
      slnType.value = SlnType.None
    } else {    
      const sr1 = r1[0] + r2[0]
      const sr2 = r1[1] + r2[1]
      if (sr1 == 0 && sr2 == 0) {
        slnType.value = SlnType.None
        return
      }
      const sb = b[0] + b[1]
      const sq = sr1*sr1 + sr2*sr2
      x[0] = (sb*sr1/sq)
      x[1] = (sb*sr2/sq)
      slnType.value = SlnType.Line
    }
  } else {
    const inv_det = 1 / det
    x[0] = inv_det * (A[1][1]*b[0] - A[0][1]*b[1])
    x[1] = inv_det * (A[0][0]*b[1] - A[1][0]*b[0])
    slnType.value = SlnType.Point
  }
}


const _deg = 180/Math.PI
const vectorTransform = (v: Vector) => `translate(${v[0]}, ${v[1]})`
const arrowheadTransform = (v: Vector) => `rotate(${_deg * Math.atan2(v[1], v[0]) - 90}, 0, 0)`
const labelTransform = (v: Vector) => {
  const ang = Math.atan2(v[1], v[0])
  return `translate(${0.8 * Math.cos(ang)}, ${0.8 * Math.sin(ang)})`
}


const drag_pt = new DOMPoint()
let activeVector: Vector
let activeSVG: SVGSVGElement
let coordMatrix: DOMMatrix
let draggingB: boolean

function startVectorDrag(e: PointerEvent, v: Vector, isB: boolean = false) {
  activeVector = v
  activeSVG = (e.target! as SVGElement).ownerSVGElement!
  coordMatrix = activeSVG.getScreenCTM()!.inverse()
  draggingB = isB
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
  if (draggingB) compute_x()
  else compute_b()
}

function endVectorDrag(e: PointerEvent) {
  activeSVG.removeEventListener('pointermove', onVectorDrag)
  activeSVG.removeEventListener('pointerup', endVectorDrag)
  activeSVG.removeEventListener('pointerleave', endVectorDrag)
}
</script>


<template lang="pug">
div(id="matrix_mult")
  div(id="graphs")
    div
      h4 Domain
      svg(class="graph" id="domain" width="30em" height="30em" viewBox="-10 -10 20 20" transform="scale(1,-1)")
        symbol(id="arrowhead" width="1.0" height="1.0" viewBox="-1 -1 2 2")
          polygon(points="-0.3,-1 0.3,-1 0,0")

        g(stroke="#555" stroke-width="0.1")
          line(x1="0" y1="-10" x2="0" y2="10")
          line(x1="-10" y1="0" x2="10" y2="0")
        
        g(class="hp row1")
          line(:x1="hp1[0]" :y1="hp1[1]" :x2="hp1[2]" :y2="hp1[3]")
        
        g(class="hp row2")
          line(:x1="hp2[0]" :y1="hp2[1]" :x2="hp2[2]" :y2="hp2[3]")
        
        g(v-if="slnType == SlnType.Line" class="x")
          line(:x1="hp2[0]" :y1="hp2[1]" :x2="hp2[2]" :y2="hp2[3]" stroke-width="0.1")
        
        g(class="vec row1")
          line(x1="0" y1="0" :x2="r1[0]" :y2="r1[1]")
          g(:transform="vectorTransform(r1)")
            g(:transform="labelTransform(r1)")
              text(class="vlabel") r1
            use(href="#arrowhead" class="arrowhead" x="-0.5" y="-0.5"
              :transform="arrowheadTransform(r1)" @pointerdown="(e) => startVectorDrag(e, r1)")
        
        g(class="vec row2")
          line(x1="0" y1="0" :x2="r2[0]" :y2="r2[1]")
          g(:transform="vectorTransform(r2)")
            g(:transform="labelTransform(r2)")
              text(class="vlabel") r2
            use(href="#arrowhead" class="arrowhead" x="-0.5" y="-0.5"
              :transform="arrowheadTransform(r2)" @pointerdown="(e) => startVectorDrag(e, r2)")
        
        g(v-if="slnType != SlnType.None" class="vec x")
          line(x1="0" y1="0" :x2="x[0]" :y2="x[1]")
          g(:transform="vectorTransform(x)")
            g(:transform="labelTransform(x)")
              text(class="vlabel") x
            use(href="#arrowhead" class="arrowhead" x="-0.5" y="-0.5"
              :transform="arrowheadTransform(x)" @pointerdown="(e) => startVectorDrag(e, x)")

    div
      h4 Codomain
      svg(class="graph" id="codomain" width="30em" height="30em" viewBox="-10, -10, 20, 20" transform="scale(1,-1)")
        g(stroke="#555" stroke-width="0.1")
            line(x1="0" y1="-10" x2="0" y2="10")
            line(x1="-10" y1="0" x2="10" y2="0")

        g(v-if="slnType != SlnType.None" class="vec col1 xcol")
          line(x1="0" y1="0" :x2="xc1[0]" :y2="xc1[1]")
          g(:transform="vectorTransform(xc1)")
            use(href="#arrowhead" class="arrowhead" x="-0.5" y="-0.5" :transform="arrowheadTransform(xc1)")
        
        g(v-if="slnType != SlnType.None" class="vec col2 xcol")
          line(:x1="xc1[0]" :y1="xc1[1]" :x2="xc1[0]+xc2[0]" :y2="xc1[1]+xc2[1]")
          g(:transform="`translate(${xc1[0]+xc2[0]}, ${xc1[1]+xc2[1]})`")
            use(href="#arrowhead" class="arrowhead" x="-0.5" y="-0.5" :transform="arrowheadTransform(xc2)")

        g(class="vec col1")
          line(x1="0" y1="0" :x2="c1[0]" :y2="c1[1]")
          g(:transform="vectorTransform(c1)")
            g(:transform="labelTransform(c1)")
              text(class="vlabel") c1
            use(href="#arrowhead" class="arrowhead" x="-0.5" y="-0.5"
              :transform="arrowheadTransform(c1)" @pointerdown="(e) => startVectorDrag(e, c1)")
        
        g(class="vec col2")
          line(x1="0" y1="0" :x2="c2[0]" :y2="c2[1]")
          g(:transform="vectorTransform(c2)")
            g(:transform="labelTransform(c2)")
              text(class="vlabel") c2
            use(href="#arrowhead" class="arrowhead" x="-0.5" y="-0.5"
              :transform="arrowheadTransform(c2)"  @pointerdown="(e) => startVectorDrag(e, c2)")

        g(class="vec b")
          line(x1="0" y1="0" :x2="b[0]" :y2="b[1]")
          g(:transform="vectorTransform(b)")
            g(:transform="labelTransform(b)")
              text(class="vlabel") b
            use(href="#arrowhead" class="arrowhead" x="-0.5" y="-0.5"
              :transform="arrowheadTransform(b)" @pointerdown="(e) => startVectorDrag(e, b, true)")
        
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

.vlabel
  stroke none
  font-size 0.04em
  transform scaleY(-1)
  text-anchor middle
  dominant-baseline middle

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
  width 2.4em
  padding 0.5em 0.2em
  white-space nowrap
  overflow hidden
  text-overflow clip
  margin-right 0.4em

  &:focus
    outline none

table
  margin 0 0 0 0.1em
  
.vtable
  margin-right -0.8em
</style>
