<template>
  <div class="app">
    <v-app>
      <v-main>
        <h2 class="ml-3" style="color:#FF9B00">
          ESTADISTICAS POR <span style="color:#CACACA">CLUSTERS</span>
        </h2>
        <!--calendar and numCall-->
        <v-container class="">
          <v-row>
            <v-col cols="6" sm="4">
              <v-menu
                ref="menu1"
                v-model="menu1"
                :close-on-content-click="false"
                transition="scale-transition"
                offset-y
                max-width="290px"
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="dates"
                    label="Date"
                    hint="YYYY/MM/DD"
                    persistent-hint
                    prepend-icon="mdi-calendar"
                    v-bind="attrs"
                    @blur="date = parseDate(dateFormatted)"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="dates"
                  range
                  color="orange accent-3 lighten-1"
                ></v-date-picker>
              </v-menu>
              <p v-show="false">{{ dateRangeText }}</p>
            </v-col>
          </v-row>
        </v-container>

        <!--Fin calendar-->
        <!--Inicio table modulos clusters positivo y negativos-->
        <input
          class=""
          placeholder="Buscar..."
          type="text"
          v-model="search"
          v-if="pxinfo == false && modulosAndClusters == false"
        />
        <div v-if="pxinfo == false && modulosAndClusters != false">
          <v-btn
            class="ma-2"
            color="orange darken-2"
            dark
            rounded
            @click="retroceder()"
          >
            <v-icon dark left> mdi-arrow-left </v-icon>Volver
          </v-btn>
        </div>

        <v-simple-table class="mt-5" v-if="pxinfo == false">
          <template v-slot:default>
            <thead>
              <tr style="background-color:#CACACA">
                <th class="white--text ">
                  MODULOS
                </th>
                <th class="white--text ">
                  CLUSTER
                </th>
                <th class="white--text " @click="changeSortOrder(1)">
                  <img src="../../assets/sort.png" class="mr-2" />
                  ENCONTRADAS
                </th>
                <th class="white--text " @click="changeSortOrder(2)">
                  <img src="../../assets/sort.png" class="mr-2" />
                  NO ENCONTRADAS
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="modulo in filteredModules" :key="modulo.id">
                <td>
                  {{ modulo.modulo }}
                </td>
                <td>
                  {{ modulo.cluster }}
                </td>
                <td
                  @click="mostrarClusterSelected(modulo.modulo, modulo.cluster)"
                >
                  {{ modulo.positive.toFixed(1) }}%
                </td>
                <td
                  @click="
                    mostrarClusterSelectedNegative(
                      modulo.modulo,
                      modulo.cluster
                    )
                  "
                >
                  {{ modulo.negative.toFixed(1) }}%
                </td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
        <!--Fin table modulos clusters positivo y negativos-->

        <!--Tabl3 clusters-->

        <!-- Fin table clusters-->

        <!-- boton retrocede-->
        <div v-if="pxinfo == false && clickCluster != false">
          <v-btn
            class="ma-2"
            color="orange darken-2"
            dark
            fab
            @click="retroceder2()"
          >
            <v-icon dark> mdi-arrow-left </v-icon>
          </v-btn>
        </div>
        <!-- Tabla Positivos-->
        <v-simple-table
          class="mt-5"
          v-if="pxinfo == false && clickCluster == true"
        >
          <template v-slot:default>
            <thead>
              <tr style="background-color:#CACACA">
                <th class="white--text ">
                  KEYWORDS
                </th>
                <th class="white--text " @click="changeSortOrderPositive(3)">
                  <img src="../../assets/sort.png" class="mr-2" />
                  PORCENTAJE
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="keyword in filteredPositive" :key="keyword.keyword">
                <td>
                  {{ keyword.keyword }}
                </td>
                <td>{{ keyword.porcentaje.toFixed(2) }}%</td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
        <!-- fin Tabla positivos -->

        <!--tabla negativos-->
        <div v-if="pxinfo == false && clickClusterNegative != false">
          <v-btn
            class="ma-2"
            color="orange darken-2"
            dark
            fab
            @click="retroceder2()"
          >
            <v-icon dark> mdi-arrow-left </v-icon>
          </v-btn>
        </div>
        <v-simple-table
          class="mt-5"
          v-if="pxinfo == false && clickClusterNegative == true"
        >
          <template v-slot:default>
            <thead>
              <tr style="background-color:#CACACA">
                <th style="text-align: center;" class="white--text ">
                  AGENTE
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="agent in filteredAgents" :key="agent.agent">
                <td @click="mostrarAgentSelectedNegative(agent.agent)">
                  {{ agent.agent }}
                </td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
        <!--Fin tabla negativos-->

        <!--table negativos grabaciones-->
        <div v-if="pxinfo == false && clickAgent != false">
          <v-btn
            class="ma-2"
            color="orange darken-2"
            dark
            fab
            @click="retroceder3()"
          >
            <v-icon dark> mdi-arrow-left </v-icon>
          </v-btn>
        </div>
        <v-simple-table
          class="mt-5"
          v-if="pxinfo == false && clickAgent == true"
        >
          <template v-slot:default>
            <thead>
              <tr style="background-color:#CACACA">
                <th style="text-align: center;" class="white--text ">
                  GRABACIONES
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="agent in filteredGrabaciones" :key="agent.records">
                <td @click="selectedGrabacion(agent.records)">
                  {{ agent.records }}
                </td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
        <!--Fin table negativos grabaciones-->

        <!--Tablas cluster-->
        <div v-if="pxinfo == false && clickGrabacion != false">
          <v-btn
            class="ma-2"
            color="orange darken-2"
            dark
            fab
            @click="retroceder4()"
          >
            <v-icon dark> mdi-arrow-left </v-icon>
          </v-btn>
        </div>
        <v-simple-table
          class="mt-5"
          v-if="pxinfo == false && clickGrabacion == true"
        >
          <template>
            <thead>
              <tr style="background-color:#CACACA">
                <th class="white--text ">
                  MODULO
                </th>
                <th class="white--text ">
                  <span class="underline cursor-pointer">CLUSTER </span>
                </th>
                <th class="white--text ">
                  <span class="underline cursor-pointer">PUNTAJE </span>
                </th>
                <th class="white--text ">
                  <span class="underline cursor-pointer">KEYWORDS</span>
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="element in scoringKeywordsContents" :key="element.id">
                <td>
                  {{ element.module }}
                </td>
                <td style="">
                  {{ element.cluster }}
                </td>
                <td style="text-align: center;">
                  {{ element.score.toFixed(1) }} %
                </td>
                <td style="">
                  {{ element.results }}
                </td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
        <!--Tablas cluster-->

        <div v-if="pxinfo == true">
          <px-info></px-info>
        </div>
      </v-main>
    </v-app>
  </div>
