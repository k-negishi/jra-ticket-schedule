<script lang="ts" setup>
import { ref, computed, onMounted, watch, type ComputedRef } from 'vue'
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

interface RaceEvent {
    race_name: string
    location: string
}
const events = ref<RaceEvent[]>([])

// 選択日付に応じて API から開催情報を取得
const fetchEvents = async (dateString: string) => {
    const d = dayjs(dateString)
    const year = d.format('YYYY')
    const month = d.format('M') // 0 埋めは不要
    const day = d.format('D')
    const url = `https://hrwqlokwkuzy4hoksjkzp3gb7y0anmkd.lambda-url.ap-northeast-1.on.aws/jra-calendar/events?year=${year}&month=${month}&day=${day}`

    try {
        const res = await fetch(url)
        if (!res.ok) throw new Error('Network response was not ok')
        const data = await res.json()
        events.value = data.events ?? []
    } catch (error) {
        console.error(error)
        events.value = []
    }
}

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

// 選択日が変わったら API 取得
watch(selectedDate, (newDate) => {
    if (newDate == null) {
        events.value = []
        return
    }
    const weekday = dayjs(newDate).day()
    if (weekday === 0 || weekday === 6) {
        fetchEvents(newDate)
    } else {
        events.value = []
    }
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
    const title = `JRA ${sectionTitle} ${period.title}`

    // レース情報のテキストを作成
    let details = ''
    if (events.value.length > 0) {
        details = events.value.map((ev) => `${ev.race_name} - ${ev.location}`).join('\n')
        details += '\n\n'
    }
    details += 'https://jra-tickets.jp/'

    const start =
        period.startDate?.format('YYYYMMDD') + 'T' + period.startTime.replace(':', '') + '00'
    const end = period.endDate?.format('YYYYMMDD') + 'T' + period.endTime.replace(':', '') + '00'

    const encodedDetails = encodeURIComponent(details)
    const googleCalendarUrl = `https://www.google.com/calendar/render?action=TEMPLATE&text=${title}&dates=${start}/${end}&details=${encodedDetails}`
    window.open(googleCalendarUrl, '_blank')
}
</script>

<template>
    <section aria-labelledby="schedule-section-title">
        <h2 id="schedule-section-title" class="section-title">競馬開催日</h2>

        <div class="calendar-container">
            <label for="calendar" class="sr-only">日付を選択</label>
            <input
                id="calendar"
                class="select-date"
                autocomplete="off"
                placeholder="日付を選択してください"
                aria-describedby="date-instructions"
            />
            <span id="date-instructions" class="sr-only">土曜日または日曜日を選択してください</span>

            <!-- 開催レースカード -->
            <div v-if="events.length" class="event-card">
                <div class="event-card-title">🏇 開催レース 🏇</div>
                <div class="event-list">
                    <div v-for="(ev, idx) in events" :key="idx" class="event-item">
                        {{ ev.race_name }} — {{ ev.location }}
                    </div>
                </div>
            </div>
        </div>

        <div v-if="isEnable" class="data-section">
            <section v-for="(sale, key) in salePeriods" :key="key" class="date-group">
                <h3 class="date-group-title">{{ sale.sectionTitle }}</h3>
                <article v-for="(period, key) in sale.periods" :key="key" class="date-info">
                    <div class="date-info-row">
                        <h4 class="date-info-title">{{ period.title }}</h4>
                        <button
                            type="button"
                            class="add-calendar-button group relative"
                            @click="addToCalendar(sale.sectionTitle, period)"
                            aria-label="Googleカレンダーに予定を登録する"
                        >
                            <img
                                class="calendar-icon"
                                src="@/assets/calender.svg"
                                alt="カレンダーアイコン"
                            />
                            登録
                            <span class="tooltip">Googleカレンダーに登録</span>
                        </button>
                    </div>
                    <p class="date-info-time">
                        <time
                            :datetime="`${period.startDate?.format('YYYY-MM-DD')}T${
                                period.startTime
                            }`"
                            >{{ period.startDate?.format('M/D(ddd)') }} {{ period.startTime }}</time
                        >
                        〜
                        <time
                            :datetime="`${period.endDate?.format('YYYY-MM-DD')}T${period.endTime}`"
                            >{{ period.endDate?.format('M/D(ddd)') }} {{ period.endTime }}</time
                        >
                    </p>
                </article>
            </section>
        </div>
        <div v-else class="information-message">
            <p class="error-message" role="alert">
                土曜・日曜を選択してください<br />金杯・ホープフルS等は未対応です
            </p>
        </div>
    </section>
</template>

<style scoped>
.select-date {
    @apply block w-72 p-2 border border-gray-300 rounded focus:outline-none focus:border-blue-400 transition duration-300 mb-4;
    margin: 0 auto 16px;
    text-align: center;
}

.section-title {
    @apply text-xl font-bold mb-4 text-center text-blue-600;
    margin-top: 0;
}

.sr-only {
    @apply absolute w-px h-px p-0 -m-px overflow-hidden whitespace-nowrap border-0;
    clip: rect(0, 0, 0, 0);
}

.calendar-container {
    @apply mb-1 flex flex-col items-center;
}

.event-card {
    @apply w-full border px-4 py-0.5 my-1 mx-4 rounded-lg shadow-sm;
    max-width: 340px;
}

.event-card-title {
    @apply font-semibold text-base  mb-2 text-gray-800 text-center;
}

.event-list {
    @apply space-y-1;
}

.event-item {
    @apply text-sm text-gray-800;
}

.data-section {
    @apply flex flex-wrap justify-center gap-2;
}

.date-group {
    @apply w-full bg-blue-50 border border-blue-200 px-4 py-4 my-2 mx-4 rounded-lg shadow-md;
    max-width: 340px;
}

.date-group-title {
    @apply text-lg font-semibold mb-1 text-blue-700;
}

.date-info {
    @apply p-2 bg-white border border-blue-200 rounded-md shadow-md mb-3;
}

.date-info-row {
    @apply flex items-center relative gap-2 mb-2;
}

.date-info-title {
    @apply text-base font-semibold text-gray-800 text-center w-full;
}

.add-calendar-button {
    @apply absolute right-0 flex items-center justify-center px-1.5 py-0.5 text-blue-600 border border-blue-400 rounded font-medium text-sm h-6 transition-colors hover:bg-blue-50 hover:text-blue-300;
}

.calendar-icon {
    @apply w-3 h-3 mr-1;
}

.date-info-time {
    @apply text-gray-700 text-sm;
}

.tooltip {
    @apply absolute bottom-7 left-1/2 transform -translate-x-1 bg-black text-white text-xs px-2 py-1 rounded-md whitespace-nowrap invisible transition-opacity group-hover:opacity-100 group-hover:visible;
}

.error-message {
    @apply text-red-600 text-sm font-semibold;
}

@media (max-width: 1280px) {
    .section-title {
        @apply text-lg mb-1;
    }

    .select-date {
        @apply mb-2 py-1 text-sm;
    }

    .event-card {
        @apply px-2 py-1 my-1 mx-2;
    }

    .event-card-title {
        @apply text-sm mb-0.5;
    }

    .event-item {
        @apply text-xs;
    }

    .data-section {
        @apply flex flex-col items-center gap-1.5;
    }

    .date-group {
        @apply w-full px-2 py-1.5 my-0.5 mx-2;
    }

    .date-group-title {
        @apply text-base mb-0.5;
    }

    .add-calendar-button {
        @apply px-0.5 py-px text-xs h-4;
    }

    .calendar-icon {
        @apply w-2.5 h-2.5 mr-0.5;
    }

    .date-info {
        @apply px-1.5 py-1 mb-1;
    }

    .date-info-title {
        @apply text-sm;
    }

    .date-info-time {
        @apply text-xs leading-tight;
    }

    .tooltip {
        display: none;
    }
}
</style>
