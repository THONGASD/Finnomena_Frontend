<template>
  <div class="hello">
    <v-card>
      <v-card-title>
        Nutrition
        <v-spacer></v-spacer>
        <v-select
          :items="fundRangeItems"
          label="Solo field"
          solo
          v-model="fundRange"
          @change="getData()"
          hide-details
        ></v-select>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          single-line
          hide-details
        ></v-text-field>
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="fundsData"
        :search="search"
      ></v-data-table>
    </v-card>
  </div>
</template>

<script>
/* eslint-disable */
const axios = require("axios");
export default {
  name: "HelloWorld",

  mounted() {
    this.getData();
  },
  data() {
    return {
      fundsData: [],
      search: "",
      fundRangeItems: ["1D", "1W", "1M", "1Y"],
      fundRange: "1D",
      headers: [
        {
          text: "ชื่อกองทุน",

          value: "thailand_fund_code",
        },

        { text: "ผลตอบแทน", value: "nav_return" },
        { text: "ราคา", value: "nav" },
        { text: "อัพเดตเมื่อ", value: "nav_date" },
      ],
    };
  },
  methods: {
    async getData() {
      const { data } = await axios(
        `http://localhost:3000/${this.fundRange}`
      );

      this.fundsData = data.data;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
