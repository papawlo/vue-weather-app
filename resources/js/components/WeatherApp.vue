<template>
  <div class="text-white mb-8">
    <div class="placesinput text-gray-800">
      <input
        type="search"
        id="address"
        class="w-full"
        placeholder="Where are we going?"
      />

      <!-- <p>Selected: <strong id="address-value">none</strong></p> -->
    </div>
    <div
      class="weather-container font-sans w-128 max-w-lg overflow-hidden bg-gray-900 shadow-lg rounded-lg mt-4"
    >
      <div class="current-weather flex items-center justify-between px-6 py-8">
        <div class="flex items-center">
          <div>
            <div class="text-6xl font-semibold">
              {{ currentTemperature.actual }}°C
            </div>
            <div>Fells like {{ currentTemperature.feels }}°C</div>
          </div>
          <div class="mx-5">
            <div class="font-semibold">
              {{ currentTemperature.summary }}
            </div>
            <div>{{ location.name }}</div>
          </div>
        </div>
        <div>
          <img
            v-bind:src="
              'http://openweathermap.org/img/wn/' +
                currentTemperature.icon +
                '@2x.png'
            "
          />
        </div>
      </div>
      <!-- end current-weather -->
      <div class="future-weather text-sm bg-gray-800 px-6 py-8 overflow-hidden">
        <div
          v-for="(day, index) in daily"
          :key="day.time"
          class="flex items-center"
          :class="{ 'mt-8': index > 0 }"
        >
          <div class="w-1/6 text-lg text-gray-200">
            {{ toDayOfWeek(day.dt) }}
          </div>
          <div class="w-4/6 px-4 flex items-center">
            <div>
              <img
                v-bind:src="
                  'http://openweathermap.org/img/wn/' +
                    day.weather[0].icon +
                    '@2x.png'
                "
                width="60"
              />
            </div>
            <div class="ml-3">{{ day.weather[0].main }}</div>
          </div>
          <div class="w-1/6 text-right">
            <div>Max {{ Math.round(day.temp.max) }}°C</div>
            <div>Min {{ Math.round(day.temp.min) }}°C</div>
          </div>
        </div>
        <!-- end future-item -->
      </div>
      <!-- /.future-weather -->
    </div>
    <!-- end weather-container-->
  </div>
</template>

<script>
export default {
  mounted() {

    this.fetchData();

    var placesAutocomplete = places({
      appId: "pl6TOW1N35VJ",
      apiKey: "1cac0d4fe76b6c481ce70b2d5eb4facd",
      container: document.querySelector("#address")
    }).configure({
      type: "city",
      aroundLatLngViaIP: false
    });

    // var $address = document.querySelector("#address-value");
    placesAutocomplete.on("change", (e)=> {
      // console.log(e)
      // $address.textContent = e.suggestion.value;

      this.location.name = `${e.suggestion.name}, ${e.suggestion.country}`;
      this.location.lat = e.suggestion.latlng.lat;
      this.location.lng = e.suggestion.latlng.lng;
    });

    // placesAutocomplete.on("clear", function() {
    //   $address.textContent = "none";
    // });

  },
  watch:{
    location:{
      handler(newValue, oldValue){
        this.fetchData()
      },
      deep:true
    }
  },
  data() {
    return {
      currentTemperature: {
        actual: "",
        feels: "",
        summary: "",
        icon: "" //http://openweathermap.org/img/wn/10d@2x.png
      },
      daily: [],
      location: {
        name: "Dublin, Ireland",
        lat: 53.3498,
        lng: 6.2603
      }
    };
  },
  methods: {
    fetchData() {
      fetch(`/api/weather?lat=${this.location.lat}&lng=${this.location.lng}`)
        .then(response => response.json())
        .then(data => {
          console.log(data);
          this.currentTemperature.actual = Math.round(data.current.temp);
          this.currentTemperature.feels = Math.round(data.current.feels_like);
          this.currentTemperature.summary = data.current.weather[0].main;
          this.currentTemperature.icon = data.current.weather[0].icon;

          // this.$nextTick()=>{

          // }

          this.daily = data.daily;
        });
    },
    toDayOfWeek(timestamp) {
      const newDate = new Date(timestamp * 1000);
      const days = ["SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"];

      return days[newDate.getDay()];
    }
  }
};
</script>
