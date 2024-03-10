<script>
import { ref } from 'vue'

export default {
  props: {
    msg: String,
  },
  data() {
    return {
      arceau: "",
    };
  },
  methods: {
    fetchData() {
      fetch('https://data.strasbourg.eu/api/explore/v2.1/catalog/datasets/stationnement-velo/records?select=*&where=within_distance(geo_point_2d%2C%20geom%27POINT(7.738085938426863%2048.52642213347295)%27%2C%200.01km)%20&limit=20',
        {
          method: "GET"
        })
        .then((response) => {
          response.json().then((data) => {
            this.arceau = data[0];
          });
        })
        .catch((err) => {
          console.log(err);
        })
    },
  },
};

</script>

<template>
  <h1>Où garer mon vélo</h1>
  <button @click="getData">Touver des arceaux</button>
  <p v-if="arceau">{{ arceau }}</p>
</template>

<style scoped>
button {
  padding: 12px 32px;
  font-size: 16px;
  border-radius: 8px;
}
</style>
