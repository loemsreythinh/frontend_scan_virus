<script setup>
import ScannerView from './components/ScannerView.vue';
import './assets/global.css';
</script>

<template>
  <main>
    <ScannerView />
  </main>
</template>

<style>
/* 1. Global Reset to kill all margins, paddings, and max-widths */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body, #app {
  width: 100% !important;
  max-width: 100% !important;
  min-height: 100vh;
  margin: 0 !important;
  padding: 0 !important;
  background-color: #030805 !important;
}
</style>