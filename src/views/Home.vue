<template>
  <div>
    <v-container grid-list-xs>
      <v-card>
        <v-card-title>
          <h4>รายชื่อกองทุน</h4>
          <v-spacer></v-spacer>

          <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="ค้นหาชื่อกองทุน"
            single-line
            hide-details
          ></v-text-field>
        </v-card-title>
        <v-container grid-list-xs>
          <div>
            <h5>ประเภท</h5>
          </div>
          <div class="d-flex">
            <v-col sm="12" md="6" lg="4">
              <div class="d-flex">
                <v-checkbox
                  class="pr-6"
                  v-model="selected[0]"
                  label="LTF"
                  value="LTF"
                ></v-checkbox>

                <v-checkbox
                  class="pr-6"
                  v-model="selected[1]"
                  label="RMF"
                  value="RMF"
                ></v-checkbox>

                <v-checkbox
                  class="pr-6"
                  v-model="selected[2]"
                  label="SSF"
                  value="SSF"
                ></v-checkbox>
              </div>
            </v-col>
            <v-col sm="12" md="6" lg="4">
              <v-btn
                v-for="i of period_selectItems"
                :key="i"
                text
                @click="handlePeriodChange(i)"
                style="margin-top: 16px; padding: 4px"
                ><span
                  :style="
                    i == period_select ? 'border-bottom:1px solid blue' : ''
                  "
                  >{{ i }}</span
                ></v-btn
              >
            </v-col>
          </div>
          <!-- @click:row="handleRowClick" -->
          <v-data-table
            :loading="data_loading"
            :headers="headers"
            :items="funds_data.inUse"
            :search="search"
          >
            <template v-slot:item.fund_rank="{ item }">
              {{ funds_data.inUse.indexOf(item) + 1 }}
            </template>
            <template v-slot:item.nav_return="{ item }">
              {{ item.nav_return.toFixed(2) }}
            </template>
            <template v-slot:item.nav_date="{ item }">
              {{ item.nav_date | moment }}
            </template>
          </v-data-table>
        </v-container>
      </v-card>

      <!-- <v-card class="mt-5">
        <div class="tradingview-widget-container">
          <div id="tradingview_262c8"></div>
        </div>
      </v-card> -->
    </v-container>
  </div>
</template>

<script>
/* eslint-disable */
const moment = require("moment");
const axios = require("axios");
export default {
  name: "Home",

  mounted() {
    this.getData();
    // this.createChart();
  },
  data() {
    return {
      funds_data: {
        all: [],
        RMF_group: [],
        SSF_group: [],
        LTF_group: [],
        inUse: [],
      },
      selected: ["LTF", "RMF", "SSF"],
      search: "",
      data_loading: false,
      period_selectItems: ["1D", "1W", "1M", "1Y"],
      period_select: "1D",
      headers: [
        {
          text: "อันดับกองทุน",
          value: "fund_rank",
          width: "10%",
        },

        {
          text: "ชื่อกองทุน",
          value: "thailand_fund_code",
        },

        { text: "ผลตอบแทน %", value: "nav_return" },
        { text: "ราคา", value: "nav" },
        { text: "อัพเดตเมื่อ", value: "nav_date" },
      ],
    };
  },
  filters: {
    moment(val) {
      return moment(val).format("DD/MM/YYYY");
    },
  },
  watch: {
    selected(val) {
      this.filterFunds(val);
      return val;
    },
  },
  methods: {
    handleRowClick(vals) {
      this.createChart(vals.thailand_fund_code);
    },
    createChart(fundName = "NASDAQ:AAPL") {
      new TradingView.widget({
        width: "100%",
        height: 610,
        symbol: fundName,
        interval: "D",
        timezone: "Asia/Bangkok",
        theme: "light",
        style: "1",
        locale: "th_TH",
        toolbar_bg: "#f1f3f6",
        enable_publishing: false,
        allow_symbol_change: true,
        container_id: "tradingview_262c8",
      });
    },
    handlePeriodChange(periodSelect) {
      this.period_select = periodSelect;
      this.getData();
    },
    async getData() {
      this.data_loading = true;
      const { data } = await axios(
        `http://localhost:3000/${this.period_select}`
      );
      this.groupingFund(data.data);
      this.funds_data.all = data.data;
      this.filterFunds(this.selected);
      this.data_loading = false;
    },
    async groupingFund(data) {
      var LTF_group = [],
        SSF_group = [],
        RMF_group = [];
      for (const fund_list of data) {
        if (fund_list.thailand_fund_code.includes("RMF"))
          RMF_group.push(fund_list);
        else if (fund_list.thailand_fund_code.includes("SSF"))
          SSF_group.push(fund_list);
        else if (fund_list.thailand_fund_code.includes("LTF"))
          LTF_group.push(fund_list);
      }
      this.funds_data.RMF_group = RMF_group;
      this.funds_data.LTF_group = LTF_group;
      this.funds_data.SSF_group = SSF_group;
      this.funds_data.inUse = data;
    },
    filterFunds(val) {
      let checkLTFGroup = val.includes("LTF"),
        checkSSFGroup = val.includes("SSF"),
        checkRMFGroup = val.includes("RMF");
      let { RMF_group, LTF_group, SSF_group, inUse, all } = this.funds_data;
      if (checkLTFGroup && checkSSFGroup && checkRMFGroup) {
        this.funds_data.inUse = this.funds_data.all;
      } else if (checkRMFGroup && checkLTFGroup) {
        let dataObj = RMF_group.concat(LTF_group).sort(
          (a, b) => b.nav_return - a.nav_return
        );
        this.funds_data.inUse = dataObj;
      } else if (checkRMFGroup && checkSSFGroup) {
        let dataObj = RMF_group.concat(SSF_group).sort(
          (a, b) => b.nav_return - a.nav_return
        );
        this.funds_data.inUse = dataObj;
      } else if (checkLTFGroup && checkSSFGroup) {
        let dataObj = LTF_group.concat(SSF_group).sort(
          (a, b) => b.nav_return - a.nav_return
        );
        this.funds_data.inUse = dataObj.sort(
          (a, b) => b.nav_return - a.nav_return
        );
      } else if (checkLTFGroup) {
        let dataObj = LTF_group;
        this.funds_data.inUse = dataObj.sort(
          (a, b) => b.nav_return - a.nav_return
        );
      } else if (checkSSFGroup) {
        let dataObj = SSF_group;
        this.funds_data.inUse = dataObj.sort(
          (a, b) => b.nav_return - a.nav_return
        );
      } else if (checkRMFGroup) {
        let dataObj = RMF_group;
        this.funds_data.inUse = dataObj.sort(
          (a, b) => b.nav_return - a.nav_return
        );
      }
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
.d-flex {
  display: flex;
  flex-wrap: wrap;
}
</style>
