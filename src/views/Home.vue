<template>
  <div>
    <v-container grid-list-xs>
      <v-card class="mt-4">
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
        <v-container>
          <div class="d-flex">
            <v-col sm="12" md="6" lg="6">
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
            <v-col sm="12" md="6" lg="6">
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
            <template v-slot:item.action="{ item }">
              <!-- {{ item.nav_date | moment }} -->
              <v-btn
                color="success"
                @click="showFundDetail(item.thailand_fund_code)"
                :disabled="dataIsLoad"
                >รายละเอียดกองทุน</v-btn
              >
            </template>
          </v-data-table>
        </v-container>
      </v-card>
      <template v-if="dataIsLoad">
        <v-row class="mt-4">
          <v-col cols="6" md="6" v-for="i in 4" :key="'loading-' + i">
            <v-skeleton-loader
              v-bind="attrs"
              type="article, actions"
            ></v-skeleton-loader>

            <v-skeleton-loader
              v-bind="attrs"
              type="table-heading, list-item-two-line, image, table-tfoot"
            ></v-skeleton-loader>
          </v-col>
        </v-row>
      </template>
      <template v-else>
        <div v-if="detail_status">
          <h2>รายละเอียดกองทุน</h2>
          <v-row>
            <v-col md="4" sm="12" lg="6">
              <v-card class="mt-5">
                <v-card-text>
                  <h2 class="mt-0">
                    {{ fund_detail.proj_abbr_name }}
                  </h2>
                  <small>
                    {{ fund_detail.proj_name_th }} ({{
                      fund_detail.proj_name_en
                    }})
                  </small>
                  <table id="fund-detail-table">
                    <tr>
                      <th>สถานะกองทุน</th>
                      <td>{{ this.fund_status[fund_detail.fund_status] }}</td>
                    </tr>
                    <tr>
                      <th>ประเภทกองทุน</th>
                      <td>
                        {{ fund_detail.policy.policy_desc }}
                      </td>
                    </tr>
                    <tr>
                      <th>ความเสี่ยงของกองทุน</th>
                      <td>
                        {{ risk_assign.risk_spectrum }}
                      </td>
                    </tr>
                    <tr>
                      <th>วันที่ลงทะเบียน</th>
                      <td>{{ fund_detail.regis_date }}</td>
                    </tr>
                    <tr>
                      <th>Feeder Fund</th>
                      <td>
                        <a :href="fund_detail.feeder_fund.feeder_fund_link">{{
                          fund_detail.feeder_fund.main_feeder_fund
                        }}</a>
                      </td>
                    </tr>
                  </table>
                  <v-icon small>mdi-clock-outline</v-icon>
                  {{ fund_detail.last_upd_date | moment }}
                </v-card-text>
              </v-card>
            </v-col>

            <v-col md="4" sm="12" lg="6">
              <v-card class="mt-5">
                <v-card-text>
                  <h2 class="mt-0">ปัจจัยความเสี่ยงที่สำคัญ</h2>
                  <table id="risk-table">
                    <thead>
                      <th>กลุ่มความเสี่ยง</th>
                      <th>คำอธิบาย</th>
                    </thead>
                    <tbody>
                      <tr v-for="(risk, i) of risk_data" :key="'risk_row_' + i">
                        <td>{{ risk.group_code_desc }}</td>
                        <td>{{ risk.code_desc }}</td>
                      </tr>
                    </tbody>
                  </table>
                </v-card-text>
              </v-card>
            </v-col>
            <v-col md="6" sm="12" lg="6">
              <v-card>
                <v-card-text>
                  <h2 class="mt-0">ผู้เกี่ยวข้องกับกองทุน</h2>
                  <v-list-item
                    two-line
                    v-for="(related, i) of related_data"
                    :key="'related_' + i"
                  >
                    <v-list-item-content>
                      <v-list-item-title>{{
                        related.entity_name
                      }}</v-list-item-title>
                      <v-list-item-subtitle>
                        {{ related.entity_type }}
                      </v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                </v-card-text>
              </v-card>
            </v-col>
            <v-col sm="12" md="6" lg="6">
              <v-card>
                <v-card-title primary-title>
                  สัดส่วนประเภททรัพย์สินที่ลงทุนของกองทุน
                </v-card-title>
                <v-card-text>
                  <div class="container-chart">
                    <Piechart :ChartData="chart_data" />
                  </div>
                </v-card-text>
              </v-card>
            </v-col>
            <v-col md="12" sm="12" lg="12">
              <v-card class="mt-5">
                <v-card-text>
                  <h2 class="mt-0">
                    ความเหมาะสมกับผู้ลงทุนและความเสี่ยงของกองทุน
                  </h2>

                  <h4>รายละเอียดบรรยายความเสี่ยงของกองทุนรวม</h4>
                  <p v-html="risk_assign.risk_spectrum_desc"></p>
                  <h4>กองทุนนี้เหมาะกับใคร</h4>
                  <p v-html="risk_assign.fund_suitable_desc"></p>
                  <h4>กองทุนนี้ไม่เหมาะกับใคร</h4>
                  <p v-html="risk_assign.fund_not_suitable_desc"></p>
                  <h4><span id="note">คำเตือนที่สำคัญ</span></h4>
                  <p v-html="risk_assign.important_notice"></p>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </div>
        <div v-else>
          <v-card class="mt-2 items-center" min-height="150">
            ไม่พบข้อมูลกองทุน
          </v-card>
        </div>
      </template>
    </v-container>
  </div>
