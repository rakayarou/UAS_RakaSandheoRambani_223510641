<template>
    <div class="flex flex-col flex-1 items-center">
      <!-- Banner -->
      <div
        v-if="route.query.preview"
        class="text-white p-4 bg-weather-secondary w-full text-center"
      >
        <p>
          You are currently previewing this city, click the "+"
          icon to start tracking this city.
        </p>
      </div>
      <!-- Weather Overview -->
      <div class="weather-overview">
        <h1 class="city-name">{{ route.params.city }}</h1>
        <p class="current-time">
          {{
            new Date(weatherData.currentTime).toLocaleDateString(
              "en-us",
              {
                weekday: "short",
                day: "2-digit",
                month: "long",
              }
            )
          }}
          {{
            new Date(weatherData.currentTime).toLocaleTimeString(
              "en-us",
              {
                timeStyle: "short",
              }
            )
          }}
        </p>
        <p class="current-temp">
          {{ Math.round(weatherData.current.temp) }}&deg;
        </p>
        <p class="feels-like">
          Feels like
          {{ Math.round(weatherData.current.feels_like) }}&deg;
        </p>
        <p class="weather-description">
          {{ weatherData.current.weather[0].description }}
        </p>
        <img
          class="weather-icon"
          :src="
            `http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`
          "
          alt=""
        />
      </div>
  
      <hr class="separator" />
  
      <!-- Hourly Weather -->
      <div class="hourly-weather">
        <div class="hourly-header">
          <h2 class="hourly-title">Hourly Weather</h2>
          <div class="hourly-list">
            <div
              v-for="hourData in weatherData.hourly"
              :key="hourData.dt"
              class="hourly-item"
            >
              <p class="hour">
                {{
                  new Date(
                    hourData.currentTime
                  ).toLocaleTimeString("en-us", {
                    hour: "numeric",
                  })
                }}
              </p>
              <img
                class="hourly-icon"
                :src="
                  `http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`
                "
                alt=""
              />
              <p class="hourly-temp">
                {{ Math.round(hourData.temp) }}&deg;
              </p>
            </div>
          </div>
        </div>
      </div>
  
      <hr class="separator" />
  
      <!-- Weekly Weather -->
      <div class="weekly-weather">
        <div class="weekly-header">
          <h2 class="weekly-title">7 Day Forecast</h2>
          <div
            v-for="day in weatherData.daily"
            :key="day.dt"
            class="daily-item"
          >
            <p class="day-name">
              {{
                new Date(day.dt * 1000).toLocaleDateString(
                  "en-us",
                  {
                    weekday: "long",
                  }
                )
              }}
            </p>
            <img
              class="daily-icon"
              :src="
                `http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`
              "
              alt=""
            />
            <div class="daily-temps">
              <p>H: {{ Math.round(day.temp.max) }}</p>
              <p>L: {{ Math.round(day.temp.min) }}</p>
            </div>
          </div>
        </div>
      </div>
  
      <div
        class="remove-city"
        @click="removeCity"
      >
        <i class="fa-solid fa-trash"></i>
        <p>Remove City</p>
      </div>
    </div>
  </template>
  
  <script setup>
  import axios from "axios";
  import { useRoute, useRouter } from "vue-router";
  
  const route = useRoute();
  const getWeatherData = async () => {
    try {
      const weatherData = await axios.get(
        `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
      );
  
      // cal current date & time
      const localOffset = new Date().getTimezoneOffset() * 60000;
      const utc = weatherData.data.current.dt * 1000 + localOffset;
      weatherData.data.currentTime =
        utc + 1000 * weatherData.data.timezone_offset;
  
      // cal hourly weather offset
      weatherData.data.hourly.forEach((hour) => {
        const utc = hour.dt * 1000 + localOffset;
        hour.currentTime =
          utc + 1000 * weatherData.data.timezone_offset;
      });
  
      return weatherData.data;
    } catch (err) {
      console.log(err);
    }
  };
  const weatherData = await getWeatherData();
  
  const router = useRouter();
  const removeCity = () => {
    const cities = JSON.parse(localStorage.getItem("savedCities"));
    const updatedCities = cities.filter(
      (city) => city.id !== route.query.id
    );
    localStorage.setItem(
      "savedCities",
      JSON.stringify(updatedCities)
    );
    router.push({
      name: "home",
    });
  };
  </script>
  
  <style scoped>
  .flex {
    display: flex;
  }
  
  .flex-col {
    flex-direction: column;
  }
  
  .flex-1 {
    flex: 1;
  }
  
  .items-center {
    align-items: center;
  }
  
  .text-white {
    color: #fff;
  }
  
  .bg-weather-secondary {
    background-color: #A9A9A9; /* Warna latar belakang banner */
  }
  
  .weather-overview {
    text-align: center;
    margin-bottom: 24px; /* Jarak bawah untuk bagian weather overview */
  }
  
  .city-name {
    font-size: 2rem; /* Ukuran teks nama kota */
    font-weight: 700; /* Ketebalan teks nama kota */
    margin-bottom: 8px; /* Jarak bawah untuk nama kota */
  }
  
  .current-time {
    font-size: 1rem; /* Ukuran teks waktu saat ini */
    margin-bottom: 8px; /* Jarak bawah untuk waktu saat ini */
  }
  
  .current-temp {
    font-size: 3rem; /* Ukuran teks suhu saat ini */
    font-weight: 700; /* Ketebalan teks suhu saat ini */
    margin-bottom: 8px; /* Jarak bawah untuk suhu saat ini */
  }
  
  .feels-like {
    font-size: 1rem; /* Ukuran teks feels like */
    margin-bottom: 8px; /* Jarak bawah untuk feels like */
  }
  
  .weather-description {
    text-transform: capitalize; /* Memformat teks deskripsi cuaca */
    margin-bottom: 16px; /* Jarak bawah untuk deskripsi cuaca */
  }
  
  .weather-icon {
    width: 100px; /* Lebar ikon cuaca */
    height: 100px; /* Tinggi ikon cuaca */
    margin: 0 auto; /* Posisi ikon cuaca di tengah */
    display: block; /* Mengubah ikon cuaca menjadi blok */
  }
  
  .separator {
    border: none; /* Tidak ada garis pemisah */
    height: 1px; /* Tinggi garis pemisah */
    background-color: #d1d5db; /* Warna garis pemisah */
    margin: 24px 0; /* Jarak atas dan bawah untuk garis pemisah */
  }
  
  .hourly-weather {
    margin-bottom: 24px; /* Jarak bawah untuk bagian hourly weather */
  }
  
  .hourly-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 16px; /* Jarak bawah untuk header hourly weather */
  }
  
  .hourly-title {
    font-size: 1.5rem; /* Ukuran teks judul hourly weather */
    font-weight: 700; /* Ketebalan teks judul hourly weather */
  }
  
  .hourly-list {
    display: flex;
    overflow-x: auto; /* Scroll horizontal untuk daftar jam */
  }
  
  .hourly-item {
    flex: 0 0 auto; /* Item hourly weather tidak fleksibel */
    text-align: center;
    margin-right: 16px; /* Jarak kanan antar item hourly weather */
  }
  
  .hour {
    font-size: 0.875rem; /* Ukuran teks jam */
    margin-bottom: 4px; /* Jarak bawah untuk jam */
  }
  
  .hourly-icon {
    width: 50px; /* Lebar ikon hourly weather */
    height: 50px; /* Tinggi ikon hourly weather */
    margin: 0 auto; /* Posisi ikon hourly weather di tengah */
    display: block; /* Mengubah ikon hourly weather menjadi blok */
  }
  
  .hourly-temp {
    font-size: 1rem; /* Ukuran teks suhu hourly */
    font-weight: 700; /* Ketebalan teks suhu hourly */
  }
  
  .weekly-weather {
    margin-bottom: 24px; /* Jarak bawah untuk bagian weekly weather */
  }
  
  .weekly-header {
    margin-bottom: 16px; /* Jarak bawah untuk header weekly weather */
  }
  
  .weekly-title {
    font-size: 1.5rem; /* Ukuran teks judul weekly weather */
    font-weight: 700; /* Ketebalan teks judul weekly weather */
  }
  
  .daily-item {
    display: flex;
    align-items: center;
    margin-bottom: 12px; /* Jarak bawah untuk item daily weather */
  }
  
  .day-name {
    flex: 1; /* Item nama hari memenuhi fleksibilitas */
    font-size: 1rem; /* Ukuran teks nama hari */
    font-weight: 700; /* Ketebalan teks nama hari */
  }
  
  .daily-icon {
    width: 40px; /* Lebar ikon daily weather */
    height: 40px; /* Tinggi ikon daily weather */
    margin-right: 8px; /* Jarak kanan untuk ikon daily weather */
  }
  
  .daily-temps {
    display: flex;
  }
  
  .remove-city {
    cursor: pointer; /* Ubah kursor menjadi tanda tangan saat dihover */
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 12px; /* Padding untuk tombol remove city */
    background-color: #ef4444; /* Warna latar belakang tombol remove city */
    color: #fff; /* Warna teks tombol remove city */
    border-radius: 8px; /* Sudut bulat tombol remove city */
  }
  
  .remove-city:hover {
    background-color: #dc2626; /* Warna latar belakang tombol remove city saat dihover */
  }
  
  .remove-city i {
    margin-right: 8px; /* Jarak kanan untuk ikon trash */
  }
  
  .remove-city p {
    font-weight: 700; /* Ketebalan teks tombol remove city */
  }
  </style>
  