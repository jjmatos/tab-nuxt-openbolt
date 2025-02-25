<template>
<div class="min-h-screen bg-gray-900 text-gray-100">
  <div class="bg-gray-800 shadow-lg px-6 py-4">
    <div class="container mx-auto flex items-center justify-between">
      <h1 class="text-2xl font-bold text-white">My App</h1>
      <nav class="space-x-6">
        <NuxtLink 
          to="/" 
          class="text-gray-300 hover:text-white transition-colors"
          active-class="text-white font-semibold">
          Home
        </NuxtLink>
        <NuxtLink 
          to="/Ben" 
          class="text-gray-300 hover:text-white transition-colors"
          active-class="text-white font-semibold">
          About
        </NuxtLink>
        <NuxtLink 
          to="/Shop" 
          class="text-gray-300 hover:text-white transition-colors"
          active-class="text-white font-semibold">
          Shop
        </NuxtLink>
      </nav>
    </div>
  </div>
  <div class="container mx-auto py-6 px-4">
    <NuxtPage/>
  </div>
</div>
</template>
