<script setup lang="ts">
import Top from './components/Top.vue'
</script>

<template>
    <div class="app">
        <header class="bg-gray-400 flex p-4">
            <div class="text-white ml-5">
                <p class="text-3xl font-semibold">JRA指定席</p>
                <p class="text-3xl font-semibold">ネット予約スケジュール</p>
            </div>
        </header>
        <div class="container">
            <main>
                <Top />
            </main>
        </div>
    </div>
</template>

<style scoped>
/* App style */
.app {
    @apply m-0 p-0 w-full min-h-screen bg-gray-50;
}

/* Header style */
header {
    @apply flex items-center px-4 py-2 bg-blue-200 shadow-sm;
}

header p {
    @apply text-gray-800 text-lg font-semibold;
}

.container {
    @apply px-6 py-8 text-center bg-gray-50 max-w-7xl mx-auto;
    margin-top: 0;
}

/* レスポンシブスタイル */
@media (max-width: 10240px) {
    header {
        @apply px-3 py-1;
    }

    .container {
        @apply px-4 py-1.5;
    }

    header p {
        @apply text-sm;
    }
}
</style>
