<script setup>
import { computed, ref } from 'vue';
import Display from './Display.vue'

const sex = ref()
const height = ref()
const weight = ref()
const preHematocrit = ref()

const mode = ref()

const midHematocrit = ref()
const postHematocrit = ref()
const fcr = ref(30)

const pRBCHematocrit = ref(55)
const pRBCVolume = ref(300)

const albumin = ref(false)


const bloodVolume = computed(() => {
    if (sex.value == 'male') {
        return Math.round(1000 * (((height.value / 100) ** 3 * 0.3669) + (0.03219 * weight.value) + 0.6041))
    } else if (sex.value == 'female') {
        return Math.round(1000 * (((height.value / 100) ** 3 * 0.3561) + (0.03308 * weight.value) + 0.1833))
    }
})

const exchangeVolume = computed(() => {


    if (mode.value == 'standard') {
        return Math.round(((postHematocrit.value - preHematocrit.value) / 100 * bloodVolume.value) / ((pRBCHematocrit.value / 100) * (Math.log(preHematocrit.value / postHematocrit.value) / Math.log(fcr.value / 100))))
    } else {
        return Math.round(((postHematocrit.value - midHematocrit.value) / 100 * bloodVolume.value) / ((pRBCHematocrit.value / 100) * (Math.log(midHematocrit.value / postHematocrit.value)) / Math.log(fcr.value / 100)))
    }
})

const replacementVolume = computed(() => {
    if (mode.value == 'isovolemic') {
        return Math.round(bloodVolume.value * (preHematocrit.value / 100) * Math.log(preHematocrit.value / midHematocrit.value))
    }
})

</script>

