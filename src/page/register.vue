<script setup lang="ts">
import { ref } from 'vue';
import { registerUser } from '../condition/auth';

const email = ref('');
const password = ref('');
const displayName = ref('');
const error = ref('');

const handleRegister = async () => {
  error.value = '';
  try {
    await registerUser(email.value, password.value, displayName.value);
    console.log('Successfully registered!');
  } catch (err: any) {
    error.value = err.message;
  }
};
</script>

<template>
  <div class="min-h-screen grid grid-cols-1 md:grid-cols-2">
    <!-- Left Side (Register Form) -->
    <div class="flex items-center justify-center bg-white p-8">
      <div class="w-full max-w-md space-y-6">
        <h2 class="text-3xl font-bold text-center text-gray-800">Create Account</h2>

        <div class="space-y-4">
          <input
            v-model="displayName"
            type="text"
            placeholder="Username"
            class="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-emerald-500 focus:outline-none"
          />
          <input
            v-model="email"
            type="email"
            placeholder="Email"
            class="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-emerald-500 focus:outline-none"
          />
          <input
            v-model="password"
            type="password"
            placeholder="Password"
            class="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-emerald-500 focus:outline-none"
          />
          <button
            @click="handleRegister"
            class="w-full bg-emerald-500 hover:bg-emerald-600 text-white py-2 rounded-md transition"
          >
            Register
          </button>
          <p v-if="error" class="text-center text-red-500 text-sm">{{ error }}</p>
        </div>
      </div>
    </div>

    <!-- Right Side -->
    <div class="bg-gradient-to-br from-emerald-400 to-cyan-500 text-white flex flex-col justify-center items-center p-8">
      <h1 class="text-4xl font-bold mb-4">Join Us Today 🚀</h1>
      <p class="text-lg text-center max-w-sm">Create your account and unlock access to amazing features.</p>
      <img src="https://source.unsplash.com/400x300/?startup" alt="Register Visual" class="mt-10 rounded-lg shadow-lg" />
    </div>
  </div>
</template>
