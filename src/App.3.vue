<template>
  <div id="app">
    <v-app id="nukiTestAnalytic">
      <v-toolbar color="white" app>
        <!----------------------------------------------FILTRI ----------------------------------------------->
        <!----------------------------------FILTER BUTTON----------------------------------------------->

        <v-btn flat icon color="indigo" label="Apply Filter" @click="filter">
          <v-icon>search</v-icon>
        </v-btn>

        <!----------------------------------SELEZIONE FILTRO ----------------------------------------------->

        <v-select :items="listafiltri" label="Filter Criteria" v-model="selectedField"></v-select>

        <!----------------------------------RAFFINA FILTRO ----------------------------------------------->

        <v-select :items="filterTuner" label="Tune Filter" v-model="selectedTune"></v-select>

        <!----------------------------------INPUT FILTRO --------------------------------------------------->

        <v-text-field label="Value" v-model="insertedValue" @keypress="filterEnter"></v-text-field>

        <!----------------------------------TOGGLE DATE----------------------------------------------->

        <v-switch v-model="dateFilterToggle" :label="`Date`"></v-switch>

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
            ></v-text-field>
          </template>
          <v-date-picker v-model="endDate" no-title></v-date-picker>
        </v-menu>
      </v-toolbar>

      <!----------------------------------HEIGHT SPACER------------------------------------------------------>
      <v-container>
        <v-layout row wrap>
          <v-flex xs12>
            <v-spacer></v-spacer>
          </v-flex>
        </v-layout>
      </v-container>

      <!----------------------------------DATA TABLE------------------------------------------------------>
      <v-container grid-list-md text-xs-center fluid>
        <v-layout row wrap>
          <v-flex xs12>
            <v-data-table :headers="headers" :items="filteredTests" class="elevation-1" light>
              <template v-slot:items="props">
                <td class="text-xs-left">{{ props.item.serial }}</td>
                <td class="text-xs-left">{{ props.item.finalResult }}</td>
                <td class="text-xs-left">{{ props.item.stationId }}</td>
                <td class="text-xs-left">{{ props.item.Operator }}</td>
                <td class="text-xs-left">{{ props.item.TestSocketIndex }}</td>
                <td class="text-xs-left">{{ props.item.dataOra }}</td>
                <td class="text-xs-left">{{ props.item.totalTime }}</td>
                <td class="text-xs-left">{{ props.item.numberOfResults }}</td>
              </template>
            </v-data-table>
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
        </div>
        <div class="text-md-center">
          <v-select :items="listafiltri" label="Extra Fields"></v-select>
        </div>
        <div class="text-md-right">
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
export default {
  name: "App",
  data: () => {
    return {
      dbUri: "http://127.0.0.1:8080/nookie",
      fetchedTests: [],
      filteredTests: [],
      //Table data
      headers: [
        { text: "Test Report Serial", value: "serial" },
        { text: "Test Result", value: "finalResult" },
        { text: "Station ID", value: "stationId" },
        { text: "Operator", value: "Operator" },
        { text: "Test Socket Index", value: "TestSocketIndex" },
        { text: "Test Date", value: "dataOra" },
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
      //Aggregated Analytics
      analyticValuesSize: "30px",
      totalPassed: 0,
      totalTerminated: 0,
      totalFailed: 0,
      passYeld: 0
    };
  },
  watch: {
    startDate: function() {
      this.dateStartCalendarTextValue = this.startDate;
    },
    endDate: function() {
      this.dateEndCalendarTextValue = this.endDate;
    }
  },
  methods: {
    filter: function() {
      //UUT array Filtering
      this.filteredTests = this.fetchedTests.filter(el => {
        return this.dateFilterToggle
          ? this.tunedFilter(el[this.selectedField]) &&
              el.dataOra > this.dateStartCalendarTextValue &&
              el.dataOra < this.dateEndCalendarTextValue
          : this.tunedFilter(el[this.selectedField]);
      });

      //Analytic data update
      this.updateAnalytics(this.filteredTests);
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
      this.totalPassed = this.totalFailed = this.totalTerminated = this.PassYeld = 0;
      filteredArray.forEach(element => {
        console.log(element);
        if (element.finalResult == "Passed") this.totalPassed++;
        if (element.finalResult == "Failed") this.totalFailed++;
        if (element.finalResult == "Terminated") this.totalTerminated++;
      });
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
    }
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

