<template>
  <header class="w-full p-4 h-16 absolute top-0 left-0 background-hero">
    <nav class="flex justify-between items-center h-full">
      <div class="">
        <img :src="Logo" alt="Logo" width="220" />
      </div>
      <ul class="flex gap-4">
        <li>
          <a href="#" class="text-white">
            <Bars3Icon class="h-6 w-6 text-white" />
          </a>
        </li>
      </ul>
    </nav>
  </header>
</template>

<script setup>
import { Bars3Icon } from '@heroicons/vue/24/outline'
import Logo from '@/assets/images/Bytetravel_yt.svg'
</script>