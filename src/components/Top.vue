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
const inputDate = ref<string | null>(null)

// Flatpickr カレンダー設定
const flatpickrOptions = {
    locale: Japanese,
    dateFormat: 'Y-m-d',
    defaultDate: null,
    weekNumbers: false,
    disableMobile: true,
    onChange: (selectedDates: Date[]) => {
        inputDate.value = dayjs(selectedDates[0]).format('YYYY-MM-DD')
    }
}

// カレンダー初期化関数
const initializeFlatpickr = () => {
    const calendarElement = document.querySelector('#calendar') as HTMLInputElement | null
    if (calendarElement instanceof HTMLInputElement) {
        //@ts-ignore flatpickrの型定義にあてはまらないため、型エラーを無視
        flatpickr(calendarElement, flatpickrOptions)
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

// FlatpickrをVueライフサイクルに適用
onMounted(() => {
    initializeFlatpickr()
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
.input-date {
    @apply block w-72 p-2 border border-gray-300 rounded focus:outline-none focus:border-blue-400 transition duration-300 mb-4;
    margin: 0 auto 16px;
    text-align: center;
}

/* Section title style */
.section-title {
    @apply text-xl font-bold mb-4 text-center text-blue-600;
}

.title-line {
    @apply py-1 font-bold text-blue-500;
}

/* Date section style */
.date-section {
    @apply flex-wrap justify-center;
    display: flex;
    gap: 8px;
}

/* Date group style */
.date-group {
    @apply w-full bg-blue-50 border border-blue-200 px-4 py-3 mx-2;
    max-width: 320px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Date group title style */
.date-group-title {
    @apply text-lg font-semibold mb-1 text-blue-700;
}

/* Date info style */
.date-info {
    @apply p-2 bg-white border border-blue-200 rounded shadow-md mb-1;
    border-radius: 8px;
}

/* Date info title style */
.date-info-title {
    @apply text-base font-semibold mb-1 text-gray-800;
}

/* Date info time style */
.date-info-time {
    @apply text-gray-700 text-sm;
}

/* Error message style */
.error-message {
    @apply text-red-600 text-sm font-semibold;
}

/* レスポンシブスタイル */
@media (max-width: 1100px) {
    .section-title {
        @apply text-lg mb-1;
    }

    .input-date {
        @apply mb-3;
    }

    .date-section {
        @apply flex-col items-center gap-3;
    }

    .date-group {
        @apply w-full px-3 py-2;
    }

    .date-info {
        @apply p-2 mb-1;
    }

    .date-info-time {
        font-size: 0.85rem;
    }
}
</style>
