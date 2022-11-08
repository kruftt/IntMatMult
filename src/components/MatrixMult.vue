<script setup lang="ts">
import {
  computed,
  ref,
  reactive,
  onMounted
} from 'vue'

const v_x = ref<SVGGElement|null>(null)

const A = reactive([[3,1],[-1,2]]) // by rows
const x = reactive([2,2])
const b = reactive([8,2])

// const hp1 = reactive([0,0]) // y values at x = -10, 10
// const hp2 = reactive([0,0]) // y values at x = -10, 10
// const hp1 = computed(() => [b[0]/A[0][0], b[0]/A[0][1]])
// const hp2 = computed(() => [b[1]/A[1][0], b[1]/A[1][1]])

// remember that line is perpendicular to the vector!
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
 
  // const m = -y_int/x_int
  // console.log('GLC: ', x_int, y_int, m)
  // const y1 = -m*(10+x_int) // x = -10
  // const y2 = m*(10-x_int) // x = 10
  // return [y1, y2]
}

const hp1 = computed(() => {
  return getLineCoords(A[0][0], A[0][1], b[0])
})

const hp2 = computed(() => {
  return getLineCoords(A[1][0], A[1][1], b[1])
})

const xcol1 = computed(() => [x[0]*A[0][0], x[0]*A[1][0]])
const xcol2 = computed(() => [x[1]*A[0][1], x[1]*A[1][1]])

// g(class="vec col2 xcol")
//   line(:x1="xcol1[0]" :y1="xcol1[1]" :x2="b[0]" :y2="b[1]")
//   use(href="#arrowhead" class="arrowhead" :x="b[0]-0.4" :y="b[1]-0.4" :transform="vTransform(A[0][1], A[1][1], b[0], b[1])")

// const scol = computed(() => [xcol1[0]+xcol])


const _deg = 180/Math.PI
const vTransform = (x: number, y: number, cx=x, cy=y) => `rotate(${_deg * Math.atan2(y, x) - 90}, ${cx}, ${cy})`


function compute_b() {
  b[0] = A[0][0]*x[0] + A[0][1]*x[1]
  b[1] = A[1][0]*x[0] + A[1][1]*x[1]
}

function compute_x() {
  const inv_det = 1 / (A[0][0]*A[1][1] - A[1][0]*A[0][1])
  x[0] = inv_det * (A[1][1]*b[0] - A[0][1]*b[1])
  x[1] = inv_det * (A[0][0]*b[1] - A[1][0]*b[0])
}

// A, x -> compute b
// b -> compute x
// computed object with lines, dependent on A, b
// A, x, b -> update arrowheads

function updateVector(e: SVGGElement, x1: number, y1: number, x2: number, y2: number) {
  const l = e.querySelector('line')
  l!.setAttribute('x1', `${x1}`)
  l!.setAttribute('y1', `${-y1}`)
  l!.setAttribute('x2', `${x2}`)
  l!.setAttribute('y2', `${-y2}`)

  const u = e.querySelector('use')
  u!.setAttribute('x', `${x2 - 0.3}`)
  u!.setAttribute('y', `${-y2 - 0.3}`)

  const angle = _deg * Math.atan2(y2, x2) - 90
  u!.setAttribute('transform', `rotate(${angle}, ${x2}, ${-y2})`)
}

// y = x-intercept
// x = y-intercept
function updateLine(e: SVGGElement, y: number, x: number) {
  const m = -y/x
  const y1 = -m*(10+x) // x = -10
  const y2 = m*(10-x) // x = 10
  const l = e.querySelector('line')
}

onMounted(() => {
  // xAxis.value!.setAttribute("stroke", "#f0f")
  // updateVector(v_x.value!, 0, 0, -3, -1.5)
})
</script>


