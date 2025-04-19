<script setup lang="ts">
import ScheduleOverview from './components/ScheduleOverview.vue'
</script>

<template>
    <div class="app">
        <header class="bg-gray-400 flex p-4">
            <div class="text-white ml-5">
                <h1 class="text-xl font-semibold">
                    <span class="block">JRA指定席</span>
                    <span class="block">ネット予約スケジュール</span>
                </h1>
            </div>
        </header>
        <div class="container">
            <main aria-label="予約スケジュール計算ツール">
                <ScheduleOverview />
            </main>
        </div>
        <footer>
            <a
                href="https://github.com/k-negishi/jra-ticket-schedule"
                class="github-link"
                target="_blank"
                rel="noopener noreferrer"
                aria-label="GitHubでソースコードを見る"
            >
                <img src="@/assets/github-icon.svg" alt="GitHub" class="github-icon" />
                GitHubで見る
            </a>
        </footer>
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

header h1 {
    @apply text-gray-800 font-semibold;
}

.container {
    @apply px-6 py-8 text-center bg-gray-50 max-w-7xl mx-auto;
    margin-top: 0;
}

footer {
    @apply flex justify-center items-center;
}

.github-link {
    @apply flex items-center gap-2 text-gray-800 font-semibold hover:text-gray-400 m-8;
    text-decoration: none;
    align-items: center;
    justify-content: center;
}

.github-icon {
    width: 20px;
    height: 20px;
    fill: currentColor;
    transition: transform 0.2s ease;
}

.github-link:hover .github-icon {
    transform: scale(1.2);
}

/* レスポンシブスタイル */
@media (max-width: 1280px) {
    header {
        @apply px-3 py-1;
    }

    .container {
        @apply px-4 py-1.5;
    }

    header h1 {
        @apply text-sm;
    }
}
</style>
