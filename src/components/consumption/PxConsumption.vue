<template>
  <div class="app">
    <v-app>
      <v-main>
        <h2 class="text-center orange--text">CONSUMO</h2>
        <v-container>
          <v-row>
            <v-col lg="2" cols="5" sm="5">
              <v-menu
                ref="menu"
                v-model="menu"
                :close-on-content-click="false"
                :return-value.sync="date"
                transition="scale-transition"
                offset-y
                color="orange accent-3 lighten-1"
                max-width="290px"
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="date"
                    label="Fecha"
                    prepend-icon="mdi-calendar"
                    color="orange accent-3 lighten-1"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker v-model="date" type="month" no-title scrollable>
                  <v-spacer></v-spacer>
                  <v-btn text color="primary" @click="menu = false">
                    Cancel
                  </v-btn>
                  <v-btn text color="primary" @click="$refs.menu.save(date)">
                    OK
                  </v-btn>
                </v-date-picker>
              </v-menu>
              <p v-show="false">{{ submitDate }}</p>
            </v-col>
            <v-col>
              <v-btn tile color="success" @click="getReport()">
                <v-icon left>
                  mdi-pencil
                </v-icon>
                Descargar
              </v-btn>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-card elevation="2">
                <h5 style="color:#FF9B00;text-align: center;">
                  TOTAL MINUTOS <br />
                </h5>
                <h1 style="text-align: center;">
                  <span style="color:#4C4C4C;">{{ totalMinutes }}</span>
                </h1>
              </v-card>
            </v-col>
            <v-col>
              <h5 style="color:#FF9B00;text-align: center;">
                TOTAL <span style=""> GRABACIONES </span>
                <br /><!--color:#4C4C4C;-->
              </h5>
              <h1 style=";text-align: center;">
                <span style="color:#4C4C4C;">{{ totalCallRecords }}</span>
              </h1>
            </v-col>
            <v-col>
              <h5 style="color:#FF9B00;text-align: center;">
                GRABACIONES LEIDAS <br />
              </h5>
              <h1 style="text-align: center;">
                <span style="color:#4C4C4C;">{{ callRead }}</span>
              </h1>
            </v-col>
          </v-row>
        </v-container>
        <v-container class="pt-lg-12">
          <v-row>
            <br />
            <h3 class="ml-lg-10" style="color:#FF9B00;">MINUTOS</h3>
            <br />
            <br />
            <column-chart
              :colors="['#FF9B00', '#CC7C00']"
              :data="graficaMinutes"
            ></column-chart>
          </v-row>
          <v-row>
            <br />
            <h3 class="ml-lg-10 pt-lg-8 pb-lg-8" style="color:#FF9B00;">
              GRABACIONES
            </h3>
            <br />
            <br />
            <column-chart
              :colors="['#FF9B00']"
              :data="graficaCallRecords"
            ></column-chart>
          </v-row>
        </v-container>
      </v-main>
    </v-app>
  </div>
</template>
<script>
let currentUrl = window.location.pathname;
console.log("currenturl", currentUrl);
let url = `${process.env.VUE_APP_URLBACKEND}${currentUrl}`; //igsSerfinanzaCO/basephrases/
console.log("url", url);

export default {
  name: "PxConsumption",
  data() {
    return {
      date: new Date().toISOString().substr(0, 7),
      menu: false,
      fechas: [],
      totalMinutes: 0,
      totalCallRecords: 0,
      callRead: 0,
      graficaMinutes: [],
      graficaCallRecords: []
    };
  },
  created() {
    this.mostar();
  },
  computed: {
    submitDate() {
      const date = new Date(this.date).toISOString().substr(0, 7);

      console.log(date);
      this.mostar();
      return date;
    }
  },
  methods: {
    async mostar() {
      const response = await this.axios.get(
        url + `?yearMonthString=${this.date}`
      );
      //console.log('fecha peticion = ', this.date)
      this.fechas = response.data.body;
      //console.log('fechas = ', this.fechas);
      console.log("response.data.body", response.data.body);
      this.mostrarTotalMinutes(this.fechas);
      this.mostrarTotalCallRecords(this.fechas);
      this.mostrarCallRead(this.fechas);
      this.mostrargraficaMinutes(this.fechas);
      this.mostrargraficaCallRecords(this.fechas);
    },
    objectToCsv(data) {
      const csvRows = [];
      const headers = Object.keys(data[0]);
      csvRows.push(headers.join(","));

      //console.log('headers',csvRows)

      for (const row of data) {
        const values = headers.map(header => {
          const scaped = ("" + row[header]).replace(/"/g, '\\"');
          return `"${scaped}"`;
        });
        csvRows.push(values.join(","));
      }

      return csvRows.join("\n");
    },
    download(data) {
      const dataF = "\ufeff" + data;
      const hora =
        new Date().getHours() +
        ":" +
        new Date().getMinutes() +
        ":" +
        new Date().getSeconds();
      const blob = new Blob([dataF], {
        type: ' type: "text/csv;charset=UTF-8"'
      });
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.setAttribute("hidden", "");
      a.setAttribute("href", url);
      a.setAttribute("download", "consumo_minutos_" + hora + ".csv");
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    },
    async getReport() {
      // console.log('this.agents', this.agents);

      const data = this.fechas.map(row => ({
        "Año y mes": row.yearMonthString,
        Dia: row.dayString,
        "Total audios": row.totalFiles,
        "Total minutos": row.totalMinutes,
        "Audios procesados": row.processedFiles,
        "Minutos procesados": row.processedMinutes,
        "Audios no procesados": row.unprocessedFiles,
        "Minutos no procesados": row.unprocessedMinutes,
        Notas: row.notes
      }));
      console.log("data getReport", data);
      //const csvData =
      const csvData = this.objectToCsv(data);
      this.download(csvData);
    },

    mostrarTotalMinutes(data) {
      let suma2 = 0;
      data.forEach(element => {
        //totales.push(element.totalMinutes);
        suma2 += element.totalMinutes;
      });
      this.totalMinutes = Number.parseFloat(suma2).toFixed(2);
    },

    mostrarTotalCallRecords(data) {
      let suma = 0;
      data.forEach(totalCallRecords => {
        //totales.push(element.totalMinutes);
        suma += totalCallRecords.totalFiles;
      });
      this.totalCallRecords = suma;
    },

    mostrarCallRead(data) {
      let suma = 0;
      data.forEach(CallRead => {
        //totales.push(element.totalMinutes);
        suma += CallRead.processedFiles;
      });
      this.callRead = suma;
    },

    mostrargraficaMinutes(data) {
      let grafica = [];

      data.forEach(graficaMinutes => {
        let graficar = [];
        graficar.dia = graficar.push(graficaMinutes.dayString);
        graficar.minutes = graficar.push(graficaMinutes.totalMinutes);
        grafica.push(graficar);
      });

      console.log("datos a graficar", grafica);
      console.log(data);

      this.graficaMinutes = grafica;
    },
    mostrargraficaCallRecords(data) {
      let grafica = [];

      data.forEach(graficaCallRecords => {
        let graficar = [];
        graficar.dia = graficar.push(graficaCallRecords.dayString);
        graficar.minutes = graficar.push(graficaCallRecords.totalFiles);
        grafica.push(graficar);
      });

      console.log("datos a graficar", grafica);
      console.log(data);

      this.graficaCallRecords = grafica;
    }
  }
};
</script>
