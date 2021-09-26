<template>
  <div class="line-chart container">
    
    <LineChart></LineChart>
  </div>
</template>
<script>
// @ is an alias to /src
import Vue from "vue";
import vSelect from "vue-select";
Vue.component("v-select", vSelect);
import "vue-select/dist/vue-select.css";

import "simple-scrollbar/simple-scrollbar.css";
import LineChart from "../components/lineChart";

export default {
  name: "Home",
  components: {
    LineChart,
  },
  data() {
    return {
      value: null,
      dimensionSelected: "Product",
      measureSelected: 'Cost',
      options: [
        "list",
        "of",
        "options",
        "options2",
        "options3",
        "options4",
        "options5",
        "options6",
      ],
    };
  },
  methods: {},
  mounted() {},
};
</script>
<style lang="scss">
.ss-scroll {
  background-color: red;
  opacity: 1;
}

// .multiselect__content-wrapper {
//   height: 200px;
// }

.test .vs__dropdown-menu {
  //ul
  height: 200px;
  overflow: hidden;
}
.line-chart {
  padding: 40px 0;
}

</style>
