<script lang="ts" setup>
import { ref, computed, onMounted, type ComputedRef } from 'vue'
import dayjs, { Dayjs } from 'dayjs'
import ja from 'dayjs/locale/ja'
import flatpickr from 'flatpickr'
import 'flatpickr/dist/flatpickr.min.css'
import { Japanese } from 'flatpickr/dist/l10n/ja.js'

// day.js ロケール設定
dayjs.locale(ja)

// Japaneseロケールに週の開始日を設定
Japanese.firstDayOfWeek = 1 // 月曜始まり

// 日付を管理する変数
const selectedDate = ref<string | null>(null)

// Flatpickr カレンダー設定
const flatpickrOptions = {
    locale: Japanese,
    dateFormat: 'Y-m-d',
    defaultDate: null,
    weekNumbers: false,
    disableMobile: true,
    onChange: (selectedDates: Date[]) => {
        selectedDate.value = dayjs(selectedDates[0]).format('YYYY-MM-DD')
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

// FlatpickrをVueライフサイクルに適用
onMounted(() => {
    initializeFlatpickr()
})

// 有効な日付か判定 (土曜または日曜)
const isEnable = computed(() => {
    if (selectedDate.value == null) {
        return false
    }
    const originDate = dayjs(selectedDate.value)
    return originDate.day() == 0 || originDate.day() == 6
})

// 日付計算用関数
const calculationDate = (standardSubtractStart: number): Dayjs | undefined => {
    if (selectedDate.value == null) return undefined

    const originDate = dayjs(selectedDate.value)
    if (originDate.day() === 0) {
        // 日曜日
        return originDate.subtract(standardSubtractStart, 'day')
    } else if (originDate.day() === 6) {
        // 土曜日
        return originDate.subtract(standardSubtractStart - 1, 'day')
    }
}

interface Period {
    startDate: Dayjs | undefined
    startTime: string
    endDate: Dayjs | undefined
    endTime: string
    title: string
}

interface SalePeriod {
    sectionTitle: string
    periods: {
        [key: string]: Period
    }
}

const salePeriods: ComputedRef<Record<string, SalePeriod>> = computed(() => ({
    preSale: {
        sectionTitle: 'JRAカード先行',
        periods: {
            entry: {
                startDate: calculationDate(16),
                startTime: '18:00',
                endDate: calculationDate(14),
                endTime: '13:00',
                title: '申込期間'
            },
            result: {
                startDate: calculationDate(12),
                startTime: '18:00',
                endDate: calculationDate(10),
                endTime: '13:00',
                title: '当選確認・購入期間'
            }
        }
    },
    generalSale: {
        sectionTitle: '一般抽選',
        periods: {
            entry: {
                startDate: calculationDate(12),
                startTime: '18:00',
                endDate: calculationDate(10),
                endTime: '13:00',
                title: '申込期間'
            },
            result: {
                startDate: calculationDate(9),
                startTime: '18:00',
                endDate: calculationDate(7),
                endTime: '13:00',
                title: '当選確認・購入期間'
            }
        }
    },
    remainingSeat: {
        sectionTitle: '残席先着',
        periods: {
            sale: {
                startDate: calculationDate(6),
                startTime: '18:00',
                endDate: dayjs(selectedDate.value),
                endTime: '15:00',
                title: '販売期間'
            }
        }
    }
}))

// Googleカレンダーにイベントを追加
const addToCalendar = (sectionTitle: String, period: Period) => {
    const title = `${sectionTitle} ${period.title}`
    const start =
        period.startDate?.format('YYYYMMDD') + 'T' + period.startTime.replace(':', '') + '00'
    const end = period.endDate?.format('YYYYMMDD') + 'T' + period.endTime.replace(':', '') + '00'
    const url = 'https://jra-tickets.jp/'

    const googleCalendarUrl = `https://www.google.com/calendar/render?action=TEMPLATE&text=${title}&dates=${start}/${end}&details=${url}`
    window.open(googleCalendarUrl, '_blank')
}
</script>

<template>
    <div class="section-title">
        <p class="title-line">競馬開催日</p>
    </div>

    <div>
        <input
            id="calendar"
            class="select-date"
            autocomplete="off"
            placeholder="日付を選択してください"
        />
    </div>
    <div v-if="isEnable" class="data-section">
        <div v-for="(sale, key) in salePeriods" :key="key" class="date-group">
            <p class="date-group-title">{{ sale.sectionTitle }}</p>
            <div v-for="(period, key) in sale.periods" :key="key" class="date-info">
                <div class="date-info-row">
                    <p class="date-info-title">{{ period.title }}</p>
                    <button
                        type="button"
                        class="add-calendar-button group relative"
                        @click="addToCalendar(sale.sectionTitle, period)"
                    >
                        <img class="calendar-icon" src="@/assets/calender.svg" />
                        登録
                        <span class="tooltip">Googleカレンダーに登録</span>
                    </button>
                </div>
                <p class="date-info-time">
                    {{ period.startDate?.format('M/D(ddd)') }} {{ period.startTime }}
                    〜
                    {{ period.endDate?.format('M/D(ddd)') }} {{ period.endTime }}
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
.select-date {
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
.data-section {
    @apply flex-wrap justify-center;
    display: flex;
    gap: 8px;
}

/* Date group style */
.date-group {
    @apply w-full bg-blue-50 border border-blue-200 px-3 py-3 mx-1.5;
    max-width: 305px;
    border-radius: 6px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Date group title style */
.date-group-title {
    @apply text-lg font-semibold mb-1 text-blue-700;
}

/* Date info style */
.date-info {
    @apply p-2 bg-white border border-blue-200 rounded shadow-md mb-3;
    border-radius: 8px;
}

.date-info-row {
    @apply flex items-center relative gap-2 mb-2;
}

.date-info-title {
    @apply text-base font-semibold text-gray-800 text-center w-full;
}

.add-calendar-button {
    @apply absolute right-0 flex items-center justify-center px-1 py-0.5 text-blue-600 border border-blue-400 rounded font-medium text-xs h-5 transition-colors;
}

.add-calendar-button:hover {
    @apply bg-blue-50 text-blue-300;
}

.calendar-icon {
    @apply w-3 h-3 mr-1;
}

.date-info-time {
    @apply text-gray-700 text-sm;
}

.tooltip {
    @apply absolute bottom-full left-1/2 transform -translate-x-1/2 bg-black text-white text-xs px-2 py-1 rounded-md whitespace-nowrap opacity-0 invisible transition-opacity duration-200 z-10;
}

.add-calendar-button:hover .tooltip {
    @apply opacity-100 visible;
}

/* Error message style */
.error-message {
    @apply text-red-600 text-sm font-semibold;
}

/* レスポンシブスタイル */
@media (max-width: 1280px) {
    .section-title {
        @apply text-lg mb-1;
    }

    .select-date {
        @apply mb-3 py-1 text-sm;
    }

    .data-section {
        @apply flex-col items-center gap-2.5;
    }

    .date-group {
        @apply w-full px-3 py-2;
    }

    .date-group-title {
        @apply text-base;
    }

    .add-calendar-button {
        @apply px-1 py-0.5 text-xs h-5;
    }

    .calendar-icon {
        @apply w-3 h-3 mr-1;
    }

    .date-info {
        @apply px-2 py-1.5 mb-1;
    }

    .date-info-title {
        @apply text-sm;
    }

    .date-info-time {
        @apply text-xs;
    }
}
</style>
