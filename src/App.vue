<template>
  <div id="app">
    <Wheel :names="defaultNames" />
  </div>
</template>

<script setup lang="ts">
import Wheel from './components/SpinningWheel.vue';

const defaultNames = [
  { name: 'Tim', color: 'Red' },
  { name: 'Pieter', color: 'Green' },
  { name: 'Anne', color: 'Blue' },
  { name: 'Laura', color: 'Yellow' }
];
</script>

<style>

</style>

