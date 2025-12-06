<script setup>
import { ref, onUnmounted, onMounted, nextTick } from 'vue'
import { gsap } from 'gsap'

const isDeleting = ref(false)
const countdown = ref(10)
const timerInterval = ref(null)
const buttonRef = ref(null)
const contentRef = ref(null)
const undoIconRef = ref(null)
const timerBadgeRef = ref(null)
const numberRef = ref(null)
const deleteTextRef = ref(null)
const cancelTextRef = ref(null)

const startDeletion = async () => {
  if (isDeleting.value) return
  
  const initialWidth = buttonRef.value.offsetWidth
  const initialHeight = buttonRef.value.offsetHeight
  
  gsap.set(buttonRef.value, { height: initialHeight })
  
  isDeleting.value = true
  countdown.value = 10
  
  await nextTick()
  
  const finalWidth = buttonRef.value.scrollWidth
  gsap.set(buttonRef.value, { width: initialWidth })
  
  const tl = gsap.timeline()
  
  // Animate text slide and button expansion simultaneously
  if (deleteTextRef.value && cancelTextRef.value) {
    gsap.set(deleteTextRef.value, { y: 0, opacity: 1, scale: 1 })
    gsap.set(cancelTextRef.value, { y: 30, opacity: 0, scale: 0.9 })
    
    tl.to(deleteTextRef.value, {
      y: -30,
      opacity: 0,
      scale: 0.9,
      duration: 0.5,
      ease: 'power2.in'
    }, 0)
    
    tl.to(cancelTextRef.value, {
      y: 0,
      opacity: 1,
      scale: 1,
      color: '#dc2626',
      duration: 0.5,
      ease: 'power2.out'
    }, 0)
  }
  
  tl.to(buttonRef.value, {
    width: finalWidth,
    backgroundColor: '#fce7f3',
    height: initialHeight,
    duration: 0.5,
    ease: 'power2.out'
  }, 0)
  
  // Fade in undo icon and timer badge
  if (undoIconRef.value) {
    gsap.set(undoIconRef.value, { scale: 0, opacity: 0 })
    tl.to(undoIconRef.value, {
      scale: 1,
      opacity: 1,
      duration: 0.2,
      ease: 'back.out(1.7)'
    }, 0.1)
  }
  
  if (timerBadgeRef.value) {
    gsap.set(timerBadgeRef.value, { scale: 0, opacity: 0 })
    tl.to(timerBadgeRef.value, {
      scale: 1,
      opacity: 1,
      duration: 0.2,
      ease: 'back.out(1.7)'
    }, 0.1)
  }
  
  // Start countdown
  setTimeout(() => {
    timerInterval.value = setInterval(() => {
      countdown.value--
      
      if (numberRef.value) {
        gsap.fromTo(numberRef.value,
          { y: -30, opacity: 0 },
          { 
            y: 0, 
            opacity: 1, 
            duration: 0.4, 
            ease: 'back.out(1.2)'
          }
        )
      }
      
      if (countdown.value <= 0) {
        confirmDeletion()
      }
    }, 1000)
  }, 500)
}

