<!-- auth.vue -->
<script setup lang="ts">
import { ref } from 'vue';
import Login from './Login.vue';
import Register from './Register.vue';

const isLoginPage = ref(true);
</script>

<template>
  <!-- <div class="min-h-screen bg-gradient-to-br from-indigo-500 to-purple-600 flex items-center justify-center px-4"> -->
  <div class="min-h-screen bg-gradient-to-br from-indigo-500  flex items-center justify-center px-4">
    <div class="bg-white rounded-2xl shadow-lg w-full max-w-4xl grid grid-cols-1 md:grid-cols-2 overflow-hidden">
      
      <!-- Left: Form -->
      <div class="p-8">
        <div class="flex justify-between items-center mb-6">
          <h2 class="text-2xl font-bold text-gray-800">
            <!-- {{ isLoginPage ? 'Welcome Back!' : 'Join Us Today 🚀' }} -->
          </h2>
          <span class="text-sm text-gray-500 cursor-pointer hover:underline" @click="isLoginPage = !isLoginPage">
            {{ isLoginPage ? 'Register' : 'Login' }}
          </span>
        </div>

        <component :is="isLoginPage ? Login : Register" />
        
        <p class="text-sm text-center mt-6 text-gray-500">
          {{ isLoginPage ? "Don't have an account?" : "Already have an account?" }}
          <span class="text-indigo-600 cursor-pointer font-medium hover:underline" @click="isLoginPage = !isLoginPage">
            {{ isLoginPage ? "Register" : "Login" }}
          </span>
        </p>
      </div>

      <!-- Right: Visual -->
      <div class="hidden md:flex flex-col items-center justify-center bg-gradient-to-tr from-purple-500 to-indigo-500 text-white p-6">
        <h1 class="text-4xl font-bold mb-4">Luna.</h1>
        <p class="text-center max-w-xs">
          <!-- {{ isLoginPage ? 'Access your dashboard and manage everything in one place.' : 'Create an account to explore new possibilities.' }} -->
          <!-- {{ isLoginPage ? 'Access your dashboard and manage everything in one place.' : 'Create an account to explore new possibilities.' }} -->
        </p>
        <!-- <img
          :src="isLoginPage ? 'https://source.unsplash.com/300x200/?technology' : 'https://source.unsplash.com/300x200/?startup'"
          class="mt-6 rounded-lg shadow-xl w-full max-w-xs"
          alt="Visual"
        /> -->
        <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRQ4Z-p69gfkc8sy1W48QpOvUsIuGQAfWadvg&s" alt="">
      </div>

    </div>
  </div>
</template>
