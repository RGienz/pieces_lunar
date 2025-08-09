<script setup lang="ts">
// import Chat from './mainpage/mainpage.vue';
import Chat2 from './page/chatPage.vue';
// import OnlineUsers from './components/OnlineUsers.vue';
import { ref, onMounted, onUnmounted, watch } from 'vue'; // Import watch and onUnmounted
import { logoutUser, sendVerificationEmail } from './condition/auth';
import { auth, db } from '@/firebase';
import { doc, setDoc, updateDoc, serverTimestamp } from 'firebase/firestore';
// import { BadgeCheck, TriangleAlert } from 'lucide-vue-next';
import AuthVue from './page/auth.vue';

const loggedInUser = ref(auth.currentUser);
const loading = ref(true);
const isSendingVerification = ref(false);

// Idle timer variables
const IDLE_TIMEOUT = 60 * 60 * 1000; // 1 hour in milliseconds
let idleTimer: number | null = null; // Use let for mutable timer ID

// Function to update user status to offline on explicit logout or idle timeout
const handleLogout = async () => {
  const user = auth.currentUser;
  if (user) {
    const userRef = doc(db, 'online_users', user.uid);
    try {
      await updateDoc(userRef, {
        isOnline: false,
        lastSeen: serverTimestamp(),
      });
    } catch (e) {
      console.error("Error setting user offline status on logout: ", e);
    }
  }
  // Clear the idle timer before logging out
  clearIdleTimer();
  // Call your existing logout function
  logoutUser();
};

// Function to handle sending the verification email
const handleSendVerificationEmail = async () => {
  if (loggedInUser.value && !isSendingVerification.value) {
    isSendingVerification.value = true; // Disable button
    try {
      await sendVerificationEmail(loggedInUser.value);
      alert('Verification email sent! Please check your inbox.'); // Simple feedback
      // You might want to add more sophisticated UI feedback here
    } catch (error) {
      console.error('Failed to send verification email:', error);
      alert('Failed to send verification email. Please try again.'); // Simple error feedback
    } finally {
      isSendingVerification.value = false; // Re-enable button
    }
  }
};

// --- Idle Timer Logic ---

// Function to start the idle timer
const startIdleTimer = () => {
  idleTimer = setTimeout(() => {
    console.log('User idle for 1 hour, logging out...');
    handleLogout(); // Call logout after idle timeout
  }, IDLE_TIMEOUT);
};

// Function to reset the idle timer
const resetIdleTimer = () => {
  clearIdleTimer(); // Clear existing timer
  startIdleTimer(); // Start a new timer
};

// Function to clear the idle timer
const clearIdleTimer = () => {
  if (idleTimer !== null) {
    clearTimeout(idleTimer);
    idleTimer = null;
  }
};

// Activity handler to reset the timer
const activityHandler = () => {
  if (loggedInUser.value) { // Only reset if a user is logged in
    resetIdleTimer();
  }
};

// Add event listeners for user activity
const addActivityListeners = () => {
  window.addEventListener('mousemove', activityHandler);
  window.addEventListener('keydown', activityHandler);
  window.addEventListener('click', activityHandler);
  // Add more events if needed (e.g., scroll, touchstart)
};

// Remove event listeners
const removeActivityListeners = () => {
  window.removeEventListener('mousemove', activityHandler);
  window.removeEventListener('keydown', activityHandler);
  window.removeEventListener('click', activityHandler);
};

// Watch for changes in loggedInUser to start/stop the timer and listeners
watch(loggedInUser, (newUser, oldUser) => {
  if (newUser) {
    // User logged in
    addActivityListeners();
    startIdleTimer();
  } else {
    // User logged out
    removeActivityListeners();
    clearIdleTimer();
  }
});


onMounted(() => {
  auth.onAuthStateChanged(async (user: any) => {
    loggedInUser.value = user;
    loading.value = false; // Set loading to false once the auth state is determined

    // When user logs in, set their status to online
    if (user) {
      const userRef = doc(db, 'online_users', user.uid);
      try {
        await updateDoc(userRef, {
          uid: user.uid,
          displayName: user.displayName || 'Anonymous',
          isOnline: true,
          lastSeen: serverTimestamp(),
        });
      } catch (e) {
        console.error("Error setting user online status on login: ", e);
        // If the document doesn't exist, create it
        await setDoc(userRef, {
          uid: user.uid,
          displayName: user.displayName || 'Anonymous',
          isOnline: true,
          lastSeen: serverTimestamp(),
        });
      }
    }
    // The watch effect above will handle starting/stopping the timer based on the user state
  });
});

// Clean up listeners and timer when the component is unmounted
onUnmounted(() => {
  removeActivityListeners();
  clearIdleTimer();
});

</script>

<template>
  <div>
    <div v-if="!loggedInUser && !loading">
      <AuthVue/>
    </div>

    <div v-if="loggedInUser">
      <div class="grid grid-cols-1 lg:grid-cols-4 gap-4 bg-slate-50">

         <button
            @click="handleLogout"
            class="border border-slate-300 rounded-md py-1 px-2 cursor-pointer hover:bg-red-400 lg:hidden"
          >
            Logout
          </button>

        <div class="col-span-1 lg:col-span-2">
          <!-- <Chat :loggedInUser="loggedInUser" /> -->
          <Chat2 :loggedInUser="loggedInUser" />
        </div>

        <div class="col-span-1 hidden lg:block mr-5 mt-5">
          <div class="flex justify-end">
            <button
            @click="handleLogout"
              class="border border-slate-300 rounded-md py-1 px-2 cursor-pointer hover:bg-red-400 bg-white"
            >
              Logout
            </button>
          </div>
          
          <!-- <div class="relative h-[85dvh] w-full mt-3 shadow-lg rounded-2xl overflow-hidden">
            <iframe
              class="h-full w-full"
              src="https://fleetstorm.lorenzhohmann.de/match"
              frameborder="0"
            ></iframe>
            <div
              class="pointer-events-none absolute inset-0 bg-black opacity-30 flex items-center justify-center text-gray-800 text-lg font-bold"
            >
            </div>
          </div> -->
        </div>

      </div>

    </div>
    <div v-else-if="loading" class="w-full h-[100dvh] flex flex-col justify-center items-center">
      <p>Loading user session...</p>
      <!-- <img src="/cattoo.gif" alt="" width="200"> -->
    </div>
  </div>
</template>