const cancelDeletion = () => {
  if (!isDeleting.value) return
  
  clearInterval(timerInterval.value)
  timerInterval.value = null
  
  // Get current width and height
  const currentWidth = buttonRef.value.offsetWidth
  const currentHeight = buttonRef.value.offsetHeight
  
  // Calculate exact target width by creating a temporary invisible element to measure
  // This avoids changing the state which causes visual glitch
  const tempSpan = document.createElement('span')
  tempSpan.textContent = 'Delete Account'
  tempSpan.style.position = 'absolute'
  tempSpan.style.visibility = 'hidden'
  tempSpan.style.whiteSpace = 'nowrap'
  tempSpan.style.fontSize = window.getComputedStyle(buttonRef.value).fontSize
  tempSpan.style.fontFamily = window.getComputedStyle(buttonRef.value).fontFamily
  tempSpan.style.fontWeight = window.getComputedStyle(buttonRef.value).fontWeight
  tempSpan.style.padding = '0'
  document.body.appendChild(tempSpan)
  
  const textWidth = tempSpan.offsetWidth
  document.body.removeChild(tempSpan)
  
  // Calculate target width: text width + padding (px-6 = 1.5rem = 24px each side = 48px total)
  // Also account for the text container min-width (130px) + padding
  const calculatedWidth = textWidth + 48
  const minButtonWidth = 130 + 48 // min-w-[130px] for text + padding
  const targetWidth = Math.max(calculatedWidth, minButtonWidth)
  
  // Set initial width to prevent overflow during animation
  gsap.set(buttonRef.value, { width: currentWidth })
  
  // Create timeline for reverse animation - perfect inverse of startDeletion
  const tl = gsap.timeline({
    onComplete: () => {
      isDeleting.value = false
      countdown.value = 10
      // Reset width and height to auto after animation
      gsap.set(buttonRef.value, { width: 'auto', height: 'auto' })
    }
  })
  
  // All animations start simultaneously for smooth transition
  // Step 1: Fade out undo icon and timer badge quickly
  if (undoIconRef.value) {
    tl.to(undoIconRef.value, {
      opacity: 0,
      scale: 0,
      duration: 0.3,
      ease: 'power2.in'
    }, 0)
  }
  
  if (timerBadgeRef.value) {
    tl.to(timerBadgeRef.value, {
      opacity: 0,
      scale: 0,
      duration: 0.3,
      ease: 'power2.in'
    }, 0)
  }
  
  // Step 2: Animate text slide DOWN simultaneously
  if (cancelTextRef.value && deleteTextRef.value) {
    // Set initial states
    gsap.set(cancelTextRef.value, { y: 0, opacity: 1, scale: 1 })
    gsap.set(deleteTextRef.value, { y: -30, opacity: 0, scale: 0.9 })
    
    // Animate both texts - start at the same time
    tl.to(cancelTextRef.value, {
      y: 30,
      opacity: 0,
      scale: 0.9,
      duration: 0.5,
      ease: 'power2.in'
    }, 0)
    
    tl.to(deleteTextRef.value, {
      y: 0,
      opacity: 1,
      scale: 1,
      color: '#ffffff',
      duration: 0.5,
      ease: 'power2.out'
    }, 0)
  }
  
  // Step 3: Shrink width simultaneously
  tl.to(buttonRef.value, {
    width: targetWidth,
    height: currentHeight,
    backgroundColor: '#dc2626',
    duration: 0.5,
    ease: 'power2.out'
  }, 0)
}

const confirmDeletion = () => {
  clearInterval(timerInterval.value)
  timerInterval.value = null
  
  // Here you would call your actual delete account API
  console.log('Account deletion confirmed')
  
  // Get current width and height
  const currentWidth = buttonRef.value.offsetWidth
  const currentHeight = buttonRef.value.offsetHeight
  
  // Calculate exact target width by creating a temporary invisible element to measure
  const tempSpan = document.createElement('span')
  tempSpan.textContent = 'Delete Account'
  tempSpan.style.position = 'absolute'
  tempSpan.style.visibility = 'hidden'
  tempSpan.style.whiteSpace = 'nowrap'
  tempSpan.style.fontSize = window.getComputedStyle(buttonRef.value).fontSize
  tempSpan.style.fontFamily = window.getComputedStyle(buttonRef.value).fontFamily
  tempSpan.style.fontWeight = window.getComputedStyle(buttonRef.value).fontWeight
  tempSpan.style.padding = '0'
  document.body.appendChild(tempSpan)
  
  const textWidth = tempSpan.offsetWidth
  document.body.removeChild(tempSpan)
  
  // Calculate target width: text width + padding (px-6 = 1.5rem = 24px each side = 48px total)
  // Also account for the text container min-width (130px) + padding
  const calculatedWidth = textWidth + 48
  const minButtonWidth = 130 + 48 // min-w-[130px] for text + padding
  const targetWidth = Math.max(calculatedWidth, minButtonWidth)
  
  // Set initial width to prevent overflow during animation
  gsap.set(buttonRef.value, { width: currentWidth })
  
  // Create timeline for reverse animation - same as cancelDeletion
  const tl = gsap.timeline({
    onComplete: () => {
      isDeleting.value = false
      countdown.value = 10
      gsap.set(buttonRef.value, { width: 'auto', height: 'auto' })
    }
  })
  
  // All animations start simultaneously for smooth transition
  // Step 1: Fade out undo icon and timer badge quickly
  if (undoIconRef.value) {
    tl.to(undoIconRef.value, {
      opacity: 0,
      scale: 0,
      duration: 0.3,
      ease: 'power2.in'
    }, 0)
  }
  
  if (timerBadgeRef.value) {
    tl.to(timerBadgeRef.value, {
      opacity: 0,
      scale: 0,
      duration: 0.3,
      ease: 'power2.in'
    }, 0)
  }
  
  // Step 2: Animate text slide DOWN simultaneously
  if (cancelTextRef.value && deleteTextRef.value) {
    // Set initial states
    gsap.set(cancelTextRef.value, { y: 0, opacity: 1, scale: 1 })
    gsap.set(deleteTextRef.value, { y: -30, opacity: 0, scale: 0.9 })
    
    // Animate both texts - start at the same time
    tl.to(cancelTextRef.value, {
      y: 30,
      opacity: 0,
      scale: 0.9,
      duration: 0.5,
      ease: 'power2.in'
    }, 0)
    
    tl.to(deleteTextRef.value, {
      y: 0,
      opacity: 1,
      scale: 1,
      color: '#ffffff',
      duration: 0.5,
      ease: 'power2.out'
    }, 0)
  }
  
  // Step 3: Shrink width simultaneously
  tl.to(buttonRef.value, {
    width: targetWidth,
    height: currentHeight,
    backgroundColor: '#dc2626',
    duration: 0.5,
    ease: 'power2.out'
  }, 0)
}

