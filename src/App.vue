<template>
  <div id="app">
    <v-app id="nukiTestAnalytic">
      <v-toolbar color="white">
        <!----------------------------------------------FILTRI ----------------------------------------------->
        <!----------------------------------FILTER BUTTON----------------------------------------------->

        <v-btn flat icon color="indigo" label="Apply Filter" @click="filter">
          <v-icon>search</v-icon>
        </v-btn>

        <!----------------------------------SELEZIONE FILTRO ----------------------------------------------->

        <v-select :items="listafiltri" label="Filter Criteria" v-model="selectedField"></v-select>
        <v-spacer></v-spacer>
        <!----------------------------------RAFFINA FILTRO ----------------------------------------------->

        <v-select :items="filterTuner" label="Tune Filter" v-model="selectedTune"></v-select>
        <v-spacer></v-spacer>
        <!----------------------------------INPUT FILTRO --------------------------------------------------->

        <v-text-field label="Value" v-model="insertedValue" @keypress="filterEnter"></v-text-field>
        <v-spacer></v-spacer>
        <!----------------------------------TOGGLE DATE----------------------------------------------->

        <v-switch v-model="dateFilterToggle" :label="`Date Filter`"></v-switch>

        <!----------------------------------START DATE FILTER----------------------------------------------->
        <v-menu
          ref="refDatestart"
          v-model="dateStartCalendarMenu"
          :close-on-content-click="true"
          :nudge-right="40"
          lazy
          transition="scale-transition"
          offset-y
          full-width
          max-width="290px"
          min-width="290px"
          v-if="dateFilterToggle"
        >
          <template v-slot:activator="{ on }">
            <v-text-field
              v-model="dateStartCalendarTextValue"
              label="Start Date"
              hint="YYYY-MM-DD format"
              persistent-hint
              prepend-icon="event"
              v-on="on"
              @keypress="filterEnter"
            ></v-text-field>
          </template>
          <v-date-picker v-model="startDate" no-title></v-date-picker>
        </v-menu>

        <!----------------------------------END DATE FILTER----------------------------------------------->
        <v-menu
          ref="refDateend"
          v-model="dateEndCalendarMenu"
          :close-on-content-click="true"
          :nudge-right="40"
          lazy
          transition="scale-transition"
          offset-y
          full-width
          max-width="290px"
          min-width="290px"
          v-if="dateFilterToggle"
        >
          <template v-slot:activator="{ on }">
            <v-text-field
              v-model="dateEndCalendarTextValue"
              label="End Date"
              hint="YYYY-MM-DD format"
              persistent-hint
              prepend-icon="event"
              v-on="on"
              @keypress="filterEnter"
            ></v-text-field>
          </template>
          <v-date-picker v-model="endDate" no-title></v-date-picker>
        </v-menu>
      </v-toolbar>

      <!----------------------------------DATA TABLE------------------------------------------------------>
      <v-container grid-list-md text-xs-center align-content-start fluid>
        <v-layout row wrap>
          <v-flex xs12>
            <v-tabs v-model="activeTab" color="cyan" dark slider-color="yellow">
              <v-tab :key="1" ripple>Plain Data</v-tab>
              <v-tab :key="2" ripple>Analytics</v-tab>
              <v-tab-item :key="1">
                <v-data-table :headers="headers" :items="filteredTests" class="elevation-1" light>
                  <template v-slot:items="props">
                    <td class="justify-center layout px-0">
                    <v-icon small class="mr-2" @click="downloadFile(props.item)">get_app</v-icon>
                    <v-icon small @click="downloadCsv(props.item)">picture_as_pdf</v-icon> 
                    </td>
                    <td class="text-xs-left">{{ props.item.AluPartNumber }}</td>
                    <td class="text-xs-left">{{ props.item.serial }}</td>
                    <td class="text-xs-left">{{ props.item.finalResult }}</td>
                    <td class="text-xs-left">{{ props.item.stationId }}</td>
                    <td class="text-xs-left">{{ props.item.Operator }}</td>
                    <td class="text-xs-left">{{ props.item.dateTimeStart }}</td>
                    <td class="text-xs-left">{{ props.item.dateTimeEnd }}</td>
                    <td class="text-xs-left">{{ props.item.totalTime }}</td>
                    <td class="text-xs-left">{{ props.item.numberOfResults }}</td>
                  </template>
                </v-data-table>
              </v-tab-item>
              <v-tab-item :key="2">
                <v-container>
                  <v-layout row>
                    <v-flex d-flex xs12>
                      <div id="chart">
                        <apexchart  ref="fpyChart" type=bar height=350 :options="chartOptions" :series="series" />
                      </div>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-tab-item>
            </v-tabs>
          </v-flex>
        </v-layout>
        <!----------------------------------FINE DATA TABLE------------------------------------------------------>
        <!----------------------------------HEIGHT SPACER------------------------------------------------------>
        <v-layout row wrap>
          <v-flex xs12>
            <v-spacer></v-spacer>
          </v-flex>
        </v-layout>
      </v-container>

      <!----------------------------------BOTTOM INFO BAR------------------------------------------------------>
      <v-bottom-nav :value="true" app color="white">
        <div class="text-md-left">
          <v-chip color="green" text-color="white">
            <v-avatar class="green darken-1">{{totalPassed}}</v-avatar>Passed
          </v-chip>
          <v-chip color="orange" text-color="white">
            <v-avatar class="orange darken-1">{{totalTerminated}}</v-avatar>Terminated
          </v-chip>
          <v-chip color="red" text-color="white">
            <v-avatar class="red darken-1">{{totalFailed}}</v-avatar>Failed
          </v-chip>
          <v-chip color="deep-orange accent-3" text-color="white">
            <v-avatar class="deep-orange accent-4">{{totalErrors}}</v-avatar>Errors
          </v-chip>
          <v-chip color="grey" text-color="white">
            <v-avatar class="grey darken-1">{{total}}</v-avatar>Total
          </v-chip>
        </div>

        <div class="text-md-right">
          <v-tooltip top>
            <template v-slot:activator="{ on }">
              <v-icon v-on="on" @click="downloadCsv()">dehaze</v-icon>
            </template>
            <span>Download CSV on filtered</span>
          </v-tooltip>
          <v-chip color="light-blue" text-color="white">
            <v-avatar class="light-blue darken-1">FPY</v-avatar>
            {{firstPassYeld}}%
          </v-chip>

          <v-chip color="teal" text-color="white">
            <v-avatar class="teal darken-1">AvgT</v-avatar>
            {{avgTestTime.toFixed(2)}}s
          </v-chip>

          <v-chip color="indigo right" text-color="white">
            <v-avatar class="indigo lighten-1">PY</v-avatar>
            {{passYeld}}%
          </v-chip>
          <v-chip color="purple right" text-color="white">
            <v-avatar :size="analyticValuesSize" class="purple lighten-1">Y</v-avatar>
            {{Yeld}}%
          </v-chip>
        </div>
      </v-bottom-nav>
    </v-app>
  </div>