<!-- sqrt(3) = 1.732 -->
<template lang="pug">
div(id="matrix_mult")
  div(id="graphs")
    div
      h4 Domain
      svg(class="graph" id="domain" width="30em" height="30em" viewBox="-10 -10 20 20" transform="scale(1,-1)")
        g(stroke="#555" stroke-width="0.1")
          line(x1="0" y1="-10" x2="0" y2="10")
          line(x1="-10" y1="0" x2="10" y2="0")
        symbol(id="arrowhead" width="0.8" height="0.8" viewBox="-1 -1 2 2")
          polygon(points="-0.3,-1 0.3,-1 0,0")
        
        g(id="hp1" class="hp row1")
          line(:x1="hp1[0]" :y1="hp1[1]" :x2="hp1[2]" :y2="hp1[3]")
          //- line(x1="-6" y1="-10" x2="10" y2="6")
        
        g(id="hp2" class="hp row2")
          line(:x1="hp2[0]" :y1="hp2[1]" :x2="hp2[2]" :y2="hp2[3]")
          //- line(x1="10" y1="-6" x2="-6" y2="10")
        
        g(id="row1" class="vec row1")
          line(x1="0" y1="0" :x2="A[0][0]" :y2="A[0][1]")
          use(href="#arrowhead" class="arrowhead" :x="A[0][0]-0.4" :y="A[0][1]-0.4" :transform="vTransform(A[0][0], A[0][1])")
        
        g(id="row2" class="vec row2")
          line(x1="0" y1="0" :x2="A[1][0]" :y2="A[1][1]")
          use(href="#arrowhead" class="arrowhead" :x="A[1][0]-0.4" :y="A[1][1]-0.4" :transform="vTransform(A[1][0], A[1][1])")
        
        g(ref="v_x" class="vec x")
          line(x1="0" y1="0" :x2="x[0]" :y2="x[1]")
          use(href="#arrowhead" class="arrowhead" :x="x[0]-0.4" :y="x[1]-0.4" :transform="vTransform(x[0], x[1])")

    div
      h4 Codomain
      svg(class="graph" id="codomain" width="30em" height="30em" viewBox="-10, -10, 20, 20" transform="scale(1,-1)")
        g(stroke="#555" stroke-width="0.1")
            line(x1="0" y1="-10" x2="0" y2="10")
            line(x1="-10" y1="0" x2="10" y2="0")

        g(class="vec col1 xcol")
          line(x1="0" y1="0" :x2="xcol1[0]" :y2="xcol1[1]")
          use(href="#arrowhead" class="arrowhead" :x="xcol1[0]-0.4" :y="xcol1[1]-0.4" :transform="vTransform(xcol1[0], xcol1[1])")
        
        g(class="vec col2 xcol")
          line(:x1="xcol1[0]" :y1="xcol1[1]" :x2="xcol1[0]+xcol2[0]" :y2="xcol1[1]+xcol2[1]")
          use(href="#arrowhead" class="arrowhead" :x="b[0]-0.4" :y="b[1]-0.4" :transform="vTransform(xcol2[0], xcol2[1], b[0], b[1])")

        g(id="col1" class="vec col1")
          line(x1="0" y1="0" :x2="A[0][0]" :y2="A[1][0]")
          use(href="#arrowhead" class="arrowhead" :x="A[0][0]-0.4" :y="A[1][0]-0.4" :transform="vTransform(A[0][0], A[1][0])")
        
        g(id="col2" class="vec col2")
          line(x1="0" y1="0" :x2="A[0][1]" :y2="A[1][1]")
          use(href="#arrowhead" class="arrowhead" :x="A[0][1]-0.4" :y="A[1][1]-0.4" :transform="vTransform(A[0][1], A[1][1])")

        g(id="b" class="vec b")
          line(x1="0" y1="0" :x2="b[0]" :y2="b[1]")
          use(href="#arrowhead" class="arrowhead" :x="b[0]-0.4" :y="b[1]-0.4" :transform="vTransform(b[0], b[1])")
        

  div(id="equation")
    div [
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
    div ][
    table
      tr
        td
          input(type="number" class="num_input" v-model="x[0]" @input="compute_b" step="0.1")
      tr
        td
          input(type="number" class="num_input" v-model="x[1]" @input="compute_b" step="0.1")
    div ] = [
    table
      tr
        td
          input(type="number" class="num_input" v-model="b[0]" @input="compute_x" step="0.1")
      tr
        td
          input(type="number" class="num_input" v-model="b[1]" @input="compute_x" step="0.1")
    div ]
</template>


<style scoped lang="stylus">
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

  font-size 7em
  color #666
  // text-align center

.num_input
  border none
  background-color #0000
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
</style>