<template>
    <!-- <h1>Red Blood Cell Exchange (RBCX)</h1> -->
    <div style="display: grid; grid-template-columns: 3rem auto; gap: 1rem; align-items: center; margin: 1rem 0rem;">
        <div>
            <label>Sex:</label>
        </div>
        <div class="radio-group">
            <button @click="sex = 'male'" :class="{ selected: sex == 'male' }">M</button>
            <button @click="sex = 'female'" :class="{ selected: sex == 'female' }">F</button>
        </div>
        <div v-show="sex">
            <label>Height:</label>
        </div>
        <div v-show="sex">
            <input type="number" v-model="height"></input>
            <label> cm</label>
        </div>
        <div v-show="sex">
            <label>Weight:</label>
        </div>
        <div v-show="sex">
            <input type="number" v-model="weight"></input>
            <label> kg</label>
        </div>
        <div v-show="bloodVolume">
            <label>PreHct:</label>
        </div>
        <div v-show="bloodVolume">
            <input type="number" v-model="preHematocrit"
            @input="() => { if (preHematocrit >= 100) { preHematocrit = 99 } else if (preHematocrit < 0) { preHematocrit = 0 } }"></input>
            <label> %</label>
        </div>
        <div v-show="preHematocrit >= 10">
            <label>Type:</label>
        </div>
        <div v-show="preHematocrit >= 10">
            <div class="radio-group">
                <button @click="mode = 'standard'" :class="{ selected: mode == 'standard' }">standard</button>
                <button v-if="preHematocrit > 22" @click="mode = 'isovolemic'" :class="{ selected: mode == 'isovolemic' }">isovolemic</button>
            </div>
        </div>
        <div v-if="mode == 'isovolemic'">
            <label>MidHct:</label>
        </div>
        <div v-if="mode == 'isovolemic'">
            <input type="number" v-model="midHematocrit"
                @input="() => { if (midHematocrit >= 100) { midHematocrit = 99 } else if (midHematocrit < 0) { midHematocrit = 0 } }"></input>
            <label> %</label>
        </div>
        <div v-show="mode">
            <label>PostHct:</label>
        </div>
        <div v-show="mode">
            <input type="number" v-model="postHematocrit"
                @input="() => { if (postHematocrit >= 100) { postHematocrit = 99 } else if (postHematocrit < 0) { postHematocrit = 0 } }"></input>
            <label> %</label>
        </div>
        <div v-show="mode">
            <label>FCR:</label>
        </div>
        <div v-show="mode">
            <input type="number" v-model="fcr"
                @input="() => { if (fcr >= 100) { fcr = 99 } else if (fcr < 0) { fcr = 0 } }"></input>
            <label> %</label>
        </div>
        <div v-show="mode">
            <label>pRBC Hct:</label>
        </div>
        <div v-show="mode">
            <input type="number" v-model="pRBCHematocrit"
                @input="() => { if (pRBCHematocrit >= 100) { pRBCHematocrit = 99 } else if (pRBCHematocrit < 0) { pRBCHematocrit = 0 } }"></input>
            <label> %</label>
        </div>
        <div v-show="mode">
            <label>pRBC Volume:</label>
        </div>
        <div v-show="mode">
            <input type="number" v-model="pRBCVolume"
                @input="() => { if (pRBCVolume <= 200) { pRBCVolume = 200 } }"></input>
            <label> mL</label>
        </div>
        <div v-show="mode == 'isovolemic'">

        </div>
        <div v-show="mode == 'isovolemic'">
            <input type="checkbox" v-model="albumin"> albumin (5%)</input>
        </div>
    </div>
    <Transition>
        <div v-if="exchangeVolume" style="display:grid; grid-template-columns: auto auto; gap: 2rem;">
            <!-- <Display v-if="bloodVolume" label="total blood volume" :value="bloodVolume" unit="mL"></Display> -->
            <Display label="exchange volume" :value="exchangeVolume" unit="mL"></Display>
            <Display label="pRBCs" :value="Math.ceil(exchangeVolume / pRBCVolume)" unit="units"></Display>
            <Display v-if="mode == 'isovolemic'" label="replacement volume" :value="replacementVolume" unit="mL">
            </Display>
            <Display v-if="albumin" label="albumin (5%)"
                :value="Math.ceil(replacementVolume * 0.05 / 12.5) * 12.5" unit="g"></Display>
            <button
                @click="() => { sex = null; height = null; weight = null; preHematocrit = null; mode = null; midHematocrit = null; postHematocrit = null }">reset
                form</button>
        </div>
    </Transition>
</template>

<style scoped>

.radio-group>*:first-child {
    border-top-left-radius: 1.5rem;
    border-bottom-left-radius: 1.5rem;
}

.radio-group>button {
    border-radius: 0;
}

.radio-group>*:last-child {
    border-top-right-radius: 1.5rem;
    border-bottom-right-radius: 1.5rem;
}

button.selected {
    background-color: rgb(240, 220, 39);
}

input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
    display: none;
    -webkit-appearance: none;
    margin: 0;
}

input[type=number] {
    -moz-appearance: textfield;
}

input[type=number] {
    padding: 0.5rem;
    border-radius: 0.5rem;
    width: 3rem;
    text-align: right;
    font-weight: bold;
    font-size: 1rem;
}

h1 {
    font-size: 1.6rem;
}

h2 {
    margin-top: 2rem;
    margin-bottom: 1rem;
}

h3 {
    font-size: 1.2rem;
}

.display {
    border-left: 1px solid lightgrey;
    padding-left: 1rem;
    padding-bottom: 1rem;
}

h3.unit {
    position: relative;
    top: 12px;
}

h1.green {
    font-size: 2.6rem;
    font-weight: 500;
    position: relative;
    top: -10px;
}

input.display {
    font-size: 2.6rem;
    font-weight: normal;
    text-align: center;
    width: 4rem;
    padding: 0rem;
    position: relative;
    top: 2px;
}

.v-enter-active,
.v-leave-active {
    transition: opacity 1s ease;
}

.v-enter-from,
.v-leave-to {
    opacity: 0;
}
</style>