onMounted(() => {
  // Initialize text positions
  if (deleteTextRef.value && cancelTextRef.value) {
    gsap.set(deleteTextRef.value, { y: 0, opacity: 1, scale: 1, color: '#ffffff' })
    gsap.set(cancelTextRef.value, { y: 30, opacity: 0, scale: 0.9 })
  }
})

onUnmounted(() => {
  if (timerInterval.value) {
    clearInterval(timerInterval.value)
  }
})
</script>

<template>
  <div class="flex items-center justify-center">
    <button
      ref="buttonRef"
      @click="isDeleting ? cancelDeletion() : startDeletion()"
      class="relative flex items-center justify-center px-6 py-4 rounded-full font-medium text-base active:scale-95 overflow-hidden"
      :class="isDeleting ? 'bg-pink-100' : 'bg-red-600'"
      style="will-change: width, height;"
    >
      <!-- Content wrapper that holds everything together -->
      <div ref="contentRef" class="flex items-center gap-3">
        <!-- Undo icon (only visible when deleting) -->
        <div
          v-if="isDeleting"
          ref="undoIconRef"
          class="flex items-center justify-center w-6 h-6 bg-red-600 rounded-full flex-shrink-0"
        >
          <svg
            class="w-4 h-4 text-white"
            fill="currentColor"
            viewBox="0 0 640 640"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path d="M320 128C263.2 128 212.1 152.7 176.9 192L224 192C241.7 192 256 206.3 256 224C256 241.7 241.7 256 224 256L96 256C78.3 256 64 241.7 64 224L64 96C64 78.3 78.3 64 96 64C113.7 64 128 78.3 128 96L128 150.7C174.9 97.6 243.5 64 320 64C461.4 64 576 178.6 576 320C576 461.4 461.4 576 320 576C233 576 156.1 532.6 109.9 466.3C99.8 451.8 103.3 431.9 117.8 421.7C132.3 411.5 152.2 415.1 162.4 429.6C197.2 479.4 254.8 511.9 320 511.9C426 511.9 512 425.9 512 319.9C512 213.9 426 128 320 128z"/>
          </svg>
        </div>

        <div class="relative h-6 overflow-hidden flex items-center">
          <span 
            ref="deleteTextRef"
            class="whitespace-nowrap"
            :class="isDeleting ? 'absolute left-0' : ''"
          >
            Delete Account
          </span>
          <span 
            ref="cancelTextRef"
            class="whitespace-nowrap"
            :class="isDeleting ? '' : 'absolute left-0'"
          >
            Cancel Deletion
          </span>
        </div>

        <div
          v-if="isDeleting"
          ref="timerBadgeRef"
          class="flex items-center justify-center min-w-[2.5rem] h-6 px-2.5 bg-red-600 rounded-full text-white font-bold text-xs overflow-hidden flex-shrink-0"
        >
          <span ref="numberRef" class="inline-block tabular-nums">
            {{ countdown }}
          </span>
        </div>
      </div>
    </button>
  </div>
</template>

<style scoped>
button {
  font-family: 'Google Sans', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}
</style>