</template>


<script>
import VueApexCharts from 'vue-apexcharts'

export default {
  name: "App",
  components: {
    apexchart: VueApexCharts
  },
  data: () => {
    return {
      dbUri: "http://127.0.0.1:8080/module_testing_reports/all/",
      fetchedTests: [],
      filteredTests: [],
      activeTab: 0,
      //Table data
      headers: [
        { text: "Options",align: 'left',sortable: false},
        { text: "AluPartNumber", value: "AluPartNumber" },
        { text: "Test Report Serial", value: "serial" },
        { text: "Test Result", value: "finalResult" },
        { text: "Station ID", value: "stationId" },
        { text: "Operator", value: "Operator" },
        { text: "Test Start Date", value: "dateTimeStart" },
        { text: "Test End Date", value: "dateTimeEnd" },
        { text: "Total Time (seconds)", value: "totalTime" },
        { text: "Results Counts", value: "numberOfResults" }
      ],
      //Toggle data
      dateFilterToggle: false,
      //Date Picker Datas Start
      startDate: null,
      dateStartCalendarMenu: false,
      dateStartCalendarTextValue: "",
      //Date Picker Datas Start
      endDate: null,
      dateEndCalendarMenu: false,
      dateEndCalendarTextValue: "",

      //Dropdown data & filter data
      listafiltri: [],
      filterTuner: ["contain", "=", "<", ">"],
      selectedField: "serial",
      insertedValue: "",
      selectedTune: "contain",

      //Analytics
      analyticValuesSize: "30px",
      total:0,
      totalPassed: 0,
      totalTerminated: 0,
      totalFailed: 0,
      totalErrors:0,
      passYeld: 0,
      Yeld:0,
      firstPassYeld:0,
      avgTestTime:0,
      avgBottom10:0,

      //CHARTS RELATED DATA
      series: [{
          name: 'FPY',
          data: [{x: '',y: 0}]
        }],
        chartOptions: {
          title: {
            text: 'First Pass Yeld on Filtered Reports',
            floating: true,
            offsetY: 0,
            align: 'center',
            style: {color: '#444'}
          },
          plotOptions: {
            bar: {
              horizontal: false,
              dataLabels: {position: 'top'}
            }
          },
          dataLabels: {
            enabled: true,
            formatter: function (val) {
              return val + "%";
            },
            offsetY: 0,
            style: {
              fontSize: '12px',
              colors: ["#304758"]
            }
          }
        }
        };
  },
  watch: {
    startDate: function() {
      this.dateStartCalendarTextValue = this.startDate;
    },
    endDate: function() {
      this.dateEndCalendarTextValue = this.endDate;
    },
    activeTab:function(){
    }
  },
  methods: {
    filter: function() {
      //UUT array Filtering
      this.filteredTests = this.fetchedTests.filter(el => {
        return this.dateFilterToggle
          ? this.tunedFilter(el[this.selectedField]) &&
              el.dateTimeStart > this.dateStartCalendarTextValue &&
              el.dateTimeStart < this.dateEndCalendarTextValue
          : this.tunedFilter(el[this.selectedField]);
      });

      //Analytic data update
      this.updateAnalytics(this.filteredTests);

      //Charts Update
      this.updateFpyChart();
    },
    tunedFilter: function(value) {
      switch (this.selectedTune) {
        case "contain":
          return value.toString().includes(this.insertedValue);
        case "=":
          return value == this.insertedValue;
        case "<":
          return value < this.insertedValue;
        case ">":
          return value > this.insertedValue;
        default:
          break;
      }
      return false;
    },
    filterEnter: function(event) {
      if (event.charCode == 13) this.filter.call();
    },
    updateAnalytics: function(filteredArray) {
      this.total=this.totalPassed = this.totalFailed = this.totalTerminated = this.totalErrors = this.PassYeld =this.avgTestTime=this.firstPassYeld=0;
      //let map = new Map(); FPY Map
      
      this.total = filteredArray.length;

      for (let i = 0; i < filteredArray.length; i++) 
      {
        const element = filteredArray[i];

        //Calcolo FPY su seriali univoci
        /*
        if(element.finalResult=="Passed"||element.finalResult=="Failed")
            if(map.has(element.serial)){
              
              let counter = (map.get(element.serial)).entriesCounter++;
              let tmpDate = (map.get(element.serial)).entry.dateTimeStart;
            
                  if(tmpDate>element.dateTimeStart)
                      map.set(element.serial,{entry:element,entriesCounter:counter});
              }else {
                    map.set(element.serial,{entry:element,entriesCounter:1});
              }
        */

        if (element.finalResult == "Passed") this.totalPassed++;
        if (element.finalResult == "Failed") this.totalFailed++;
        if (element.finalResult == "Terminated") this.totalTerminated++;
        if (element.finalResult == "Error") this.totalErrors++;
        if (element.finalResult=="Passed"||element.finalResult=="Failed"||element.finalResult=="Terminated")
          this.avgTestTime+=element.totalTime;
      }

      this.passYeld = (
        (this.totalPassed /
          (this.totalPassed + this.totalFailed + this.totalTerminated)) *
        100
      ).toFixed(2);

      this.Yeld = (
        (this.totalFailed /
          (this.totalPassed + this.totalFailed + this.totalTerminated)) *
        100
      ).toFixed(2);

      this.avgTestTime/=filteredArray.length;
      this.avgTestTime  = this.avgTestTime;

      //Calcolo FPY
      /*
      let totalUniqueFirstSerialPassed=0;
      let totalUniqueFirstSerialFailed=0;
      for (let  value of map.values() )
      {
          if(value.entry.finalResult=="Passed")
          {
            totalUniqueFirstSerialPassed++;
          }else if(value.entry.finalResult=="Failed"){
              totalUniqueFirstSerialFailed++;
          }
      }*/
      //FPY
      this.firstPassYeld = this.calculateFpyOnArray(filteredArray)
      
      //Tempo mediano sul 10% della parte inferiore dei valori che rientrano nella varianza
      let variance = this.calculateVariance(filteredArray,this.avgTestTime,filteredArray.length);
      console.log(variance);
      let topRange = this.avgTestTime+variance;
      let botRange = this.avgTestTime-variance;
      let tmpInVarianceReports=[];
      let avgBottom10=0;
      filteredArray.forEach(element => {
          if((element.totalTime>botRange)&&(element.totalTime<topRange))
            tmpInVarianceReports.push(element.totalTime);
          //console.log(botRange,element.totalTime,topRange);
      });

      tmpInVarianceReports = tmpInVarianceReports.sort();
      for (let index = 0; index < tmpInVarianceReports.length/10; index++) 
        avgBottom10+=tmpInVarianceReports[index];
      this.avgBottom10 = avgBottom10/ tmpInVarianceReports.length/10;

    }
  ,
  calculateVariance:function(filteredArray,avgTime,totalReports)
  {
    let variance=0;
    filteredArray.forEach(element => {
      variance+=Math.pow(element.totalTime-avgTime, 2);
    });
    return Math.sqrt(variance/totalReports);
  },
  updateFpyChart:function(){
    let minDate=this.filteredTests[0].dateTimeStart;
    let MaxDate=this.filteredTests[0].dateTimeStart;
    let FpyMap = new Map();

    this.filteredTests.forEach(element => {
      if (element.dateTimeStart<minDate)  
           minDate = element.dateTimeStart
      if (element.dateTimeStart>MaxDate)  
           MaxDate = element.dateTimeStart
    });

    minDate  = new Date(minDate);
    MaxDate  = new Date(MaxDate);

    for(let year=minDate.getFullYear();year<=MaxDate.getFullYear();year++)
    {
      let endingMonth;
      let startingMonth;

      if(year == MaxDate.getFullYear())
          endingMonth=MaxDate.getMonth();
      else
        endingMonth=11;
      
      if(year==minDate.getFullYear())
        startingMonth = minDate.getMonth();
      else
        startingMonth=0;

      for(let month=startingMonth;month<=endingMonth;month++)
      {
          let intTrueMonth =month+1;
          let dateStringKey=year+'-'+intTrueMonth;
          FpyMap.set(dateStringKey,new Array());    

          this.filteredTests.forEach(element => 
          {
            let tmpEntry  = new Date(element.dateTimeStart);
            if(tmpEntry.getFullYear()==year && tmpEntry.getMonth()==month)
            {
              let fpyMonthArray=FpyMap.get(dateStringKey);
              fpyMonthArray.push(element);
              FpyMap.set(dateStringKey,fpyMonthArray); 
            } 
          });
      }
    }
 
    this.series[0].data = [];
    FpyMap.forEach((value, key) => {

      this.series[0].data.push({x: key,y: this.calculateFpyOnArray(value)});

    });

    this.$refs.fpyChart.updateSeries();
  },
  calculateFpyOnArray: function(targetArray)
  {
      let map = new Map();
      

      targetArray.forEach(element => {
        //Map seriali univoci con indice il seriale e contenente come valore la coppia entry qty volte testate
        if(element.finalResult=="Passed"||element.finalResult=="Failed")
            if(map.has(element.serial)){
              
              let counter = (map.get(element.serial)).entriesCounter++;
              let tmpDate = (map.get(element.serial)).entry.dateTimeStart;
            
                  if(tmpDate>element.dateTimeStart)
                      map.set(element.serial,{entry:element,entriesCounter:counter});
              }else {
                    map.set(element.serial,{entry:element,entriesCounter:1});
              }
      });

      //Calcolo FPY
      let totalUniqueFirstSerialPassed=0;
      let totalUniqueFirstSerialFailed=0;
      for (let  value of map.values() )
      {
          if(value.entry.finalResult=="Passed")
          {
            totalUniqueFirstSerialPassed++;
          }else if(value.entry.finalResult=="Failed"){
              totalUniqueFirstSerialFailed++;
          }
      }
      
      let fpyToReturn = (totalUniqueFirstSerialPassed/(totalUniqueFirstSerialPassed+totalUniqueFirstSerialFailed)*100).toFixed(0);
      return ((fpyToReturn!=null)&&(!isNaN(fpyToReturn))?fpyToReturn:0);

  },
  downloadFile: function(entry) {
      console.log(entry.fileName);
      window.location.href = this.dbUri+"/download/"+entry.fileName;
    },
    downloadCsv: function() 
    {
      let csv

      for(let row = 0; row < this.filteredTests.length; row++){
          let keysAmount = Object.keys( this.filteredTests[row]).length;
          let keysCounter = 0;
          if(row === 0){
            for(let key in this.filteredTests[row]){
                csv += key + (keysCounter+1 < keysAmount ? ',' : '\r\n' )
                keysCounter++
            }
          }else{
            for(let key in this.filteredTests[row]){
                csv += this.filteredTests[row][key] + (keysCounter+1 < keysAmount ? ',' : '\r\n' )
                keysCounter++
            }
          }

          keysCounter = 0
      }
      
      let link = document.createElement('a')
      link.id = 'download-csv'
      link.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(csv));
      link.setAttribute('download', 'filteredTests.csv');
      document.body.appendChild(link)
      document.querySelector('#download-csv').click()
    },
    },
  ///Lifecycle methods
  mounted: function() {
    fetch(this.dbUri)
      .then(response => {
        return response.json();
      })
      .then(json => {
        this.fetchedTests = json;
        this.filteredTests = this.fetchedTests;
        this.updateAnalytics(this.filteredTests);
        this.updateFpyChart();
        return;
      });
    //Popola dinamicamente l'array della dropList in base ai campi dell'oggetto
    this.headers.forEach(element => {
      this.listafiltri.push(element.value);
    });

    //Inizializza i dati relativi al calendario
    this.startDate = new Date().toISOString().substr(0, 10);
    this.endDate = new Date().toISOString().substr(0, 10);
  }
};
</script>

<style>
tbody tr:nth-of-type(odd) {
  background-color: rgba(20, 139, 236, 0.089);
}
</style>

