<script lang="ts" setup>
import { ref, computed, onMounted } from 'vue'
import dayjs from 'dayjs'
import ja from 'dayjs/locale/ja'
import flatpickr from 'flatpickr'
import 'flatpickr/dist/flatpickr.min.css'
import { Japanese } from 'flatpickr/dist/l10n/ja.js'

// day.js ロケール設定
dayjs.locale(ja)

// Japaneseロケールに週の開始日を設定
Japanese.firstDayOfWeek = 1 // 月曜始まり

// 日付を管理する変数
const inputDate = ref(null)

// Flatpickr カレンダー設定
const flatpickrOptions = {
    locale: Japanese,
    dateFormat: 'Y-m-d',
    defaultDate: null,
    weekNumbers: false,
    onChange: (selectedDates) => {
        inputDate.value = dayjs(selectedDates[0]).format('YYYY-MM-DD')
    }
}

// 有効な日付か判定 (土曜または日曜)
const isEnable = computed(() => {
    if (inputDate.value == null) {
        return false
    }
    const originDate = dayjs(inputDate.value)
    return originDate.day() == 0 || originDate.day() == 6
})

// 日付計算用関数
const calculationDate = (standardSubtractStart: number) => {
    if (!inputDate.value) return null
    const originDate = dayjs(inputDate.value)
    if (originDate.day() === 0) {
        // 日曜日
        return originDate.subtract(standardSubtractStart, 'day').format('M/D(ddd)')
    } else if (originDate.day() === 6) {
        // 土曜日
        return originDate.subtract(standardSubtractStart - 1, 'day').format('M/D(ddd)')
    }
}

// JRAカード先行抽選販売
const preSaleEntryStart = computed(() => calculationDate(16))
const preSaleEntryEnd = computed(() => calculationDate(14))
const preSaleResultStart = computed(() => calculationDate(12))
const preSaleResultEnd = computed(() => calculationDate(10))

// 一般抽選販売
const generalSaleEntryStart = computed(() => calculationDate(12))
const generalSaleEntryEnd = computed(() => calculationDate(10))
const generalSaleResultStart = computed(() => calculationDate(9))
const generalSaleResultEnd = computed(() => calculationDate(7))

// 残席先着販売
const remainingSeatSaleStart = computed(() => calculationDate(6))

// DOMにカレンダーを適用
onMounted(() => {
    flatpickr('#calendar', flatpickrOptions)
})
</script>

<template>
    <div class="section-title">
        <p class="title-line">競馬開催日</p>
    </div>

    <div>
        <input id="calendar" class="input-date" placeholder="日付を選択してください" />
    </div>
    <div v-if="isEnable" class="date-section">
        <!-- JRAカード先行販売 -->
        <div class="date-group">
            <p class="date-group-title">JRAカード先行</p>
            <div class="date-info">
                <p class="date-info-title">申込期間</p>
                <p class="date-info-time">
                    {{ preSaleEntryStart }} 18:00 〜 {{ preSaleEntryEnd }} 13:00
                </p>
            </div>
            <div class="date-info">
                <p class="date-info-title">当選確認・購入期間</p>
                <p class="date-info-time">
                    {{ preSaleResultStart }} 18:00 〜 {{ preSaleResultEnd }} 13:00
                </p>
            </div>
        </div>
        <!-- 一般抽選販売 -->
        <div class="date-group">
            <p class="date-group-title">一般抽選</p>
            <div class="date-info">
                <p class="date-info-title">申込期間</p>
                <p class="date-info-time">
                    {{ generalSaleEntryStart }} 18:00 〜 {{ generalSaleEntryEnd }} 13:00
                </p>
            </div>
            <div class="date-info">
                <p class="date-info-title">当選確認・購入期間</p>
                <p class="date-info-time">
                    {{ generalSaleResultStart }} 18:00 〜 {{ generalSaleResultEnd }} 13:00
                </p>
            </div>
        </div>
        <!-- 残席先着販売 -->
        <div class="date-group">
            <p class="date-group-title">残席先着</p>
            <div class="date-info">
                <p class="date-info-title">販売期間</p>
                <p class="date-info-time">
                    {{ remainingSeatSaleStart + ' 18:00 〜 当日 15:00' }}
                </p>
            </div>
        </div>
    </div>
    <div v-else>
        <p class="error-message">
            土曜・日曜を選択してください<br />金杯・ホープフルS等は未対応です
        </p>
    </div>
</template>

<style scoped>
/* Input style */
.input-date {
    @apply block w-64 p-2 border border-gray-300 rounded focus:outline-none focus:border-blue-400 transition duration-300 mb-8;
    margin: 0 auto 16px;
}

/* Section title style */
.section-title {
    @apply text-xl font-bold mb-1 text-center;
}

.title-line {
    @apply py-1 font-bold;
}

.schedule-text {
    @apply text-base font-normal mb-4 text-center;
}

/* Date section style */
.date-section {
    @apply flex-wrap justify-center -mx-2;
}

/* Date group style */
.date-group {
    @apply w-full bg-green-50 border border-green-200 px-2 mb-6;
    min-width: 240px;
    max-width: 600px;
    border-radius: 10px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
    margin-right: auto;
    margin-left: auto;
    margin-bottom: 24px;
    flex-flow: column;
}

/* Date group title style */
.date-group-title {
    @apply text-base font-semibold mb-1;
}

/* Date info style */
.date-info {
    @apply p-3 bg-white border border-green-200 rounded shadow-md mb-1.5;
    border-radius: 10px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
    margin-right: 4px;
    margin-left: 4px;
    margin-bottom: 14px;
}

/* Date info title style */
.date-info-title {
    @apply text-base font-semibold mb-1;
}

/* Date info time style */
.date-info-time {
    @apply text-gray-700 text-sm;
}

/* Error message style */
.error-message {
    @apply text-red-600 text-base;
}

@media (max-width: 1024px) {
    /* Adjust styles for 1024px screens and smaller */
    .input-date {
        @apply mb-4;
    }

    .date-group {
        @apply w-full px-2 mb-2.5;
    }

    .date-section {
        @apply flex-col;
    }

    .date-info {
        margin-right: 0;
        margin-left: 0;
        margin-bottom: 4px;
    }
}
</style>
