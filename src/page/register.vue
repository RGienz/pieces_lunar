<!-- register.vue -->
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
    console.log('Registered!');
  } catch (err: any) {
    error.value = err.message;
  }
};
</script>

<template>
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
</template>
