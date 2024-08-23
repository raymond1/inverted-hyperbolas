<script setup lang="ts">

const emit = defineEmits(['move', 'changeMagnification'])

function handleLeftClick(){
  emit('move', 'left')
}

function handleRightClick(){
  emit('move', 'right')
}

function handleUpClick(){
  emit('move', 'up')
}

function handleDownClick(){
  emit('move', 'down')
}

function handleIncreaseMagnification(){
  emit('changeMagnification', 'increase')
}

function handleDecreaseMagnification(){
  emit('changeMagnification', 'decrease')
}


</script>

<template>
  <div class="grid">
    <div class="item filler2"></div><div class="item up" v-on:click="handleUpClick"></div><div class="item filler"></div><div class="item filler"></div><div class="item increaseMagnification" v-on:click="handleIncreaseMagnification"></div>
    <div class="item left" v-on:click="handleLeftClick"></div><div class="item filler"></div><div class="item right" v-on:click="handleRightClick"></div>
    <div class="item filler2"></div><div class="item down" v-on:click="handleDownClick"></div><div class="item filler"></div><div class="decreaseMagnification" v-on:click="handleDecreaseMagnification"></div>
  </div>
</template>

<style scoped>
.grid {
  display: grid;
  grid-template-columns: 5;
  grid-template-rows: 3;
  width: 100px;
  height:60px;
}

.grid .item{
  width: 20px;
  height: 20px;
}

.increaseMagnification{
  background: url("../assets/increase magnification.png");
  background-size: cover;
  grid-column-start: 5;
  grid-column-end: 5;
  grid-row-start: 1;
  grid-row-end: 1;
}

.decreaseMagnification{
  background: url("../assets/decrease magnification.png");
  background-size: cover;

  grid-column-start: 5;
  grid-column-end: 5;

  grid-row-start: 3;
  grid-row-end: 3;
}

.left {
  background: url("../assets/left.png");
  background-size: cover;
  width: 20px;
  height:20px;
  border: 1px red solid;
  grid-column-start:1;
  grid-column-end:1;
}

.right {
  background: url("../assets/left.png");
  background-size: cover;
  width: 20px;
  height:20px;
  border: 1px red solid;
  transform: rotate(180deg);
  grid-column-start:3;
  grid-column-end:3;
}

.up {
  background: url("../assets/left.png");
  background-size: cover;
  width: 20px;
  height:20px;
  border: 1px red solid;
  transform: rotate(90deg);
}

.down {
  background: url("../assets/left.png");
  background-size: cover;
  width: 20px;
  height:20px;
  border: 1px red solid;
  transform: rotate(270deg);
  grid-column-start:2;
  grid-column-end:2;
  grid-row-start:3;
  grid-row-end:3;
}

</style>
