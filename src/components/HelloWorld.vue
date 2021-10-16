<template>

  <div class="container-fluid">

    <ul class="nav nav-tabs">
      <li class="nav-item" v-for="(Tab, tidx) in Tabs" :key="tidx">
        <a class="nav-link" :class="{'active' : Tab.Visible }" @click="SelectTab(Tab.Label)">{{ Tab.Label }}</a>
      </li>
    </ul>

    <div class="row" v-show="Tabs[0].Visible">
      
      <div class="col-2">

        <select class="form-select" v-model="CurrentTestKey" @change="Redraw">
          <option selected value="-1">Elegir prueba</option>
          <option
            v-for="(test, testIdx) in Tests"
            :value="test.Key"
            :key="testIdx">
            {{test.Label}}
          </option>
        </select>

        <select class="form-select" v-model="ModelIdx" @change="Redraw">
          <option selected value="-1">Elegir modelo</option>
          <option
            v-for="(model, modelIdx) in Models"
            :value="modelIdx"
            :key="modelIdx">
            {{model}}
          </option>
        </select>

        <select class="form-select" v-model="ColumnKey" @change="Redraw">
          <option selected value="">Elegir columna</option>
          <option
            v-for="(col, colIdx) in VisibleColumns"
            :value="col"
            :key="colIdx">
            {{col}}
          </option>
        </select>

        <select class="form-select" v-model="ChartType" @change="Redraw">
          <option selected value="">Tipo Chart</option>
          <option
            v-for="(ctype, ctypeidx) in ChartTypes"
            :value="ctype"
            :key="ctypeidx">
            {{ctype}}
          </option>
        </select>
        
        <template v-if="Ok">
          <h3>Valores M:</h3>
          <ul><li v-for="(item, idx) in KeyedDataM" :key="idx">{{item.Key}} : {{item.Value}}</li></ul>

          <h3>Valores F:</h3>
          <ul><li v-for="(item, idx) in KeyedDataF" :key="idx">{{item.Key}} : {{item.Value}}</li></ul>

          <h3>Diferencia M-F:</h3>
          <ul><li v-for="(item, idx) in KeyedDiff" :key="idx"><span :style="{'color':KeyedDiffColors.find(x => x.Key === item.Key).Value}">{{item.Key}}</span> {{item.Value}}</li></ul>

        </template>

        <div class="form-check" v-if="Ok">
        <input class="form-check-input" type="checkbox" v-model="RemoveQuestionMarks" @change="Redraw">
        <label class="form-check-label" for="flexCheckDefault">
          Ocultar [???]
        </label>
        </div>




      </div>

      <div class="col-10">
      <h1 v-if="Ok">{{ CurrentModelName }}
        <small v-if="Dumps && Dumps[CurrentTestKey]?.Link">
          <a :href="Dumps[CurrentTestKey].Link" target="_blank"> Paper</a>
        </small>
        <small v-if="Ok">
          <a :href="CurrentModelLink" target="_blank"> Model </a>
        </small>
      </h1>
      <h1 v-else>Sin configurar</h1>
      <div class="caja">
        <Radial
          :labels="Labels"
          :datam="DataM"
          :dataf="DataF"
          :type="ChartType"
          v-if="Ok && !Loading"
          ></Radial>
      </div>

      </div>

    </div>

    <div class="row" v-show="Tabs[1].Visible">
      
      <div class="col-md-2">

        <select class="form-select" v-model="CurrentTestKey" @change="Redraw">
          <option selected value="-1">Elegir prueba</option>
          <option
            v-for="(test, testIdx) in Tests"
            :value="test.Key"
            :key="testIdx">
            {{test.Label}}
          </option>
        </select>

        <select class="form-select" v-model="CurrentLabel" @change="Redraw">
          <option selected value="">Elegir label</option>
          <option
            v-for="(label, labelIdx) in Labels"
            :value="label"
            :key="labelIdx">
            {{label}}
          </option>
        </select>
        <select class="form-select" v-model="TableShow" @change="Redraw">
          <option selected value="">Elegir label</option>
          <option
            v-for="(ts, tsIdx) in TableShowTypes"
            :value="ts"
            :key="tsIdx">
            {{ts}}
          </option>
        </select>

        <div v-if="TableShow==='Experimental'">
          <label> Corrección </label><input type="number" step="0.1" v-model="ExperimentalCorrection"> {{ ExperimentalCorrection }}
        </div>
      </div>

      <div class="col-8">

        <table v-if="CurrentLabel" class="table">
          <tr>
            <th></th>
            <th v-for="(col, colIdx) in VisibleColumns" :key="colIdx">
              {{ col }}
            </th>
          </tr>
          <tr v-for="(model, mIdx) in Models" :key="mIdx">
            <th style="text-align: right;">{{model}} {{mIdx+1}}</th>
            <template v-for="(col, colIdx) in VisibleColumns" :key="colIdx">
              <td v-if="TableShow === 'M'">{{ DataTable[0][mIdx][col] }}</td>
              <td v-if="TableShow === 'F'">{{ DataTable[1][mIdx][col] }}</td>
              <td v-if="TableShow === 'M-F'" :style="{'background-color': DataTable[0][mIdx][col] - DataTable[1][mIdx][col] > 0 ? 'aliceblue' : 'lightpink'}">
                {{ DataTable[0][mIdx][col] - DataTable[1][mIdx][col] }}
              </td>
              <td v-if="TableShow === 'M/F'" :style="{'background-color': DataTable[0][mIdx][col] - DataTable[1][mIdx][col] > 0 ? 'aliceblue' : 'lightpink'}">
                {{ DataTable[0][mIdx][col] / DataTable[1][mIdx][col] }}
              </td>
              <td v-if="TableShow === 'Experimental'" :style="{'background-color': DataTable[0][mIdx][col] - DataTable[1][mIdx][col] + ExperimentalCorrection > 0 ? 'aliceblue' : 'lightpink'}">
                  {{ DataTable[0][mIdx][col] - DataTable[1][mIdx][col] }}
              </td>
              <td v-if="TableShow === 'Summary'">
                <span v-if="DataTable[0][mIdx][col] - DataTable[1][mIdx][col] > 0">Masculino</span>
                <span v-else>Femenino</span>
              </td>
            </template>
          </tr>

        </table>

      </div>

    </div>

    <div class="row" v-show="Tabs[2].Visible">

      <div class="col-md-12">
        <span v-for="(col, cidx) in Columns" :key="cidx" @click="ToggleColumn(col)" :style="{'background-color' : ColumnMeta[col]?.Hide ? 'lightgrey' : 'lightblue'}" style="margin-right: 10px">
          {{ col }}
        </span>
      </div>

    </div>

    <div class="row" v-show="Tabs[3].Visible">

      <div class="col" v-if="Ok">
        {{ DataM }}
      </div>

      <div class="coenable-prc" v-if="Ok">
        {{ DataF }}
      </div>

      <div class="col">
        <pre>
          ModelIdx: {{ $data.ModelIdx }}
          ColumnKey: {{ $data.ColumnKey }}  
          OK: {{ Ok}}
        </pre>
      </div>

        <div class="col">
          Models
          <pre>{{ Models }}</pre>
        </div>

        <div class="col">
        Columns
          <pre>{{ Columns }}</pre>
        </div>

        <div class="col">
          VisibleColumns
          {{ VisibleColumns }}
        </div>

        <div class="col">
          Labels
          <pre>{{ Labels }}</pre>
        </div>

        <div class="col">
          <pre>
            {{ DataTable }}
          </pre>
        </div>

    </div>

    <div class="row" v-show="Tabs[4].Visible">
      <h1> Latex </h1>
      <textarea v-model="LatexContent"></textarea>
    </div>

  <div class="row" v-show="Tabs[5].Visible">
    <div class="col-3">
      <qty-line-chart :EnableWordCount="true" named="wc"></qty-line-chart>
    </div>
    <div class="col-4">
      <qty-line-chart :EnableTotalDiff="true" named="td"></qty-line-chart>
    </div>
    <div class="col-4">
      <qty-line-chart :EnablePrc="true" named="prc"></qty-line-chart>
    </div>
  </div>

  <div class="row" v-show="Tabs[6].Visible">
      <tabla-adjetivos></tabla-adjetivos>
  </div>



