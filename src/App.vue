<script setup>
import { ref } from 'vue'

import PspdfkitContainer from './components/PspdfkitContainer.vue'

const containerIsLoaded = ref(false)
const onLoaded = () => containerIsLoaded.value = true
</script>

<template>
  <div style="font-size: 20px;">PSPDFKit container is below. If you scroll down before its loaded, then 'Y' coordinates
    are
    messed up.
    <br> If you scroll only once the container is loaded, then the 'Y' coordinates are correct. <br>Container is: <span
      style="font-size: 32px; font-weight: bold; color: red">{{ containerIsLoaded ? 'loaded' : 'loading' }}</span>
  </div>
  <div style="margin-bottom: 500px"></div>
  <PspdfkitContainer @loaded="onLoaded" />
</template>
