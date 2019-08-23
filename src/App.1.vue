<template>
  <div id="app">
    <v-app id="nukiTestAnalytic">
      <v-container grid-list-md text-xs-center fluid>
        <!----------------------------------------------FILTRI ----------------------------------------------->
        <v-layout row wrap>
          <!----------------------------------SELEZIONE FILTRO ----------------------------------------------->
          <v-flex xs3>
            <v-select :items="listafiltri" label="Filter Criteria"></v-select>
          </v-flex>

          <!----------------------------------INPUT FILTRO --------------------------------------------------->
          <v-flex xs3>
            <v-text-field label="Value"></v-text-field>
            <!--<v-select :items="filterTuner" label="Tune Research"></v-select>-->
          </v-flex>

          <!----------------------------------START DATE FILTER----------------------------------------------->
          <v-flex xs3></v-flex>

          <!----------------------------------END DATE FILTER----------------------------------------------->
          <v-flex xs3></v-flex>
        </v-layout>

        <!----------------------------------DATA TABLE------------------------------------------------------>
        <v-layout row wrap>
          <v-flex xs12>
            <v-data-table :headers="headers" :items="fetchedTests" class="elevation-1">
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
      //Table data
      headers: [
        { text: "Test Report Serial", value: "serial" },
        { text: "Test Result", value: "finalResult" },
        { text: "Station ID", value: "stationId" },
        { text: "Operator", value: "Operator" },
        { text: "Test Socket Index", value: "TestSocketIndex" },
        { text: "Test Date", value: "dataOra" },
        { text: "Total Time (seconds)", value: "totalTime" },
        { text: "Results counts", value: "numberOfResults" }
      ],
      //Date Picker Datas

      //Dropdown data & filter data
      listafiltri: [],
      filterTuner: ["=", "<", ">"]
    };
  },
  ///Lifecycle methods
  created: () => {
    console.log("TEST");
  },
  mounted: () => {
    fetch(this.dbUri)
      .then(response => {
        return response.json();
      })
      .then(json => {
        this.fetchedTests = json;
        console.log(this.fetchedTests);
        return;
      });
    this.headers.forEach(element => {
      this.listaFiltri.push(element.value.toUpperCase);
    });
  }
};
</script>
