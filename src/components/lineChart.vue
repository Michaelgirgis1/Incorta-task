<template>
  <b-row>
    <b-col cols="3">
      <ul class="colums-list">
        <li
          v-for="item in columsList"
          :key="item.key"
          :data-type="item.function"
          @click="changeColum(item.name, item.function)"
        >
          {{ item.name }}
        </li>
      </ul>
    </b-col>
    <b-col cols="9">
      <div class="line-chart__header">
        <div class="line-chart__header__dimension">
          <span class="chart-axis-title">Dimension</span>
          <div class="line-chart__header__dimension__selected selected-item">
            <span v-if="dimensionSelected != ''">{{ dimensionSelected }}</span>
            <span v-else>{{ dimensionSelectedLabel }}</span>
            
            <button class="clear-btn" @click="clearDimension()">Clear</button>
          </div>
        </div>
        <div class="line-chart__header__measure">
          <span class="chart-axis-title">Measures</span>
          <div class="line-chart__header__measure__selected selected-item">
            <span v-if="measureSelected != ''">{{ measureSelected }}</span>
            <span v-else>{{ measureSelectedLabel }}</span>

            <button class="clear-btn" @click="clearMeasure()">Clear</button>
          </div>
        </div>
      </div>
      <apexchart
        v-if="
          series[0].data.length > 0 && chartOptions.xaxis.categories.length > 0
        "
        type="line"
        height="350"
        :options="chartOptions"
        :series="series"
      ></apexchart>
    </b-col>
  </b-row>
</template>
<script>
import Vue from "vue";
import VueApexCharts from "vue-apexcharts";
Vue.use(VueApexCharts);
import axios from "axios";

Vue.component("apexchart", VueApexCharts);
export default {
  name: "LineChart",
  components: {
    apexchart: VueApexCharts,
  },
  data() {
    return {
      dimensionSelected: "Product",
      measureSelected: "Cost",
      dimensionSelectedLabel:"Please Select Dimension",
      measureSelectedLabel: 'Please Select Measure',
      isClearItem: false,
      columsList: [],
      series: [
        {
          name: "Desktops",
          data: [],
        },
      ],
      chartOptions: {
        chart: {
          height: 350,
          type: "line",
          zoom: {
            enabled: false,
          },
        },
        dataLabels: {
          enabled: false,
        },
        stroke: {
          curve: "straight",
        },
        title: {
          text: "Product Trends by Month",
          align: "left",
        },
        grid: {
          row: {
            colors: ["#f3f3f3", "transparent"], // takes an array which will be repeated on columns
            opacity: 0.5,
          },
        },
        xaxis: {
          categories: [],
          title: {
            text: "Month",
          },
        },
        yaxis: {
          title: {
            text: "Temperature",
          },
        },
      },
    };
  },
  methods: {
    clearDimension() {
      this.dimensionSelected = '';
      this.series[0].data = [];
      this.isClearItem = true;


    },
    clearMeasure() {
      this.measureSelected = '';
      this.series[0].data = [];
      this.isClearItem = true;

    },
    getColumsList() {
      axios.get("https://plotter-task.herokuapp.com/columns").then((res) => {
        console.log(res.data);
        this.columsList = res.data;
      });
    },
    changeColum(name, func) {
      console.log(name, func);
      if(name != this.dimensionSelected  && name != this.measureSelected) {
        if (func == "dimension") {
          this.dimensionSelected = name;
        } else {
          this.measureSelected = name;
        }
        if(this.dimensionSelected != '' && this.measureSelected != '') {
          this.getChartData();
          if(!this.isClearItem) {
            this.getChartData();
           this.isClearItem = false;

          }
         }
      }
    },
    getChartData() {
      debugger;
      axios({
        url: "https://plotter-task.herokuapp.com/data",
        method: "POST",
        data: {
          measures: [`${this.measureSelected}`],
          dimension: `${this.dimensionSelected}`,
        },
        headers: {
          "Content-Type": "application/json",
        },
      }).then((res) => {
        console.log(res);
        let dimensionData = [],
          measureData = [];
        if (res.data[0].name == this.measureSelected) {
          measureData = res.data[0];
          dimensionData = res.data[1];
        } else {
          measureData = res.data[1];
          dimensionData = res.data[0];
        }
        setTimeout(() => {
          
          this.handleChartOption(dimensionData, measureData);
        }, 50);
      });
    },
    handleChartOption(dimensionData, measureData) {
      debugger;
      console.log(dimensionData, measureData);
      let categories = dimensionData.values.toString();

      categories = categories.split(",");
      this.chartOptions = {
        markers: {
       size: [4, 7],
          colors: undefined,
          strokeColors: "#fff",
          strokeWidth: 2,
          strokeOpacity: 0.9,
          strokeDashArray: 0,
          fillOpacity: 1,
          discrete: [],
          shape: "circle",
          radius: 2,
          offsetX: 0,
          offsetY: 0,
          onClick: undefined,
          onDblClick: undefined,
          showNullDataPoints: true,
          hover: {
            size: undefined,
            sizeOffset: 3,
          },
        },
        chart: {
          height: 350,
          type: "line",
          zoom: {
            enabled: false,
          },
        },
        dataLabels: {
          enabled: false,
        },
        stroke: {
          curve: "straight",
        },
        title: {
          text: `${dimensionData.name} Trends by ${measureData.name}`,
          align: "left",
        },
        grid: {
          row: {
            colors: ["#f3f3f3", "transparent"], // takes an array which will be repeated on columns
            opacity: 0.5,
          },
        },
        xaxis: {
          categories: `${dimensionData.values}`,
          title: {
            text: `${dimensionData.name}`,
          },
        },
        yaxis: {
          title: {
            text: `${measureData.name}`,
          },
        },
      };
      this.series = [
        {
          data: measureData.values,
        },
      ];
      console.log(
        "catelength",
        this.chartOptions.xaxis.categories,
        categories,
        this.chartOptions.xaxis.categories.length,
        dimensionData.values
      );
    },
  },

  mounted() {
    this.getColumsList();
    this.getChartData();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.line-chart__header {
  > div {
    display: flex;
    flex-direction: row;
    margin-bottom: 25px;
    align-items: center;
  }
}

.selected-item {
  display: flex;
  justify-content: space-between;
  width: 100%;
  border: 1px solid #ccc;
  border-radius: 7px;
  padding: 8px 15px;
  align-items: center;
}
.chart-axis-title {
  display: inline-flex;
  margin-right: 25px;
}

.clear-btn {
  border: 0;
  color: #fff;
  padding: 5px 15px;
  border-radius: 7px;
  background-color: #5932ab;
}
.line-chart__header__measure {
  .chart-axis-title {
    margin-right: 30px;
  }
}
.colums-list {
  list-style: none;
  li {
    text-align: left;
    margin-bottom: 10px;
    cursor: pointer;
  }
}
</style>
