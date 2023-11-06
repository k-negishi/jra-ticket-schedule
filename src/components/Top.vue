<script lang="ts" setup>
import { ref, computed } from 'vue'
import dayjs from 'dayjs'
import ja from 'dayjs/locale/ja'

dayjs.locale(ja)

const inputDate = ref(null)

const isEnable = computed(() => {
    if (inputDate.value == null) {
        return false
    }
    const originDate = dayjs(inputDate.value)
    return originDate.day() == 0 || originDate.day() == 6
})

const calculationDate = (standardSubtractStart: number) => {
    const originDate = dayjs(inputDate.value)
    if (originDate.day() == 0) {
        return originDate.subtract(standardSubtractStart, 'day').format('M/D(ddd)')
    } else if (originDate.day() == 6) {
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
</script>

<template>
    <div class="section-title">
        <p class="title-line">競馬開催日</p>
    </div>

    <div>
        <input class="input-date" style="text-align: center" type="date" v-model="inputDate" />
    </div>
    <div v-if="isEnable" class="date-section">
        <p class="schedule-text">の予約スケジュールは...</p>
        <!-- JRAカード先行販売 -->
        <div class="date-group">
            <p class="date-group-title">JRAカード先行</p>
            <div class="date-info">
                <p class="date-info-title">申込期間</p>
                <p class="date-info-time">
                    {{ preSaleEntryStart + ' 18:00 〜 ' + preSaleEntryEnd + ' 13:00' }}
                </p>
            </div>
            <div class="date-info">
                <p class="date-info-title">当選確認・購入期間</p>
                <p class="date-info-time">
                    {{ preSaleResultStart + ' 18:00 〜 ' + preSaleResultEnd + ' 13:00' }}
                </p>
            </div>
        </div>
        <!-- 一般抽選販売 -->
        <div class="date-group">
            <p class="date-group-title">一般抽選</p>
            <div class="date-info">
                <p class="date-info-title">申込期間</p>
                <p class="date-info-time">
                    {{ generalSaleEntryStart + ' 18:00 〜 ' + generalSaleEntryEnd + ' 13:00' }}
                </p>
            </div>
            <div class="date-info">
                <p class="date-info-title">当選確認・購入期間</p>
                <p class="date-info-time">
                    {{ generalSaleResultStart + ' 18:00 〜 ' + generalSaleResultEnd + ' 13:00' }}
                </p>
            </div>
        </div>
        <!-- 残席先着販売 -->
        <div class="date-group">
            <p class="date-group-title">残席先着</p>
            <div class="date-info">
                <p class="date-info-title">販売期間</p>
                <p class="date-info-time">{{ remainingSeatSaleStart + ' 18:00 〜 当日 15:00' }}</p>
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
    @apply block w-64 p-2 border border-gray-300 rounded focus:outline-none focus:border-blue-400 transition duration-300 mb-4;
    margin: 0 auto 12px;
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
    @apply flex flex-wrap justify-center -mx-2;
}

/* Date group style */
.date-group {
    @apply w-full bg-green-50 border border-green-200 md:max-w-3xl px-2 mb-1;
    min-width: 240px;
    border-radius: 10px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

/* Date group title style */
.date-group-title {
    @apply text-xl font-semibold mb-1;
}

/* Date info style */
.date-info {
    @apply p-4 bg-white border border-green-200 rounded shadow-md mb-2;
    border-radius: 10px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
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
</style>
