<script setup>
import SocialFooter from './components/SocialFooter.vue'
import DeleteAccountButton from './components/DeleteAccountButton.vue'
</script>

<template>
  <div class="min-h-screen bg-white flex flex-col items-center p-4">
    <div class="flex-1 flex items-center justify-center">
      <DeleteAccountButton />
    </div>
    
    <div class="py-8">
      <SocialFooter />
    </div>
  </div>
</template>