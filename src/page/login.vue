<script setup lang="ts">
import { ref } from 'vue';
import { loginUser } from '../condition/auth';

const email = ref('');
const password = ref('');
const error = ref('');

const handleLogin = async () => {
  error.value = '';
  try {
    await loginUser(email.value, password.value);
    console.log('Successfully logged in!');
  } catch (err: any) {
    error.value = err.message;
  }
};
</script>

<template>
  <div class="min-h-screen grid grid-cols-1 md:grid-cols-2">
    <!-- Left Side -->
    <div class="bg-gradient-to-br from-indigo-600 to-purple-600 text-white flex flex-col justify-center items-center p-8">
      <h1 class="text-4xl font-bold mb-4">Welcome Back!</h1>
      <p class="text-lg text-center max-w-sm">Login to access your dashboard and continue your journey.</p>
      <img src="https://source.unsplash.com/400x300/?technology" alt="Login Visual" class="mt-10 rounded-lg shadow-lg" />
    </div>

    <!-- Right Side (Login Form) -->
    <div class="flex items-center justify-center bg-white p-8">
      <div class="w-full max-w-md space-y-6">
        <h2 class="text-3xl font-bold text-gray-800 text-center">Login</h2>

        <div class="space-y-4">
          <input
            v-model="email"
            type="email"
            placeholder="Email"
            class="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-indigo-500 focus:outline-none"
          />
          <input
            v-model="password"
            type="password"
            placeholder="Password"
            class="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-indigo-500 focus:outline-none"
          />
          <button
            @click="handleLogin"
            class="w-full bg-indigo-600 hover:bg-indigo-700 text-white py-2 rounded-md transition"
          >
            Sign In
          </button>
          <p v-if="error" class="text-center text-red-500 text-sm">{{ error }}</p>
        </div>
      </div>
    </div>
  </div>
</template>
