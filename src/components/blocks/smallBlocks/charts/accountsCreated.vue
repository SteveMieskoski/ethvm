<template>
  <div id="GraphsLineChart" class="line-chart">

     <vue-chart type="line" :data="chartData" 
                            :options="chartOptions" 
                            :redraw="redraw" 
                            :chartTitle="newTitle"
                            :chartDescription="newDescription"
                            unfilled="true"></vue-chart>

  </div>
</template>

<script lang="ts">
import Vue from "vue";
import sEvents from "@/configs/socketEvents.json";
import BN from "bignumber.js";
import ethUnits from "ethereumjs-units";

/* Time Variables: */
const STATES = {
  BEGIN: "beginning",
  YEAR: "year",
  MONTH: "month",
  DAY: "day"
};
const DES = {
  BEGIN: "Number of accounts created in Ethereum blockchain since the ",
  OTHER: "Number of accounts created in Ethereum blockchain in last "
};
let currentState = STATES.DAY;
let stateChanged = false;

/* Chart Details: */

let title = "Accounts Growth";
let description = DES.OTHER + currentState;
let MAX_ITEMS = 10;
let lineOptions = {
  title: {
    text: "Accounts Growth",
    lineHeight: 1
  },
  responsive: true,
  scales: {
    yAxes: [
      {
        position: "left",
        ticks: {
          beginAtZero: true
        },
        gridLines: {
          color: "rgba(0, 0, 0, 0)"
        },
        scaleLabel: {
          display: true,
          labelString: "Average Size"
        }
      }
    ],
    xAxes: [
      {
        type: "time",
        distribution: "series",
        ticks: {
          source: "labels"
        }
      }
    ]
  },

  scaleShowLabels: false
};
export default Vue.extend({
  name: "BarChart",
  data: () => ({
    chartData: {},
    chartOptions: lineOptions,
    redraw: false,
    newTitle: title,
    newDescription: description
  }),
  created() {

         


    // this.$eventHub.$on(sEvents.pastBlocksR, () => {
    //   console.log(this.initData);
    //   this.chartData = this.initData;
    //   this.redraw = true;
    // });
    // this.$eventHub.$on(sEvents.newBlock, _block => {
    //   //console.log(_block)
    //   // if (this.chartData.datasets[0]) {
    //   //   this.redraw = false
    //   //   if (!_block.getIsUncle()) {
    //   //     let _tempD = _block.getStats()
    //   //     this.chartData.labels.push(_block.getNumber().toNumber())
    //   //     this.chartData.labels.shift()
    //   //     this.chartData.datasets[0].data.push(ethUnits.convert(new BN(_tempD.avgTxFees).toFixed(), 'wei', 'eth').substr(0, 8))
    //   //     this.chartData.datasets[0].data.shift()
    //   //     this.chartData.datasets[1].data.push(ethUnits.convert(new BN(_tempD.avgGasPrice).toFixed(), 'wei', 'gwei').substr(0, 5))
    //   //     this.chartData.datasets[1].data.shift()
    //   //   }
    //   // }
    // });


     
  },
  beforeDestroy() {
    
  },
  computed: {
    initData() {
      let data = {
        labels: [],
        avgFees: [],
        points:[],
        avgPrice: []
      };

        this.$socket.emit('getChartsData', (err, result) => {
      console.log(err, result)
      if (!err && result) {
        result.forEach( function(block) { 
           data.points.push({
                "x": block.timestamp,
              "y": block.accounts.length
              })
           data.labels.push(block.timestamp)

        
        } );
        }

 
    })
      return {
        labels: data.labels,
        datasets: [
          {
            label: "Accounts Number",
            backgroundColor: "#20c0c7",
            data: data.points,
            borderColor: "#20c0c7",
            fill: false
          }
        ]
      };
    },

    /*Method to change description string: */
    changeDescription() {
      if (stateChanged && currentState == STATES.BEGIN) {
        description = DES.BEGIN + currentState;
      } else if (stateChanged && currentState != STATES.BEGIN) {
        description = DES.OTHER + currentState;
      }
    }
  },
  mounted: function() {}
});
</script>


