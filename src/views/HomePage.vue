<script setup lang="ts">
import { ref } from 'vue';
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonItem, IonLabel, IonInput, IonButton } from '@ionic/vue';
import { Geolocation } from '@capacitor/geolocation';

const location = ref('');
const weather = ref<any>(null);
const apikey = 'NM37G4FXXYWFPPKWSFVJKFYJL';

// Función para convertir Fahrenheit a Celsius
const toCelsius = (fahrenheit: number) => {
  return ((fahrenheit - 32) * 5) / 9;
};

const getWeather = async () => {
  if (!location.value) return;
  try {
    const response = await fetch(`https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${location.value}?key=${apikey}`);
    const data = await response.json();
    console.log("Respuesta de la API:", data);

    if (!data.currentConditions || !data.days) {
      console.error("Datos incompletos o incorrectos de la API");
      return;
    }

    weather.value = {
      current: {
        temp: toCelsius(data.currentConditions.temp).toFixed(1), // Convertir a Celsius
        windSpeed: data.currentConditions.windspeed,
        precipitation: data.currentConditions.precip,
        condition: data.currentConditions.conditions
      },
      past24Hours: data.days[0].hours.map((hour: any) => ({
        datetime: hour.datetime,
        temp: toCelsius(hour.temp).toFixed(1), // Convertir a Celsius
        windspeed: hour.windspeed,
        precip: hour.precip,
        conditions: hour.conditions
      })),
      next24Hours: data.days[1].hours.map((hour: any) => ({
        datetime: hour.datetime,
        temp: toCelsius(hour.temp).toFixed(1), // Convertir a Celsius
        windspeed: hour.windspeed,
        precip: hour.precip,
        conditions: hour.conditions
      }))
    };
  } catch (error) {
    console.error("Error al obtener el clima:", error);
  }
};

const getCurrentLocationWeather = async () => {
  try {
    const position = await Geolocation.getCurrentPosition();
    const { latitude, longitude } = position.coords;
    const response = await fetch(`https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${latitude},${longitude}?key=${apikey}`);
    const data = await response.json();

    weather.value = {
      current: {
        temp: toCelsius(data.currentConditions.temp).toFixed(1), // Convertir a Celsius
        windSpeed: data.currentConditions.windspeed,
        precipitation: data.currentConditions.precip,
        condition: data.currentConditions.conditions
      },
      past24Hours: data.days[0].hours.map((hour: any) => ({
        datetime: hour.datetime,
        temp: toCelsius(hour.temp).toFixed(1), // Convertir a Celsius
        windspeed: hour.windspeed,
        precip: hour.precip,
        conditions: hour.conditions
      })),
      next24Hours: data.days[1].hours.map((hour: any) => ({
        datetime: hour.datetime,
        temp: toCelsius(hour.temp).toFixed(1), // Convertir a Celsius
        windspeed: hour.windspeed,
        precip: hour.precip,
        conditions: hour.conditions
      }))
    };
  } catch (error) {
    console.error("Error al obtener la ubicación actual:", error);
  }
};
</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar class="header-toolbar">
        <ion-title class="header-title">Aplicación del Clima</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true" class="ion-content">
      <div id="container">
        <ion-item class="input-item">
          <ion-label position="floating" class="input-label">Ingresa tu ubicación</ion-label>
          <ion-input v-model="location" placeholder="Ciudad, País" class="input-field"></ion-input>
        </ion-item>
        <ion-button expand="full" @click="getWeather" class="action-button">Obtener Clima</ion-button>
        <ion-button expand="full" @click="getCurrentLocationWeather" class="action-button">Usar Ubicación Actual</ion-button>

        <div v-if="weather" class="weather-container">
          <h2 class="section-title">Información Actual del Clima</h2>
          <div class="current-weather">
            <p><span class="weather-label">Temperatura:</span> {{ weather.current.temp }}°C</p>
            <p><span class="weather-label">Velocidad del Viento:</span> {{ weather.current.windSpeed }} km/h</p>
            <p><span class="weather-label">Probabilidad de Lluvia:</span> {{ weather.current.precipitation }}%</p>
            <p><span class="weather-label">Condición:</span> {{ weather.current.condition }}</p>
          </div>

          <h2 class="section-title">Últimas 24 Horas</h2>
          <div v-for="(hour, index) in weather.past24Hours" :key="index" class="weather-card">
            <p><span class="weather-label">Hora:</span> {{ hour.datetime }}</p>
            <p><span class="weather-label">Temperatura:</span> {{ hour.temp }}°C</p>
            <p><span class="weather-label">Velocidad del Viento:</span> {{ hour.windspeed }} km/h</p>
            <p><span class="weather-label">Probabilidad de Lluvia:</span> {{ hour.precip }}%</p>
            <p><span class="weather-label">Condición:</span> {{ hour.conditions }}</p>
          </div>

          <h2 class="section-title">Próximas 24 Horas</h2>
          <div v-for="(hour, index) in weather.next24Hours" :key="index" class="weather-card">
            <p><span class="weather-label">Hora:</span> {{ hour.datetime }}</p>
            <p><span class="weather-label">Temperatura:</span> {{ hour.temp }}°C</p>
            <p><span class="weather-label">Velocidad del Viento:</span> {{ hour.windspeed }} km/h</p>
            <p><span class="weather-label">Probabilidad de Lluvia:</span> {{ hour.precip }}%</p>
            <p><span class="weather-label">Condición:</span> {{ hour.conditions }}</p>
          </div>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<style scoped>
#container {
  text-align: center;
  padding: 20px;
  max-width: 600px;
  margin: 0 auto;
}


.header-toolbar {
  --background: #1e3a8a; 
  --color: white;
}

.header-title {
  font-size: 24px;
  font-weight: bold;
}

.input-item {
  margin-bottom: 20px;
  --background: #f3f4f6; 
  --border-radius: 12px;
}

.input-label {
  color: #4b5563; 
  font-weight: 500;
}

.input-field {
  --padding-start: 10px;
  --padding-end: 10px;
  margin-top: 10px;
  color: #4b5563; 
}

.action-button {
  --background: #3b82f6; 
  --color: white;
  --border-radius: 12px;
  margin-top: 10px;
  font-weight: bold;
}

.action-button:hover {
  --background: #2563eb;
}

.weather-container {
  margin-top: 20px;
}

.section-title {
  font-size: 20px;
  font-weight: bold;
  color: #1e3a8a; 
  margin-bottom: 15px;
}

.current-weather {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.weather-card {
  background-color: #ffffff;
  padding: 15px;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 15px;
}

.weather-label {
  font-weight: bold;
  color: #1e3a8a; 
}

p {
  font-size: 16px;
  color: #4b5563; 
  margin: 8px 0;
}
</style>