<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'

const v_x = ref<SVGGElement|null>(null)

const A = reactive([[1,0],[0,1]]) // by rows
const x = reactive([1,1])
const b = reactive([1,1])

const hp1 = reactive([0,0]) // y values at x = -10, 10
const hp2 = reactive([0,0]) // y values at x = -10, 10

// am I even using vue here?
// can I bind these vector values automatically?
// 

function compute_b() {
  b[0] = A[0][0]*x[0] + A[0][1]*x[1]
  b[1] = A[1][0]*x[0] + A[1][1]*x[1]
}

function compute_x() {
  const inv_det = 1 / (A[0][0]*A[1][1] - A[1][0]*A[0][1])
  x[0] = inv_det * (A[1][1] - A[0][1])
  x[1] = inv_det * (A[0][0] - A[1][0])
}

// watch A, x -> compute b
// watch b -> compute x
// computed object with lines, dependent on A, b
// watch A, x, b -> update arrowheads


const _deg = -180/Math.PI
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
  updateVector(v_x.value!, 0, 0, -3, -1.5)
})
</script>


<!-- sqrt(3) = 1.732 -->
<template lang="pug">
div(id="matrix_mult")
  div(id="graphs")
    svg(class="graph" id="domain" width="30em" height="30em" viewBox="-10 -10 20 20")
      g(stroke="#555" stroke-width="0.1")
        line(x1="0" y1="-10" x2="0" y2="10")
        line(x1="-10" y1="0" x2="10" y2="0")
      symbol(id="arrowhead" width="0.6" height="0.6" viewBox="-1 0 2 2")
        polygon(points="-1,0 1,0 0,1.732")
      
      g(id="hp1" class="hp col1")
        line(x1="-6" y1="-10" x2="10" y2="6")
      
      g(id="hp2" class="hp col2")
        line(x1="10" y1="-6" x2="-6" y2="10")
      
      g(id="col1" class="vec col1")
        line(x1="0" y1="0" x2="5" y2="-5")
        use(href="#arrowhead" class="arrowhead" x="4.7" y="-5.3" transform="rotate(-135, 5, -5)")
      
      g(id="col2" class="vec col2")
        line(x1="0" y1="0" x2="3" y2="3")
        use(href="#arrowhead" class="arrowhead" x="2.7" y="2.7" transform="rotate(-45, 3, 3)")
      
      g(ref="v_x" class="vec x")
        line(x1="0" y1="0" x2="4" y2="0")
        use(href="#arrowhead" class="arrowhead" x="3.7" y="-0.3" transform="rotate(-90, 4, 0)")


    svg(class="graph" id="codomain" width="30em" height="30em" viewBox="-10, -10, 20, 20")
      g(stroke="#555" stroke-width="0.1")
          line(x1="0" y1="-10" x2="0" y2="10")
          line(x1="-10" y1="0" x2="10" y2="0")

      g(id="row1" class="vec row1")
        line(x1="0" y1="0" x2="1" y2="-1.732")
        use(href="#arrowhead" class="arrowhead" x="0.7" y="-2.32" transform="rotate(-150, 1, -1.732)")
      
      g(id="row2" class="vec row2")
        line(x1="1" y1="-1.732" x2="6" y2="0")
        use(href="#arrowhead" class="arrowhead" x="5.7" y="-0.3" transform="rotate(-70, 6, 0)")

      g(id="b" class="vec b")
        line(x1="0" y1="0" x2="6" y2="0")
        use(href="#arrowhead" class="arrowhead" x="5.7" y="-0.3" transform="rotate(-90, 6, 0)")
      

  div(id="equation") equation
</template>


<style scoped lang="stylus">
.graph
  border 2px solid black
  margin 2em

.arrowhead
  &:hover
    stroke-width 0.6

.hp
  stroke-width 0.08
  opacity 0.5

.vec
  stroke-width 0.1

.col1
  fill #FF0
  stroke #FF0

.col2
  fill #F80
  stroke #F80

.x
  fill #F00
  stroke #F00

.row1
  fill #0F4
  stroke #0F4

.row2
  fill #0DD
  stroke #0DD

.b
  fill #80F
  stroke #80F
</style>
