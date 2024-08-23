<script lang="ts">
import {defineEmits} from 'vue'
import '@interactjs/auto-start'
import '@interactjs/actions/drag'
import '@interactjs/actions/resize'
import '@interactjs/modifiers'
import '@interactjs/dev-tools'

// es6 import
// import '@interactjs/interact' 
import interact from 'interactjs'

// let emit = defineEmits(['drag'])
// console.log(typeof(emit))
const position = { x: 0, y: 0 }



export default {
  emits: ['drag'],
  setup(props, context) {
    interact('.draggable').draggable({
      listeners: {
        start (event: any) {
          console.log(event.type, event.target)
        },
        move (event: any) {
          
          context.emit('drag', event.dx, event.dy)
          // position.x += event.dx
          // position.y += event.dy

          // event.target.style.transform =
          //   `translate(${position.x}px, ${position.y}px)`
        },
      }
    })
  }
}
// export default {
//   setup(props, context) {
//       context.emit('drag');
//   }
// }


</script>

<template>
  <canvas class="draggable" id="canvas"></canvas>
</template>

<style scoped>
</style>