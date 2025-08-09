<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed, nextTick, watch } from 'vue';
import { auth, db } from '@/firebase';
import { collection, query, orderBy, onSnapshot } from 'firebase/firestore';
import linkify from '@/condition/useLinkify';

import { formatTimestamp } from '@/condition/format';
import { useSmartChatScroll } from '@/condition/useSmartChatScroll';
import { useReactions } from '@/condition/useReactions';
import { useStagger } from '@/condition/useStagger';
import { useSendMessage } from '@/condition/useSendMessage';
import { useTyping } from '@/condition/useTyping';
import { useGroupedMessages } from '@/condition/useGroupedMessages';
import { useClipboardImage } from '@/condition/useClipboardImage';

const messageLimit = ref(15);
const allMessages = ref<ChatMessage[]>([]);
const hasMoreMessages = ref(true);

function updateVisibleMessages() {
    messages.value = allMessages.value.slice(-messageLimit.value);
    if (messageLimit.value >= allMessages.value.length) {
        hasMoreMessages.value = false;
    }
}

function loadMoreMessages() {
    if (hasMoreMessages.value) {
        const container = chatContainerRef.value;
        const oldScrollHeight = container?.scrollHeight || 0;

        messageLimit.value += 15;

        nextTick(() => {
        updateVisibleMessages();

        // Maintain scroll position after loading
        nextTick(() => {
            if (container) {
            const newScrollHeight = container.scrollHeight;
            container.scrollTop = newScrollHeight - oldScrollHeight;
            }
        });
        });
    }
}

const {
    chatContainerRef,
    isUserNearBottom,
    newMessageCount,
    scrollToBottom,
    handleScroll,
    handleNewMessage
} = useSmartChatScroll(loadMoreMessages);

const {
    reactions,
    fetchReactions,
    addReaction,
    toggleReactionPicker,
    groupedReactions,
    openReactionMessageId,
    reactionEmojis,
} = useReactions();

const {
    staggerEnter,
    staggerLeave
} = useStagger();

const newMessage = ref('');

const {
    sendMessage,
    isSending,
    isCurrentUserBanned,
    banReason,
    banEndTime,
} = useSendMessage(newMessage);

const {
    isSomeoneTyping,
    handleTyping,
    stopTyping
} = useTyping();

// Use the new composable
const {
    imageUrl,
    shortenedImageUrl,
    isLoading: isProcessingImage,
    error: imageProcessError,
    processClipboardImage,
    clearImage,
} = useClipboardImage();


interface ChatMessage {
    id: string;
    text: string;
    userId: string;
    displayName: string;
    timestamp: any;
}

const loggedInUser = computed(() => auth.currentUser);
const messages = ref<ChatMessage[]>([]);
const originalTitle = document.title;

const { groupedMessages } = useGroupedMessages(messages);

// Function to handle the paste event on the textarea
const handlePaste = async (event: ClipboardEvent) => {
    if (event.clipboardData && event.clipboardData.items) {
        const items = Array.from(event.clipboardData.items);
        const hasImage = items.some(item => item.type.indexOf('image') !== -1);

        if (hasImage) {
        event.preventDefault(); // Prevent default text paste if an image is detected
        await processClipboardImage(event);
        }
    }
};

const onlineOfflineList = [
  { name: 'Alice', avatar: 'https://i.pravatar.cc/40?img=1', online: true },
  { name: 'Bob', avatar: 'https://i.pravatar.cc/40?img=2', online: true },
  { name: 'Charlie', avatar: 'https://i.pravatar.cc/40?img=3', online: false }
]

const friendsList = [
  { name: 'Jane Doe', avatar: 'https://i.pravatar.cc/40?img=4' },
  { name: 'Mike Ross', avatar: 'https://i.pravatar.cc/40?img=5' },
  { name: 'Sarah Lee', avatar: 'https://i.pravatar.cc/40?img=6' }
]

const groupList = [
  { name: 'Work Buddies', avatar: 'https://i.pravatar.cc/40?img=7' },
  { name: 'Gaming Squad', avatar: 'https://i.pravatar.cc/40?img=8' },
  { name: 'Study Group', avatar: 'https://i.pravatar.cc/40?img=9' }
]

// Create a wrapper function for sending messages
const sendChatMessage = async () => {
    // Call the original sendMessage from useSendMessage
    await sendMessage();
    // After the message is sent, clear the image preview
    clearImage();
};