</template>
<script>
import PxInfo from "@/components/agentsAudit/PxInfo.vue";
let currentUrl = window.location.pathname;
let nameBDconn = currentUrl.split("/");
let url = `${process.env.VUE_APP_URLBACKEND}${currentUrl}`;
let urlClusterScore = `${process.env.VUE_APP_URLBACKEND}/${nameBDconn[1]}/scoringkeywords`;
export default {
  name: "PxStatisticsofclusters",
  components: {
    PxInfo
  },
  data() {
    return {
      dates: [new Date().toISOString().substr(0, 10)],
      cantidadLlamadas: 0,
      menu1: false,
      statisticsofclusters: [],
      modulosAndClusters: false,
      modulos: [],
      clusters: [],
      keywordsPositive: [],
      agentsNegative: [],
      grabacionesNegative: [],
      scoringKeywordsContents: [],
      clusterScore: {},
      agentSelectedNegative: "",
      search: "",
      search2: "",
      search3: "",
      search4: "",
      clickCluster: false,
      clickAgent: false,
      clickGrabacion: false,
      moduleSelected: "",
      clusterSelected: "",
      pxinfo: true,
      bandera: false,
      bandera2: false,
      bandera3: false,
      selectGrabacion: "",
      ordenamiento1: 0,
      sortOrder: 1
    };
  },
  created() {
    this.mostrar();
  },
  computed: {
    dateRangeText() {
      return this.dates.join(" ~ ") && this.mostrar();
    },
    filteredModules: function() {
      const altOrder = this.sortOrder == 1 ? -1 : 1;
      const filtrarPor1 = this.ordenamiento1;
      return this.modulos
        .filter(modulo => {
          return (
            //modulo.modulo.toLowerCase().includes(this.search.toLowerCase()) ||
            modulo.cluster.toLowerCase().includes(this.search.toLowerCase())
          );
        })
        .sort((a, b) => {
          if (filtrarPor1 == 1) {
            if (parseFloat(a.positive) > parseFloat(b.positive)) {
              return this.sortOrder;
            }
          }
          if (filtrarPor1 == 2) {
            if (parseFloat(a.negative) > parseFloat(b.negative)) {
              return this.sortOrder;
            }
          }
          this.ordenamiento1 = 0;

          return altOrder;
        });
    },
    filteredPositive: function() {
      const altOrder = this.sortOrder == 1 ? -1 : 1;
      const filtrarPor1 = this.ordenamiento1;
      return this.keywordsPositive
        .filter(keyword => {
          return keyword.keyword
            .toLowerCase()
            .includes(this.search4.toLowerCase());
        })
        .sort((a, b) => {
          if (filtrarPor1 == 3) {
            if (parseFloat(a.porcentaje) > parseFloat(b.porcentaje)) {
              return this.sortOrder;
            }
          }
          this.ordenamiento1 = 0;

          return altOrder;
        });
    },
    filteredAgents: function() {
      return this.agentsNegative.filter(agent => {
        return agent.agent.toLowerCase().includes(this.search2.toLowerCase());
      });
    },
    filteredGrabaciones: function() {
      return this.grabacionesNegative.filter(grabacion => {
        return grabacion.records
          .toLowerCase()
          .includes(this.search3.toLowerCase());
      });
    }
  },
  methods: {
    mostrarClusterSelected(modulo, cluster) {
      if (this.bandera == false) {
        console.log("3btre false");
        this.search = cluster;
        this.clusterMostrar = true;
        this.clickCluster = true;
        this.bandera = true;
        this.clusterSelected = cluster;
        this.moduleSelected = modulo;
        this.modulosAndClusters = true;
        //this.clusterSelectedNegative="";
      } else {
        this.search = "";
        console.log("entre true");
        this.clusterSelected = "";
        this.moduleSelected = "";
        this.clusterMostrar = false;
        this.clickCluster = false;
        this.bandera = false;
        this.keywordsPositive = [];
        this.modulosAndClusters = false;
        this.clickClusterNegative = false;
        this.agentsNegative = [];
        //this.clusterSelectedNegative="";
      }

      this.mostrar();
    },
    mostrarClusterSelectedNegative(modulo, cluster) {
      if (this.bandera == false) {
        this.search = cluster;
        this.clusterMostrar = true;
        this.clickClusterNegative = true;
        this.moduleSelected = modulo;
        this.clusterSelectedNegative = cluster;
        this.bandera = true;
        this.modulosAndClusters = true;
        //this.clusterSelected="";
      } else {
        this.moduleSelected = "";
        this.clusterSelectedNegative = "";
        this.search = "";
        this.clusterMostrar = false;
        this.clickClusterNegative = false;
        this.bandera = false;
        this.agentsNegative = [];
        this.clickCluster = false;
        this.keywordsPositive = [];
        this.modulosAndClusters = false;
        //this.clusterSelected="";
      }

      this.mostrar();
    },
    mostrarAgentSelectedNegative(name) {
      if (this.bandera2 == false) {
        this.search2 = name;
        this.bandera2 = true;
        this.clickAgent = true;
        this.agentSelectedNegative = name;
        //this.clusterSelected="";
      } else {
        this.search2 = "";

        this.clickAgent = false;
        this.bandera2 = false;
        this.grabacionesNegative = [];
        //this.clusterSelected="";
      }

      this.mostrar();
    },
    selectedGrabacion(name) {
      if (this.bandera3 == false) {
        this.selectGrabacion = name;
        this.clickGrabacion = true;
        this.search3 = name;
        this.bandera3 = true;
      } else {
        this.selectGrabacion = "";
        this.clickGrabacion = false;
        this.search3 = "";
        this.scoringKeywordsContents = [];
        this.bandera3 = false;
      }

      this.mostrar();
    },
    changeSortOrder(order1) {
      this.sortOrder = this.sortOrder == 1 ? -1 : 1;
      this.ordenamiento1 = order1;

      console.log("como se va a organizar", this.ordenamiento1);
    },
    changeSortOrderPositive(order1) {
      this.sortOrder = this.sortOrder == 1 ? -1 : 1;
      this.ordenamiento1 = order1;

      console.log("como se va a organizar", this.ordenamiento1);
    },

    retroceder() {
      this.search = "";
      this.bandera = false;
      this.search2 = "";
      this.modulosAndClusters = false;
      this.clickCluster = false;
      this.keywordsPositive = [];
      this.clickClusterNegative = false;
      this.agentsNegative = [];
      this.clickAgent = false;
      this.search3 = "";
      this.scoringKeywordsContents = [];
      this.clickGrabacion = false;
      this.bandera3 = false;
    },

    retroceder2() {
      this.search = "";
      this.bandera = false;
      this.search2 = "";
      this.modulosAndClusters = false;
      this.clickCluster = false;
      this.keywordsPositive = [];
      this.clickClusterNegative = false;
      this.agentsNegative = [];
      this.clickAgent = false;
      this.search3 = "";
      this.scoringKeywordsContents = [];
      this.clickGrabacion = false;
      this.bandera3 = false;
    },
    retroceder3() {
      this.search2 = "";
      this.grabacionesNegative = [];
      this.clickAgent = false;
      this.bandera2 = false;
      this.search3 = "";
      this.scoringKeywordsContents = [];
      this.clickGrabacion = false;
      this.bandera3 = false;
    },
    retroceder4() {
      this.search3 = "";
      this.scoringKeywordsContents = [];
      this.clickGrabacion = false;
      this.bandera3 = false;
    },

    async mostrar() {
      let response;
      if (this.dates.length == 1) {
        response = await this.axios.get(
          url + `?eventDate=${this.dates}T00:00:00.000Z`
        );
      } else {
        response = await this.axios.get(
          url +
            `?eventDate=${this.dates[0]}T00:00:00.000Z&&eventDate=${this.dates[1]}T00:00:00.000Z`
        );
      }

      this.statisticsofclusters = response.data.body;
      //console.log('statisticsofclusters',this.statisticsofclusters);

      this.mostrarTableModulesAndClusters(this.statisticsofclusters);
      // this.mostrarTableClusters(this.statisticsofclusters,this.moduleSelected);
      if (this.clusterSelected != "") {
        this.mostrarTablePositive(
          this.statisticsofclusters,
          this.moduleSelected,
          this.clusterSelected
        );
      }
      if (this.clusterSelectedNegative != "") {
        this.mostrarTableNegative(
          this.statisticsofclusters,
          this.moduleSelected,
          this.clusterSelectedNegative
        );
        this.mostrarTableGrabaciones(
          this.statisticsofclusters,
          this.moduleSelected,
          this.clusterSelectedNegative,
          this.agentSelectedNegative
        );
      }
      if (this.selectGrabacion != "") {
        const responseClusterScore = await this.axios.get(
          urlClusterScore + `?keyfile=${this.selectGrabacion}`
        );
        console.log(
          "url Phrases",
          urlClusterScore + `?keyfile=${this.selectGrabacion}`
        );
        console.log("object", responseClusterScore.data.body);
        this.clusterScore = responseClusterScore.data.body;
        this.mostrarTableCluster(this.clusterScore, this.selectGrabacion);
      }
    },
    mostrarTableModulesAndClusters(data) {
      //let totalRecordings = 0;
      if (data.length == 0 || data.length < 0) {
        this.pxinfo = true;
      } else {
        this.pxinfo = false;
      }
      //let superModulo = {};
      //  for(let i=0;i<data.length;i++){
      //      for(let modulo in data[i].contents){

      //          if(!Object.prototype.hasOwnProperty.call(superModulo,modulo)){
      //              superModulo[modulo] = data[i].contents[modulo];
      //          }else{
      //              superModulo[modulo] += data[i].contents[modulo];
      //          }
      //      }
      //  }
      let superModuloandClusterArray = [];
      let superModuloandClusterDicc = {};

      for (let i = 0; i < data.length; i++) {
        console.log("00000000000000-data.length", i);
        for (let modulo in data[i].contents) {
          console.log("1111111111-modulo", modulo);
          for (let cluster in data[i].contents[modulo]) {
            console.log("222222222-cluster", cluster);

            let key = JSON.stringify([modulo, cluster]);
            if (!(key in superModuloandClusterDicc)) {
              superModuloandClusterDicc[key] = {
                positive: data[i].contents[modulo][cluster].positive.percentage,
                negative: data[i].contents[modulo][cluster].negative.percentage
              };
            } else {
              superModuloandClusterDicc[key]["positive"] +=
                data[i].contents[modulo][cluster].positive.percentage;
              superModuloandClusterDicc[key]["negative"] +=
                data[i].contents[modulo][cluster].negative.percentage;
            }

            // let moduloPackage = {
            //   id: id++,
            //   modulo: modulo,
            //   cluster: cluster,
            //   positive:
            //     data[i].contents[modulo][cluster].positive.percentage * 100,
            //   negative:
            //     data[i].contents[modulo][cluster].negative.percentage * 100
            // };

            //superModuloandClusterArray.push(moduloPackage);
          }
        }
      }

      //console.log("supermoduloDicc", superModuloandClusterDicc);
      let id = 0;
      for (let key in superModuloandClusterDicc) {
        let moduleCluster = JSON.parse(key);

        let moduloPackage = {
          id: id++,
          modulo: moduleCluster[0],
          cluster: moduleCluster[1],
          positive:
            (superModuloandClusterDicc[key]["positive"] / data.length) * 100,

          negative:
            (superModuloandClusterDicc[key]["negative"] / data.length) * 100
        };

        superModuloandClusterArray.push(moduloPackage);
        //console.log("moduloPackage", moduloPackage);
      }

      this.modulos = superModuloandClusterArray;
    },
    mostrarTableClusters(data, selectedCluster) {
      //console.log(data,selectedCluster);
      //let superClusters = {};
      let superClustersArray = [];
      for (let i = 0; i < data.length; i++) {
        //for(let cluster in data[i].contents[selectedCluster]){

        //console.log('identificacion',identificacion);
        //  if(!(cluster in superClusters)){

        //      superClusters[cluster] = data[i].contents[selectedCluster][cluster].length;
        //      superClusters[cluster] = data[i].contents[selectedCluster][cluster].negative.percentage;
        //      superClusters[cluster] = data[i].contents[selectedCluster][cluster].positive.percentage;

        //  }else{
        //      superClusters[cluster] += data[i].contents[selectedCluster][cluster].length;
        //      superClusters[cluster] = data[i].contents[selectedCluster][cluster].negative.percentage;
        //      superClusters[cluster] = data[i].contents[selectedCluster][cluster].positive.percentage;
        //  }

        //}

        for (let cluster in data[i].contents[selectedCluster]) {
          let clusterPackage = {
            cluster: cluster,
            positive:
              data[i].contents[selectedCluster][cluster].positive.percentage,
            negative:
              data[i].contents[selectedCluster][cluster].negative.percentage
          };
          superClustersArray.push(clusterPackage);
        }
      }
      //console.log("super CLU",superClusters);

      this.clusters = superClustersArray;
      //console.log("superC",superClustersArray)
    },
    mostrarTablePositive(data, selectedModulo, selectedCluster) {
      //let superKeywords= {};
      let superKeywordsArray = [];
      let superKeywordsDicc = {};
      for (let i = 0; i < data.length; i++) {
        for (let keyword in data[i].contents[selectedModulo][selectedCluster]
          .positive.keywordPercentage) {
          if (!(keyword in superKeywordsDicc)) {
            superKeywordsDicc[keyword] =
              data[i].contents[selectedModulo][
                selectedCluster
              ].positive.keywordPercentage[keyword];
          } else {
            superKeywordsDicc[keyword] +=
              data[i].contents[selectedModulo][
                selectedCluster
              ].positive.keywordPercentage[keyword];
          }
        }
      }

      for (let key in superKeywordsDicc) {
        let keywordPackage = {
          keyword: key,
          porcentaje: (superKeywordsDicc[key] / data.length) * 100
        };

        superKeywordsArray.push(keywordPackage);
      }
      console.log("superKeywordsDicc", superKeywordsDicc);

      console.log("superKeywordsArray", superKeywordsArray);
      this.keywordsPositive = superKeywordsArray;
    },
    mostrarTableNegative(data, selectedModulo, selectedCluster) {
      let superAgents = {};
      for (let i = 0; i < data.length; i++) {
        for (let agent in data[i].contents[selectedModulo][selectedCluster]
          .negative.agentRecord) {
          console.log("agent", agent);
          if (!(agent in superAgents)) {
            superAgents[agent] =
              data[i].contents[selectedModulo][
                selectedCluster
              ].negative.agentRecord.length;
          } else {
            superAgents[agent] +=
              data[i].contents[selectedModulo][
                selectedCluster
              ].negative.agentRecord.length;
          }
        }
      }

      let superAgentsArray = [];

      for (let agent in superAgents) {
        let agentPackage = {
          agent: agent
        };
        superAgentsArray.push(agentPackage);
      }

      this.agentsNegative = superAgentsArray;
    },
    mostrarTableGrabaciones(
      data,
      selectedModulo,
      selectedCluster,
      selectedAgent
    ) {
      let superGrabaciones = {};

      for (let i = 0; i < data.length; i++) {
        let records =
          data[i].contents[selectedModulo][selectedCluster].negative
            .agentRecord[selectedAgent];
        for (let j = 0; j < records.length; j++) {
          console.log("records", records[j]);
          superGrabaciones[j] = records[j];
        }
      }
      console.log("superGrabaciones", superGrabaciones);

      let superGrabacionesArray = [];

      for (let records in superGrabaciones) {
        let grabacionPackage = {
          records: superGrabaciones[records]
        };
        superGrabacionesArray.push(grabacionPackage);
      }
      console.log("superGrabacionesArray", superGrabacionesArray);
      this.grabacionesNegative = superGrabacionesArray;
    },

    mostrarTableCluster(data, keyfile) {
      //this.scoringKeywordsContents=data[0].contents;
      console.log("keyfile", keyfile);
      let contents = data[0].contents;
      let scoringArray = [];
      let id = 0;
      for (let moduleKey in contents) {
        //console.log('modulekey',moduleKey);

        //let clusterArray = [];
        for (let clusterKey in contents[moduleKey]) {
          //console.log("cluster[key]",clusterKey);

          let kp = contents[moduleKey][clusterKey].results;
          let kpStrings = kp.join(", ");
          let clusterPackage = {
            id: id,
            module: moduleKey,
            cluster: clusterKey,
            score: contents[moduleKey][clusterKey].score * 100,
            results: kpStrings
          };
          //clusterArray.push(clusterPackage);
          scoringArray.push(clusterPackage);
          id++;
        }
        //  let modulePackage = {

        //    module:moduleKey,
        //    clusters:clusterArray
        //  }
        //scoringArray.push(modulePackage);
        console.log(scoringArray);
        this.scoringKeywordsContents = scoringArray;
      }
    }
  }
};
</script>