</div>



</template>

<script>
/* eslint-disable no-debugger */
/* eslint-disable vue/no-unused-components */

import TestOriginal from './../../../StereoES/result_fillmask/categorias_polaridad_visibilidad/run_result.json'
import TestOriginalNegado from './../../../StereoES/result_fillmask/categorias_polaridad_visibilidad_negadas/run_result.json'
import TestFoaFoa from './../../../StereoES/result_fillmask/categorias_polaridad_foa_foa/run_result.json'
import TestYulia from './../../../StereoES/result_fillmask/categorias_yulia/run_result.json'
import TestProfesiones from './../../../StereoES/result_fillmask_profesiones/base/run_result.json'
import Radial from './Radar.vue';
import QtyLineChart from './Chart/QtyLineChart.vue';
import TablaAdjetivos from './TablaAdjetivos.vue'

export default {
  name: 'HelloWorld',
  components: { Radial, QtyLineChart, TablaAdjetivos },
  created(){
  },
  props: {
    msg: String
  },
  data(){
    return {
      // App state
      Loading: false,
      ModelIdx: -1,
      ColumnKey: '',
      ChartType: 'radar',
      CurrentTestKey: 'Original',
      RemoveQuestionMarks: true,
      CurrentLabel: '',

      // Tab - Config
      ColumnMeta: {},

      // Tab - Table
      TableShow: 'M',
      TableShowTypes: ['M', 'F', 'M-F','M/F', 'Experimental', 'Summary'],
      ExperimentalCorrection: 0,


      // Vista
      Tabs: [
        {
          Label: "Charts",
          Visible: true,
        },
        {
          Label: "Tables",
          Visible: false,
        },
        {
          Label: "Config",
          Visible: false,
        },
        {
          Label: "Debug",
          Visible: false,
        },
        {
          Label: "Latex",
          Visible: false
        },
        {
          Label: "Size",
          Visible: false,
        },
        {
          Label: "Adjective Stats",
          Visible: false,
        }
      ],

      // Tab - Latex
      LatexContent: "",

      // Data
      Dumps: {
        Original: { 
          Key: 'Original',
          Label: 'Original VISIBLE/INVISIBLE POSITIVO/NEGATIVO',
          Data: TestOriginal
        },
        OriginalNegado: { 
          Label: 'Original Negado',
          Key: 'OriginalNegado',
          Data: TestOriginalNegado
        },
        // https://sci-hub.ru/10.1037//0022-3514.37.3.395

        FoaFoa: {
          Key: 'FoaFoa',
          Label: 'Foa & Foa (Solo Invisibles/Traits)',
          Data: TestFoaFoa,
          Link: 'https://sci-hub.ru/10.1037//0022-3514.37.3.395'
        },
        Yulia: {
          Key: 'Yulia',
          Label: 'Yulia',
          Data: TestYulia,
          Link: 'http://www.cs.cmu.edu/~ytsvetko/papers/adj-lrec14.pdf'
        },
        Profesiones: {
          Key: 'Profesiones',
          Label: 'Profesiones',
          Data: TestProfesiones,
          Link: 'https://www.cnae.com.es/lista-actividades.php',
          DataLink: 'https://www.ine.es/jaxiT3/Tabla.htm?t=4128'
        }
      },

      // Static
      ChartTypes: ['radar', 'bar', 'line', 'doughnut', 'polarArea'],
    }
  }, 
  methods: {
    Redraw(){
      this.Loading = true;
      setTimeout( () => this.Loading = false, 100);
    },
    KVData(idx){
      let modelm = this.Data[this.ModelIdx]["py/tuple"][idx];
      let values = [];
      this.Labels.forEach( label => values.push({ Key: label, Value: modelm[label][this.ColumnKey]}) );
      return values;
    },
    SelectTab(label){
      this.Tabs = this.Tabs.map( (tab) => {
        tab.Visible = false;
        return tab;
      });
      this.Tabs.find( x => x.Label === label).Visible = true;
    },
    WriteLatex(text){
      this.LatexContent = text;
      this.SelectTab("Latex");
    },
    ToggleColumn(col){
      if(!this.ColumnMeta[col]) this.ColumnMeta[col] = {};
      this.ColumnMeta[col].Hide = !this.ColumnMeta[col].Hide
    },
    AsLatex(){},
//       let cols
//       let template = `
// \\begin{center}
// \\begin{tabular}{ c c c }
//  cell1 & cell2 & cell3 \\ 
//  cell4 & cell5 & cell6 \\  
//  cell7 & cell8 & cell9    
// \\end{tabular}
// \\end{center}
// `;
//     }

  },
  computed: {
    Data(){
      let key = this.CurrentTestKey;
      return this.Dumps[key].Data;
    },
    DataTable(){
      let tableM = [];
      let tableF = [];
      let data = this.Data;
      // let models = this.Models;

      // table.push(" ");
      // for( let model in model)
      //   table.push(model);

      for( let i = 0; i < data.length; i++){
        let m = this.Data[i]["py/tuple"][1];
        let f = this.Data[i]["py/tuple"][2];
        tableM.push(...Object.values(m).filter( xrow => xrow.cat === this.CurrentLabel ));
        tableF.push(...Object.values(f).filter( xrow => xrow.cat === this.CurrentLabel ));

        // tableM.push(...Object.values(m).map( row => Object.values(row).splice(1) ).filter( xrow => xrow[0] === this.CurrentLabel ));
        // tableF.push(...Object.values(f).map( row => Object.values(row).splice(1) ).filter( xrow => xrow[0] === this.CurrentLabel ));
      }

      return [tableM, tableF ];
    },
    KeyedDataM(){
      return this.KVData(1);
    },
    KeyedDataF(){
      return this.KVData(2);
    },
    KeyedDiff(){
      let items = Object.values(this.KeyedDataM);
      let arr = [];
      debugger;
      for( let item of items ) {
        arr.push( {Key: item.Key, Value: this.KeyedDataM.find(x => x.Key === item.Key).Value - this.KeyedDataF.find(x => x.Key === item.Key).Value});
      }
      return arr;
    },
    KeyedDiffColors(){
      let items = Object.values(this.KeyedDiff);
      let arr = [];

      for( let item of items ) {
        let color = item.Value > 0 ? 'blue' : 'darkmagenta';
        arr.push( {Key: item.Key, Value: color} );
      }
      return arr;
    },
    DataM(){
      return this.KVData(1).map( x => x.Value);
    },
    DataF(){
      return this.KVData(2).map( x => x.Value);
    },
    CurrentModelName(){
      return this.Data[this.ModelIdx]["py/tuple"][0];
    },
    Ok(){
      return this.ModelIdx !== -1 && this.ColumnKey !== '';
    },
    Labels(){
      let labels = Object
        .keys(this.Data[0]["py/tuple"][1])
        .filter( x => x !== "[_TOTAL]");
      if(this.RemoveQuestionMarks)
      return labels.filter( x => x !== "[???]");
      return labels;
    },
    Models(){
      return this.Data.map( modelResult => modelResult["py/tuple"][0] );
    },
    Columns(){
      return Object.keys(this.Data[0]["py/tuple"][1]["[???]"])
      .filter( x => !x.includes("py/") )
      .filter( x => x !== "cat" );
    },
    VisibleColumns(){
      return this.Columns.filter( col => this.ColumnMeta[col]?.Hide !== true);
    },
    Tests(){
      return Object.values(this.Dumps);
    },
    CurrentModelLink(){
      return 'https://huggingface.co/' + this.CurrentModelName;
    }

  }
  
}
</script>

<style scoped>
pre,h1, li {
  text-align: left;
}
.caja {
  min-height: 75vh;
  border: 1px solid black;
  width: 65%;
}
li {
  font-family: monospace;
  list-style-type: none;
  
}

h3 {
  margin-top: 15px !important
}
</style>
