<script setup>
import { computed, ref } from 'vue';
import Display from './Display.vue'

const sex = ref()
const height = ref()
const weight = ref()
const hematocrit = ref()
const exchangeVolumeMultiplier = ref(1.0)
const ffpFraction = ref(0)
const ffpUnitsStatic = ref(2)

const bloodVolume = computed(() => {
    if (!(sex.value && (height.value >=10) && (weight.value >=10))) {
        return 
    }
    if (sex.value == 'male') {
        return Math.round(1000 * (((height.value / 100) ** 3 * 0.3669) + (0.03219 * weight.value) + 0.6041))
    } else if (sex.value == 'female') {
        return Math.round(1000 * (((height.value / 100) ** 3 * 0.3561) + (0.03308 * weight.value) + 0.1833))
    }
})

const plasmaVolume = computed(() => {
    if (hematocrit.value) {
        return Math.round(bloodVolume.value * (1 - (hematocrit.value / 100)))
    }
})

const exchangeVolume = computed(() => {
    return Math.round(plasmaVolume.value * exchangeVolumeMultiplier.value)
})

const albuminAmount = computed(() => {
    if (!plasmaVolume.value) {
        return 'N/A'
    }
    if (ffpFraction.value != 'custom') {
        return Math.ceil(((exchangeVolume.value * (1 - ffpFraction.value)) * 0.05) / 12.5) * 12.5
    } else {
        return Math.ceil(((exchangeVolume.value - (ffpUnitsStatic.value * 250)) * 0.05) / 12.5) * 12.5
    }
})

const ffpAmount = computed(() => {
    return Math.ceil((exchangeVolume.value * ffpFraction.value) / 250)
})
</script>

<template>
    <!-- <h1>Therapeutic Plasma Exchange (TPE/PLEX)</h1> -->
     <h2>Patient Parameters</h2>
     <div style="display: grid; grid-template-columns: auto auto;">
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
             <div v-show="weight && height">
                 <label>Hct:</label>
             </div>
             <div v-show="weight && height">
                 <input type="number" v-model="hematocrit"
                     @input="() => { if (hematocrit >= 100) { hematocrit = 99 } else if (hematocrit < 0) { hematocrit = 0 } }"></input>
                 <label> %</label>
             </div>
            </div>
            <div style="display: grid; gap: 1rem; grid-template-columns: auto; justify-content: end; grid-auto-rows: min-content; padding-top: 0.5rem;">
                <Transition>
                    <Display v-if="bloodVolume" label="total blood volume" :value="bloodVolume" unit="mL"></Display>
                </Transition>
                <Transition>
                    <Display v-if="plasmaVolume && (hematocrit >= 10)" label="plasma volume" :value="plasmaVolume" unit="mL"></Display>
                </Transition>
            </div>
        </div>
        <Transition>
            <div v-show="plasmaVolume && (hematocrit >= 10)">
                    <h2 v-if="plasmaVolume">Procedure Parameters</h2>
                <div  style="display: grid; grid-template-columns: 3rem auto; gap: 1rem; align-items: center; margin: 1rem 0rem;">
                    <div>
                        <label>Volume:</label>
                    </div>
                    <div>
                        <div class="radio-group">
                            <button v-for="volume in [1, 1.2, 1.5]" @click="exchangeVolumeMultiplier = volume"
                                :class="{ selected: exchangeVolumeMultiplier == volume }">{{ volume.toFixed(1) }}x</button>
                        </div>
                    </div>
                    <div><label>FFP:</label></div>
                    <div>
                        <div class="radio-group">
                            <button @click="ffpFraction = 0" :class="{ selected: ffpFraction == 0 }">none</button>
                            <button @click="ffpFraction = 0.33" :class="{ selected: ffpFraction == 0.33 }">1/3</button>
                            <button @click="ffpFraction = 0.5" :class="{ selected: ffpFraction == 0.5 }">1/2</button>
                            <button @click="ffpFraction = 0.67" :class="{ selected: ffpFraction == 0.67 }">2/3</button>
                            <button @click="ffpFraction = 1" :class="{ selected: ffpFraction == 1 }">all</button>
                            <button @click="ffpFraction = 'custom'" :class="{ selected: ffpFraction == 'custom' }">custom</button>
                        </div>
                    </div>
                </div>
             </div>
            </Transition>
    <Transition>
        <div v-if="plasmaVolume && (hematocrit >= 10)">
            <h2>Orders</h2>
            <div 
                style="display:grid; grid-template-columns: auto auto; gap: 2rem; grid-auto-rows: min-content; padding-top: 0.5rem;">
                <Display label="exchange volume" :value="exchangeVolume" unit="mL" style="grid-column-start: 1; grid-column-end: 3;"></Display>
                <Display label="albumin (5%)" :value="albuminAmount" unit="g"></Display>
                <Display label="FFP" :value="ffpAmount" unit="units" v-if="ffpFraction != 'custom'"></Display>
                <div class="display" v-else>
                    <h3>FFP</h3>
                    <div style="display: flex; align-items: center; gap: 0.6rem;">
                        <h1 class="green" v-if="ffpFraction != 'custom'">{{ ffpAmount }}</h1>
                        <input class="display" v-model="ffpUnitsStatic" v-else
                            @input="() => { if (ffpUnitsStatic < 0) { ffpUnitsStatic = 0 } else if (ffpUnitsStatic > Math.ceil(exchangeVolume / 250)) { ffpUnitsStatic = Math.ceil(exchangeVolume / 250) } }"></input>
                        <h3 class="unit">units</h3>
                    </div>
                </div>
                <button
                    @click="() => { sex = null; height = null; weight = null; hematocrit = null; exchangeVolumeMultiplier = 1; ffpFraction = 0; ffpUnitsStatic = 2}" style="grid-column-start: 1; grid-column-end: 3; width: 8rem; justify-self: center;">reset
                    form</button>
            </div>
        </div>
    </Transition>
</template>

<style scoped>
.radio-group>*:first-child {
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
}

.radio-group>*:not(:first-child, :last-child) {
    border-radius: 0;
}

.radio-group>*:last-child {
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
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

input {
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
    margin-top: 1rem;
}

h3 {
    font-size: 1.2rem;
}

.display {
    border-left: 1px solid lightgrey;
    padding-left: 1rem;
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
    transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
    opacity: 0;
}
</style>
