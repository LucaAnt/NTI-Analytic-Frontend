<template>
  <div id="app">
    <v-app id="nukiTestAnalytic">
      <v-toolbar dark color="primary" app floating>
        <v-container grid-list-md text-xs-center fluid>
          <!----------------------------------------------FILTRI ----------------------------------------------->
          <v-layout row wrap>
            <!----------------------------------FILTER BUTTON----------------------------------------------->
            <v-flex xs1>
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <v-btn flat icon color="white" label="Apply Filter" @click="filter">
                    <v-icon>search</v-icon>
                  </v-btn>
                </template>
                <span>Apply Filter</span>
              </v-tooltip>
            </v-flex>

            <!----------------------------------SELEZIONE FILTRO ----------------------------------------------->
            <v-flex xs2>
              <v-select :items="listafiltri" label="Filter Criteria" v-model="selectedField"></v-select>
            </v-flex>
            <!----------------------------------RAFFINA FILTRO ----------------------------------------------->
            <v-flex xs1>
              <v-select :items="filterTuner" label="Tune Filter" v-model="selectedTune"></v-select>
            </v-flex>

            <!----------------------------------INPUT FILTRO --------------------------------------------------->
            <v-flex xs1>
              <v-text-field label="Value" v-model="insertedValue" @keypress="filterEnter"></v-text-field>
            </v-flex>

            <!----------------------------------TOGGLE DATE----------------------------------------------->
            <v-flex xs1>
              <v-switch v-model="dateFilterToggle" :label="`Date`"></v-switch>
            </v-flex>

            <!----------------------------------START DATE FILTER----------------------------------------------->
            <v-flex xs2>
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
            </v-flex>

            <!----------------------------------END DATE FILTER----------------------------------------------->
            <v-flex xs2>
              <v-menu
                ref="refDatestart"
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
            </v-flex>
          </v-layout>
        </v-container>
      </v-toolbar>

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
      </v-container>
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
      selectedTune: "contain"
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
      this.filteredTests = this.fetchedTests.filter(el => {
        return this.dateFilterToggle
          ? this.tunedFilter(el[this.selectedField]) &&
              el.dataOra > this.dateStartCalendarTextValue &&
              el.dataOra < this.dateEndCalendarTextValue
          : this.tunedFilter(el[this.selectedField]);
      });
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