// Watch for imageUrl changes and append to newMessage if available
watch(imageUrl, (newImageUrl) => {
    if (newImageUrl) {
        // Check if the URL is already present to avoid duplicates on re-paste attempts
        if (!newMessage.value.includes(newImageUrl)) {
        // Append the new image URL to the message. Add a newline for better readability.
        newMessage.value += `\n${newImageUrl}`;
        }
    }
});



onMounted(() => {
    chatContainerRef.value?.addEventListener('scroll', handleScroll);

    // chatContainerRef.value?.addEventListener('click', handleMessageContentClick);


});

onBeforeUnmount(() => {
    chatContainerRef.value?.removeEventListener('scroll', handleScroll);
    document.title = originalTitle;
});

</script>

<template>
  <div class="h-screen flex bg-gray-100 p-4">
    <!-- LEFT SIDEBAR -->
    <div class="w-60 flex flex-col border border-gray-300 rounded-lg overflow-hidden bg-white shadow">
    <!-- ONLINE/OFFLINE -->
    <div>
      <div class="bg-blue-500 text-white px-3 py-2 text-sm font-semibold">Online/Offline</div>
      <ul class="divide-y divide-gray-200">
        <li v-for="user in onlineOfflineList" :key="user.name"
            class="flex items-center gap-2 p-2 hover:bg-gray-50 cursor-pointer">
          <img :src="user.avatar" alt="" class="w-8 h-8 rounded-full object-cover border" />
          <span class="flex-1 text-sm">{{ user.name }}</span>
          <span :class="['w-3 h-3 rounded-full', user.online ? 'bg-green-500' : 'bg-gray-400']"></span>
        </li>
      </ul>
    </div>

    <!-- FRIENDS -->
    <div>
      <div class="bg-green-500 text-white px-3 py-2 text-sm font-semibold">Friends</div>
      <ul class="divide-y divide-gray-200">
        <li v-for="friend in friendsList" :key="friend.name"
            class="flex items-center gap-2 p-2 hover:bg-gray-50 cursor-pointer">
          <img :src="friend.avatar" alt="" class="w-8 h-8 rounded-full object-cover border" />
          <span class="flex-1 text-sm">{{ friend.name }}</span>
        </li>
      </ul>
    </div>

    <!-- GROUP -->
    <div class="flex-1">
      <div class="bg-purple-500 text-white px-3 py-2 text-sm font-semibold">Group</div>
      <ul class="divide-y divide-gray-200">
        <li v-for="group in groupList" :key="group.name"
            class="flex items-center gap-2 p-2 hover:bg-gray-50 cursor-pointer">
          <img :src="group.avatar" alt="" class="w-8 h-8 rounded-full object-cover border" />
          <span class="flex-1 text-sm">{{ group.name }}</span>
        </li>
      </ul>
    </div>
  </div>

    <!-- MAIN CHAT AREA -->
    <div class="flex flex-col flex-1 ml-4">
      <!-- HEADER -->
      <div class="flex justify-between items-center mb-3 bg-white p-3 rounded-md shadow">
        <p class="text-lg">
          Hello, <strong><i>{{ loggedInUser?.displayName }}</i></strong> !
        </p>
        <div class="flex items-center gap-3">
          <img
            class="w-12 h-12 rounded-md border"
            src="https://i.pinimg.com/736x/7d/f9/73/7df973f1d0f4d0a449c4aab20f45627f.jpg"
            alt="Profile"
          />
          <button
            class="border border-gray-300 rounded-md px-3 py-1 bg-white hover:bg-red-500 hover:text-white transition"
          >
            Logout
          </button>
        </div>
      </div>

      <!-- CHAT MESSAGES -->
      <div
        ref="chatContainerRef"
        class="flex-1 border border-black bg-white overflow-y-auto rounded-md shadow p-3"
      >
        <!-- Messages rendering from your groupedMessages -->
      </div>

      <!-- MESSAGE INPUT -->
      <div class="mt-3 bg-white p-2 border border-gray-300 rounded-md flex items-center shadow">
        <textarea
          v-model="newMessage"
          placeholder="Type your message..."
          @keydown.enter.exact.prevent="sendChatMessage"
          class="flex-1 p-2 border border-gray-300 rounded-md resize-none focus:outline-none focus:ring-2 focus:ring-blue-400"
          rows="2"
        />
        <button
          @click="sendChatMessage"
          :disabled="!newMessage.trim()"
          class="ml-2 px-5 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-400 disabled:bg-gray-300"
        >
          Send
        </button>
      </div>
    </div>
  </div>
</template>