</template>

<script>
/* eslint-disable */
const moment = require("moment");
const axios = require("axios");
import Piechart from "../components/Piechart.vue";
export default {
  name: "Home",
  components: { Piechart },
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
          width: "15%",
        },

        {
          text: "ชื่อกองทุน",
          value: "thailand_fund_code",
        },

        { text: "ผลตอบแทน %", value: "nav_return" },
        { text: "ราคา", value: "nav" },
        { text: "อัพเดตเมื่อ", value: "nav_date" },
        { text: "", value: "action" },
      ],
      fund_detail: { fund_status: "" },
      fund_status: {
        SE: "อนุมัติ(อยู่ระหว่าง Filing)",
        EX: "หมดเวลาเสนอขาย",
        RG: "จดทะเบียน",
        CA: "เลิกโครงการ",
        LI: "จดทะเบียนเลิก",
      },
      chart_data: {},
      risk_data: [],
      risk_assign: {},
      detail_status: false,
      related_data: [],
      dataIsLoad: true,
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
      this.showFundDetail(data.data[0].thailand_fund_code);
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
        let dataObj = this.sortGroupData(RMF_group.concat(LTF_group));
        this.funds_data.inUse = dataObj;
      } else if (checkRMFGroup && checkSSFGroup) {
        let dataObj = this.sortGroupData(RMF_group.concat(SSF_group));
        this.funds_data.inUse = dataObj;
      } else if (checkLTFGroup && checkSSFGroup) {
        let dataObj = this.sortGroupData(LTF_group.concat(SSF_group));
        this.funds_data.inUse = dataObj;
      } else if (checkLTFGroup) {
        let dataObj = this.sortGroupData(LTF_group);
        this.funds_data.inUse = dataObj;
      } else if (checkSSFGroup) {
        let dataObj = this.sortGroupData(SSF_group);
        this.funds_data.inUse = dataObj;
      } else if (checkRMFGroup) {
        let dataObj = this.sortGroupData(RMF_group);
        this.funds_data.inUse = dataObj;
      }
    },
    async showFundDetail(fund_name) {
      try {
        this.dataIsLoad = true;
        const response = await axios(
          `http://localhost:3000/fund/detail/${fund_name}`
        );

        var { data } = response.data;

        this.fund_detail = data.daily;
        this.chart_data = data.assets;
        this.risk_data = data.risk;
        this.related_data = data.related;
        this.risk_assign = data.risk_assessment;
        this.detail_status = true;
        this.dataIsLoad = false;
      } catch (error) {
        console.log(error.response);
        this.detail_status = false;
        this.dataIsLoad = false;
      }
    },

    sortGroupData(data) {
      return data.sort((a, b) => b.nav_return - a.nav_return);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h2 {
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
table#risk-table {
  border: 1px solid;
  width: 100%;
  border-collapse: collapse;
  margin-top: 0.5rem;
}
table#risk-table td,
table#risk-table th {
  border: 1px solid black;

  padding: 10px 5px;
}
#note {
  color: red;
}
#note::before {
  content: "** ";
}
.v-card {
  overflow: auto;
  min-height: 250px;
}
.items-center {
  display: flex;
  align-items: center;
  justify-content: center;
}
table#fund-detail-table {
  width: 100%;
}
table#fund-detail-table th {
  text-align: left;
}
</style>
