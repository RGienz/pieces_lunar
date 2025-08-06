<!-- login.vue -->
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
    console.log('Logged in!');
  } catch (err: any) {
    error.value = err.message;
  }
};
</script>

<template>
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
</template>
