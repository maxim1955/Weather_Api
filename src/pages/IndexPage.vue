<template>
    <q-page class="flex flex-center">
        <div class="q-pa-md row items-start q-gutter-md">
            <div class="col-12 flex justify-end" style="z-index: 1000">
                <q-btn color="primary" icon="settings" text-color="green">
                    <q-menu
                            transition-show="rotate"
                            transition-hide="rotate"
                            fit
                    >
                        <q-list style="width: 900px" class="q-mr-xl">
                            <div class="flex justify-center">
                                <p class="text-black text-h3 ">Settings</p>
                            </div>
                            <q-item v-for="item of localArray " class="flex justify-center">
                                <div class="flex justify-around " style="width: 50%;">
                                    <div class="">
                                        <q-img
                                                src="~assets/hamburger-menu.jpg"
                                                style="width: 30px;height: 30px"
                                        />
                                    </div>
                                    <div class="">
                                        {{ item.name }}
                                    </div>
                                    <div class="cursor-pointer" @click="deleteCity(item)">
                                        <q-img
                                                src="~assets/basket.png"
                                                style="width: 30px;height: 30px; color: #21BA45"
                                                spinner-color="white"

                                        />
                                    </div>
                                </div>
                            </q-item>
                            <p class="text-black text-center text-h3">add location</p>
                            <q-item clickable>
                                <q-item-section class="flex">
                                    <q-input outlined label="Outlined" v-model="cityInput"/>
                                    <q-btn
                                            flat
                                            color="primary"
                                            icon="check"
                                            size="10px"
                                            @click="getCoordinatesCity(cityInput)"
                                    />
                                </q-item-section>
                            </q-item>
                        </q-list>
                    </q-menu>
                </q-btn>
            </div>
            <q-card class="my-card flex items-center column" v-for="city of localArray">
                <div class="flex justify-between" style="width: 500px">
                    <q-card-section class="" style="width: 100%;">
                        <q-item-section class="text-black text-h4 text-center">
                            {{ city.name }}
                        </q-item-section>
                    </q-card-section>
                </div>
                <q-card-section>
                    Данные на:
                    {{ new Date(city.dt * 1000) }}
                    {{
                    '0' + new Date(city.dt * 1000).getHours() + ":" + new Date(city.dt * 1000).getMinutes() + ":" + new Date(city.dt * 1000).getSeconds()
                    }}
                </q-card-section>
                <q-card-section>
                    <div class="flex no-wrap justify-center items-center" style="width: 200px">
                        <q-img
                                src="~assets/weather_logo.jpg"
                                sizes="500px"

                        />
                        <h3>{{ city.main.temp }}&#8451;</h3>
                    </div>

                </q-card-section>
                <q-card-section class="flex justify-between" style="width: 50%">
                    <div>
                        <p>Feels like {{ city.main.feels_like }}</p>
                    </div>
                    <div v-for="weather of city.weather">
                        <p>{{ weather.description }}</p>
                    </div>
                </q-card-section>
                <q-card-section class="flex justify-between">
                    <div class="flex justify-center " style="width: 60px">
                        <q-img
                                src="~assets/wind-logo.png"
                        />
                        <p>{{ city.wind.speed }}m/s</p>
                    </div>
                    <div class="flex justify-center " style="width: 60px">
                        <q-img
                                src="~assets/barometr.png"
                        />
                        <p style="">{{ city.main.pressure }}</p>
                    </div>
                </q-card-section>
                <q-card-section class="flex justify-between" style="width: 250px">
                    <div class="flex justify-center no-wrap ">
                        <p>Humidity {{ city.main.humidity }} &#37;</p>
                    </div>
                    <div class="flex justify-center ">
                        <p style="">DewPoint &#8451; </p>
                    </div>
                </q-card-section>
                <q-card-section>
                    <div>
                        <p style=""> Visibility: {{ (city.visibility / 1000).toFixed(1) }}km</p>
                    </div>
                </q-card-section>
            </q-card>
        </div>
    </q-page>
</template>

<script setup>
import {api} from "boot/axios";
import {ref} from "vue";

let api_key = '516a6ce9be275e03b317760949f52f50'
let array = []
let localArray = ref([])
localArray.value = JSON.parse(localStorage.getItem("cities"));
if (localArray.value.length) {
    localArray.value.forEach(async (item) => {
        console.log(item)
        await api.get(`https://api.openweathermap.org/data/2.5/weather?lat=${item.coord.lat}&lon=${item.coord.lon}&appid=${api_key}&units=metric`)
            .then((res) => {
                array.push(res.data)
                localStorage.setItem('cities', JSON.stringify(array))
            })
            .catch((err) => {
                console.log(err)
            })
    })
}


let cityInput = ref('')

let getDataOfCity = ref({})
const getCoordinatesCity = async () => {
    await api.get(`http://api.openweathermap.org/geo/1.0/direct?q=${cityInput.value},{643}&limit=5&appid=${api_key}`)
        .then((res) => {
            getDataOfCity.value = res.data[1]
            cityInput.value = ''
        })
        .catch((err) => {
            console.log(err)
        })
    await api.get(`https://api.openweathermap.org/data/2.5/weather?lat=${getDataOfCity.value.lat}&lon=${getDataOfCity.value.lon}&appid=${api_key}&units=metric`)
        .then((res) => {
            localArray.value.push(res.data)
            localStorage.setItem('cities', JSON.stringify(localArray.value))
        })
        .catch((err) => {
            console.log(err)
        })
}
const deleteCity = (item) => {
    localArray.value.forEach((city) => {
        console.log(item)
        if (city.name === item.name) {
            console.log('ok')
            let index = localArray.value.indexOf(item)
            console.log(index)
            localArray.value.splice(index, 1)
            localStorage.setItem('cities', JSON.stringify(localArray.value))
        }
    })
}

</script>
