<template>
  <div class="text-center mt-4">
    <img src="../assets/image.png" alt="covid-19" />
    <b-container>
      <div class="my-3">
        <b-card-group deck>
          <b-card bg-variant="light" class="text-left" style="border-bottom: 10px solid #6703fc">
            <b-card-sub-title class="mb-2">Infected</b-card-sub-title>
            <b-card-title v-if="status">
              <number
                class="transition"
                :from="numberFrom"
                :to="status.confirmed.value"
                :duration="duration"
              />
            </b-card-title>
            <small class="text-muted">{{ today }}</small>
            <b-card-text>Number of active cases COVID-19</b-card-text>
          </b-card>

          <b-card bg-variant="light" class="text-left" style="border-bottom: 10px solid #a5fc03">
            <b-card-sub-title class="mb-2">Recovered</b-card-sub-title>
            <b-card-title v-if="status">
              <number
                class="transition"
                :from="numberFrom"
                :to="status.recovered.value"
                :duration="duration"
              />
            </b-card-title>
            <small class="text-muted">{{ today }}</small>
            <b-card-text>Number of recoveries from COVID-19</b-card-text>
          </b-card>

          <b-card bg-variant="light" class="text-left" style="border-bottom: 10px solid #fc0341">
            <b-card-sub-title class="mb-2">Deaths</b-card-sub-title>
            <b-card-title v-if="status">
              <number
                class="transition"
                :from="numberFrom"
                :to="status.deaths.value"
                :duration="duration"
              />
            </b-card-title>
            <small class="text-muted">{{ today }}</small>
            <b-card-text>Number of deaths from COVID-19</b-card-text>
          </b-card>
        </b-card-group>
      </div>
      <b-row class="my-4">
        <b-col>
          <b-form-select v-model="selectedCountry" @change="getStatisticsData">
            <b-form-select-option
              v-for="(item, index) in countries"
              :key="index"
              :value="item.iso3"
            >{{ item.name }}</b-form-select-option>
          </b-form-select>
        </b-col>
      </b-row>
      <b-dd-text>Global Covid-19 Stats</b-dd-text>
      <area-chart :data="[
        { name: 'Infected' , data: infectedChartData}, 
        { name: 'Recovered' , data: recoveredChartData}, 
        { name: 'Deaths', data: deadChartData}
        ]"
        :colors="['#6703fc', '#a5fc03', '#fc0341']"
        xtitle="Date" ytitle="Count"
        label="Global Covid-19 Data"
        ></area-chart>
    </b-container>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "Home",
  data() {
    return {
      countries: {},
      selectedCountry: "IND",
      summary: null,
      confirmed: null,
      recovered: null,
      deaths: null,
      image: null,
      status: null,
      numberFrom: 0,
      duration: 5,
      dailyData: null,
      recoveredChartData: {},
      infectedChartData: {},
      deadChartData: {},
      today: null
    };
  },
  mounted() {
    this.getCountries();
    this.getSummary();
    this.getTotalConfirmedCases();
    this.getTotalRecovered();
    this.getTotalDeaths();
    this.getStatisticsData();
    this.dailySummary();
    this.getToday();
  },
  methods: {
    getCountries() {
      axios
        .get("https://covid19.mathdro.id/api/countries")
        .then(res => {
          this.countries = res.data.countries;
        })
        .catch(err => {
          console.log("Country Error: " + err);
        });
    },
    getSummary() {
      axios
        .get("https://covid19.mathdro.id/api/daily")
        .then(res => {
          this.summary = res.data;
        })
        .catch(err => {
          console.log("Summary Error: " + err);
        });
    },
    getTotalConfirmedCases() {
      axios
        .get("https://covid19.mathdro.id/api/confirmed")
        .then(res => {
          this.confirmed = res.data;
        })
        .catch(err => console.log(err));
    },
    getTotalRecovered() {
      axios
        .get("https://covid19.mathdro.id/api/recovered")
        .then(res => {
          this.recovered = res.data;
        })
        .catch(err => console.log(err));
    },
    getTotalDeaths() {
      axios
        .get("https://covid19.mathdro.id/api/deaths")
        .then(res => {
          this.deaths = res.data;
        })
        .catch(err => console.log(err));
    },
    getStatisticsData() {
      if (this.selectedCountry !== null) {
        axios
          .get(
            "https://covid19.mathdro.id/api/countries/" + this.selectedCountry
          )
          .then(res => {
            this.status = res.data;
          })
          .catch(err => console.log(err));
      }
    },
    dailySummary() {
      axios.get('https://covid19.mathdro.id/api/daily')
      .then(res => {
        this.dailyData = res.data;
        if(this.dailyData) {
          this.dailyData.forEach((item) => {
            this.infectedChartData[item.reportDate] = item.confirmed.total;
            this.recoveredChartData[item.reportDate] = item.recovered.total;
            this.deadChartData[item.reportDate] = item.deaths.total;
          })
        }
      }).catch(err => console.log(err));
    },
    getToday() {
      this.today = new Date().getDate() + '-' + new Date().getMonth() + "-" + new Date().getFullYear()
    }
  }
};
</script>

<style scoped>
* {
  font-family: "Roboto", sans-serif;
}
.transition {
  transition: all 0.3s ease-in;
}
</style>