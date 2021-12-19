<template>
  <div class="container-fluid">
    <ul class="nav nav-tabs">
      <li class="nav-item" v-for="(Tab, tidx) in Tabs" :key="tidx">
        <a
          class="nav-link"
          v-if="!Tab.Hide"
          :class="{ 'active': Tab.Visible }"
          @click="SelectTab(Tab.Label)"
        >{{ Tab.Label }}</a>
      </li>
    </ul>

    <div class="row" v-show="Tabs[0].Visible">
      <div class="col-2">
        <select class="form-select" v-model="CurrentTestKey" @change="Redraw">
          <option selected value="-1">Elegir prueba</option>
          <option v-for="(test, testIdx) in Tests" :value="test.Key" :key="testIdx">{{ test.Label }}</option>
        </select>

        <select class="form-select" v-model="ModelIdx" @change="Redraw">
          <option selected value="-1">Elegir modelo</option>
          <option v-for="(model, modelIdx) in Models" :value="modelIdx" :key="modelIdx">{{ model }}</option>
        </select>

        <select class="form-select" v-model="ColumnKey" @change="Redraw">
          <option selected value>Elegir columna</option>
          <option v-for="(col, colIdx) in VisibleColumns" :value="col" :key="colIdx">{{ col }}</option>
        </select>

        <select class="form-select" v-model="ChartType" @change="Redraw">
          <option selected value>Tipo Chart</option>
          <option v-for="(ctype, ctypeidx) in ChartTypes" :value="ctype" :key="ctypeidx">{{ ctype }}</option>
        </select>

        <template v-if="Ok">
          <h3>Valores M:</h3>
          <ul>
            <li v-for="(item, idx) in KeyedDataM" :key="idx">{{ item.Key }} : {{ item.Value }}</li>
          </ul>

          <h3>Valores F:</h3>
          <ul>
            <li v-for="(item, idx) in KeyedDataF" :key="idx">{{ item.Key }} : {{ item.Value }}</li>
          </ul>

          <h3>Diferencia M-F:</h3>
          <ul>
            <li v-for="(item, idx) in KeyedDiff" :key="idx">
              <span
                :style="{ 'color': KeyedDiffColors.find(x => x.Key === item.Key).Value }"
              >{{ item.Key }}</span>
              {{ item.Value }}
            </li>
          </ul>
        </template>

        <div class="form-check" v-if="Ok">
          <input
            class="form-check-input"
            type="checkbox"
            v-model="RemoveQuestionMarks"
            @change="Redraw"
          />
          <label class="form-check-label" for="flexCheckDefault">Ocultar [???]</label>
        </div>
      </div>

      <div class="col-10">
        <h1 v-if="Ok">
          {{ CurrentModelName }}
          <small v-if="Dumps && Dumps[CurrentTestKey]?.Link" class="mr5">
            <a :href="Dumps[CurrentTestKey].Link" target="_blank">Paper</a>
          </small>
          <small v-if="Ok">
            <a :href="CurrentModelLink" target="_blank">Model</a>
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

    <div class="row" v-show="Tabs[1].Visible" v-if="!RedrawingTable">

      <div class="col-12">
        <div class="row mb5">
          <div class="col-md-3">
            <select class="form-select" v-model="CurrentTestKey" @change="Redraw">
              <option selected value="-1">Elegir prueba</option>
              <option v-for="(test, testIdx) in Tests" :value="test.Key" :key="testIdx">{{ test.Label }}</option>
            </select>
          </div>

          <div class="col-md-3">
            <select class="form-select" v-model="CurrentLabel" @change="Redraw">
              <option selected value>Elegir label</option>
              <option v-for="(label, labelIdx) in Labels" :value="label" :key="labelIdx">{{ label }}</option>
            </select>
          </div>

          <div class="col-md-3">
            <select class="form-select" v-model="TableShow" @change="Redraw">
              <option selected value>Elegir label</option>
              <option v-for="(ts, tsIdx) in TableShowTypes" :value="ts" :key="tsIdx">{{ ts }}</option>
            </select>
          </div>

          <div class="col-md-3">
            <div v-if="TableShow === 'Experimental'">
              <label>Corrección</label>
              <input type="number" step="0.1" v-model="ExperimentalCorrection" />
              {{ ExperimentalCorrection }}
            </div>
          </div>
        </div>
      </div>

      <div class="col-8">
        <table v-if="CurrentLabel" class="table b1">
          <tbody>
            <tr>
              <th></th>
              <th
                v-for="(col, colIdx) in VisibleColumns"
                :key="colIdx"
                class="bb1"
              >{{ ColumnRename(col) }}</th>
            </tr>
            <tr v-for="(model, mIdx) in VisibleModels" :key="mIdx">
              <th style="text-align: right;" class="br1">
                {{ model }}
                <span v-show="MostrarX" @click="ToggleModel(model)">❌</span>
              </th>
              <template v-for="(col, colIdx) in VisibleColumns" :key="colIdx">
                <td v-if="TableShow === 'M'">{{ GetValueMale(model, col) }}</td>
                <td v-if="TableShow === 'F'">{{ GetValueFemale(model, col) }}</td>
                <td
                  v-if="TableShow === 'M-F'"
                  :style="{ 'background-color': GetValueMale(model, col) - GetValueFemale(model, col) > 0 ? 'aliceblue' : 'lightpink' }"
                >{{ Cut(GetValueMale(model, col) - GetValueFemale(model, col))  }}</td>
                <td
                  v-if="TableShow === 'M-F Heat'"
                  :style="StyleHeatmap(col, GetValueMale(model, col) - GetValueFemale(model, col))"
                  class="br1"
                >{{ Cut( GetValueMale(model, col) - GetValueFemale(model, col) ) }}</td>
                <td
                  v-if="TableShow === 'M/F'"
                  :style="{ 'background-color': GetValueMale(model, col) - GetValueFemale(model, col) > 0 ? 'aliceblue' : 'lightpink' }"
                >{{ GetValueMale(model, col) / GetValueFemale(model, col) }}</td>
                <td
                  v-if="TableShow === 'Experimental'"
                  :style="{ 'background-color': GetValueMale(model, col) - GetValueFemale(model, col) + ExperimentalCorrection > 0 ? 'aliceblue' : 'lightpink' }"
                >{{ GetValueMale(model, col) - GetValueFemale(model, col) }}</td>
                <td v-if="TableShow === 'Summary'">
                  <span
                    v-if="GetValueMale(model, col) - GetValueFemale(model, col) > 0"
                  >Masculino</span>
                  <span v-else>Femenino</span>
                </td>
              </template>
            </tr>
            <tr v-if="ShowLatexCount">
              <th>Latex Count</th>
              <template v-for="(col, colIdx) in VisibleColumns" :key="colIdx">
                <td>{{ SummaryRow(col) }}</td>
              </template>
            </tr>
          </tbody>

          <tfoot class="bt3">
            <tr v-if="ShowMinMax">
              <th class="r br1">Min</th>
              <template v-for="(col, colIdx) in VisibleColumns" :key="colIdx">
                <td>{{ Cut(TableMin[col]) }}</td>
              </template>
            </tr>

            <tr v-if="ShowMinMax">
              <th class="r br1">Max</th>
              <template v-for="(col, colIdx) in VisibleColumns" :key="colIdx">
                <td>{{ Cut(TableMax[col]) }}</td>
              </template>
            </tr>
          </tfoot>
        </table>
      </div>

      <div class="col-md-4">
        <hr>
        <span @click="MostrarX = !MostrarX">Mostrar ❌ {{ MostrarX }}</span><br>
        <span @click="ShowLatexCount = !ShowLatexCount">ShowLatexCount {{ ShowLatexCount }}</span><br>
        <span @click="ShowMinMax = !ShowMinMax">ShowMinMax {{ ShowMinMax }}</span><br>
        <span @click="ChangeHeatmapType">UseSymmetricHeatMap {{ UseSymmetricHeatMap }}</span><br>

        <hr>
        <h3>Columns</h3>
        <span
          v-for="(col, cidx) in Columns"
          :key="cidx"
          @click="ToggleColumn(col)"
          :style="{ 'background-color': ColumnMeta[col]?.Hide ? 'lightgrey' : 'lightblue' }"
          style="margin-right: 10px"
        >{{ ColumnRename(col) }}</span>

        <hr>
        <h3>Models
          <small>
            (All
            <span @click="ToggleAllModels(false)">On</span>/
            <span @click="ToggleAllModels(true)">Off</span>)
          </small>
        </h3>
        <span
          v-for="(model, midx) in Models"
          :key="midx"
          @click="ToggleModel(model)"
          :style="{ 'background-color': ModelMeta[model]?.Hide ? 'lightgrey' : 'lightblue' }"
          style="margin-right: 10px"
        >{{ model }}</span>
      </div>

      </div>


    <div class="row" v-show="Tabs[2].Visible">
      <!-- <div class="col-md-12">
        <h3>Configurar columnas</h3>
        <span v-for="(col, cidx) in Columns" :key="cidx" @click="ToggleColumn(col)" :style="{'background-color' : ColumnMeta[col]?.Hide ? 'lightgrey' : 'lightblue'}" style="margin-right: 10px">
          {{ col }}
        </span>
      </div>

      <div class="col-md-12">
        <h3>Configurar modelos</h3>
        <span v-for="(model, midx) in Models" :key="midx" @click="ToggleModel(model)" :style="{'background-color' : ColumnMeta[model]?.Hide ? 'lightgrey' : 'lightblue'}" style="margin-right: 10px">
          {{ model }}
        </span>
      </div>-->
    </div>

    <div class="row" v-show="Tabs[3].Visible">
      <div class="col" v-if="Ok">{{ DataM }}</div>

      <div class="coenable-prc" v-if="Ok">{{ DataF }}</div>

      <div class="col">
        <pre>
          ModelIdx: {{ $data.ModelIdx }}
          ColumnKey: {{ $data.ColumnKey }}  
          OK: {{ Ok }}
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
      <h1>Latex</h1>
      <textarea v-model="LatexContent"></textarea>
    </div>

    <div class="row" v-show="Tabs[5].Visible"></div>

    <div class="row" v-show="Tabs[6].Visible">
      <tabla-adjetivos></tabla-adjetivos>

      <hr />

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

    <div class="row" v-show="Tabs[7].Visible">
      <explorador-predicciones></explorador-predicciones>
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
// import TestProfesiones from './../../../StereoES/result_fillmask_profesiones/base/run_result.json'
import Radial from './Radar.vue';
import QtyLineChart from './Chart/QtyLineChart.vue';
import TablaAdjetivos from './TablaAdjetivos.vue'
import ExploradorPredicciones from './ExploradorPredicciones.vue'

export default {
  name: 'HelloWorld',
  components: { Radial, QtyLineChart, TablaAdjetivos, ExploradorPredicciones },
  created() {
    this.Columns.forEach(col => this.ColumnMeta[col] = { Hide: true });
    this.Models.forEach((col, idx) => this.ModelMeta[col] = { Hide: false, Index: idx });
    this.ColumnMeta["prc_retrieval_status_value"].Hide = false;
    this.ColumnMeta["prc_probability"].Hide = false;
    this.CurrentLabel = this.Labels[0];
    this.ColumnKey = this.VisibleColumns[0];

  },
  props: {
    msg: String
  },
  data() {
    return {
      // App state
      Loading: false,
      ModelIdx: 0,
      ColumnKey: '',
      ChartType: 'bar',
      CurrentTestKey: 'Yulia',
      RemoveQuestionMarks: true,
      CurrentLabel: '',

      // Tab - Config
      ColumnMeta: {},
      ModelMeta: {},
      MostrarX: false,
      ShowLatexCount: false,
      ShowMinMax: true,
      UseSymmetricHeatMap: true,
      RedrawingTable: false,

      // Tab - Table
      TableShow: 'M-F Heat',
      TableShowTypes: ['M', 'F', 'M-F', 'M-F Heat', 'M/F', 'Experimental', 'Summary'],
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
          Hide: true,
        },
        {
          Label: "Debug",
          Visible: false,
          Hide: true,
        },
        {
          Label: "Latex",
          Visible: false,
          Hide: true,
        },
        {
          Label: "Size",
          Visible: false,
          Hide: true,
        },
        {
          Label: "Adjective Stats",
          Visible: false,
        },
        {
          Label: "Explorer",
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
        // Profesiones: {
        //   Key: 'Profesiones',
        //   Label: 'Profesiones',
        //   Data: TestProfesiones,
        //   Link: 'https://www.cnae.com.es/lista-actividades.php',
        //   DataLink: 'https://www.ine.es/jaxiT3/Tabla.htm?t=4128'
        // }
      },

      // Static
      ChartTypes: ['radar', 'bar', 'line', 'doughnut', 'polarArea'],
    }
  },
  methods: {
    Redraw() {
      this.Loading = true;
      setTimeout(() => this.Loading = false, 100);
    },
    KVData(idx) {
      let modelm = this.Data[this.ModelIdx]["py/tuple"][idx];
      let values = [];
      this.Labels.forEach(label => values.push({ Key: label, Value: modelm[label][this.ColumnKey] }));
      return values;
    },
    SelectTab(label) {
      this.Tabs = this.Tabs.map((tab) => {
        tab.Visible = false;
        return tab;
      });
      this.Tabs.find(x => x.Label === label).Visible = true;
    },
    WriteLatex(text) {
      this.LatexContent = text;
      this.SelectTab("Latex");
    },
    ToggleColumn(col) {
      if (!this.ColumnMeta[col]) this.ColumnMeta[col] = {};
      this.ColumnMeta[col].Hide = !this.ColumnMeta[col].Hide
    },
    ToggleModel(model) {
      if (!this.ModelMeta[model]) this.ModelMeta[model] = {};
      this.ModelMeta[model].Hide = !this.ModelMeta[model].Hide
    },
    ToggleAllModels(hidden) {
      Object.keys(this.ModelMeta).forEach((meta) => this.ModelMeta[meta].Hide = hidden);
    },
    Cut(item) {
      item = item + "";
      if (!item.includes(".")) return item;

      let partesExponente = item.split("e");
      let exponente = "";
      if (partesExponente.length > 1) {
        item = partesExponente[0];
        exponente = partesExponente[1];
      }

      let parts = item.split(".");
      let res = parts[0] + "." + parts[1].slice(0, 2);
      if (exponente !== "")
        res = res + "e" + exponente;
      return res;
    },
    AsLatex() { },
    StyleHeatmap(column, val) {
      //let heatmapResult = this.CustomHeatMap(column, val);
      let heatmapResult = this.UseSymmetricHeatMap ? this.CustomHeatMapSymmetric(column, val) : this.CustomHeatMap(column, val);
      
      let heatMap = heatmapResult[0];
      let colorValue = heatmapResult[1];

      let color = "black";
      if (val > 0 && colorValue < 150)
        color = "white";

      return {
        'background-color': heatMap,
        color,
      };
    },
    HeatMap(column, val) {
      let normVal = this.Normalize(val, this.TableMin[column], this.TableMax[column]);
      let h = (1.0 - normVal) * 240
      return "hsl(" + h + ", 100%, 50%)";
    },
    CustomHeatMap(column, val) {
      return this.CommonHeatMap(val, this.TableMin[column], this.TableMax[column]);
    },
    CustomHeatMapSymmetric(column, val) {

      let limit1 = this.TableMin[column]; 
      if(limit1 < 0) limit1 = limit1*-1;

      let limit2 = this.TableMax[column]; 
      if(limit2 < 0) limit2 = limit2*-1;

      let limitBig = limit1 > limit2 ? limit1 : limit2;
      let limitMax = limitBig;
      let limitMin = limitBig * -1;

      return this.CommonHeatMap(val, limitMin, limitMax); 
    },
    CommonHeatMap(val, limitMin, limitMax){
      if (val < 0) {
        let normVal = this.Normalize(val, limitMin, 0);
        let h = (normVal * 250);
        return ["rgb(255," + h + "," + h + ")", h];
      } else {
        let normVal = this.Normalize(val, 0, limitMax);
        let h = (1 - normVal) * 250;
        return ["rgb(" + h + "," + h + ",255)", h];
      }
    },
    Normalize(val, min, max) {
      return (val - min) / (max - min);
    },
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

    SummaryRow(col) {
      let res = this.Models.map((model, mIdx) => {
        let val = this.DataTable[0][mIdx][col] - this.DataTable[1][mIdx][col];
        if (val === 0) return '';
        let tag = val > 0 ? 'Masculino' : 'Femenino';
        return tag;
      }).reduce((cnt, cur) => (cnt[cur] = cnt[cur] + 1 || 1, cnt), {});
      let latex = this.CurrentLabel + ' & ' + (res['Masculino'] ?? 0) + ' & ' + (res['Femenino'] ?? 0) + '  \\\\';
      return latex.replace("[", "").replace("]", "");
    },
    Mindex(model) {
      return this.ModelMeta[model].Index;
    },
    ColumnRename(col) {
      if (col === "prc_retrieval_status_value") return "% RSV";
      if (col === "prc_probability") return "% Probability";
      if (col === "probability") return "Probability";
      if (col === "retrieval_status_value") return "RSV";
      if (col === "count") return "Count";
      if (col === "prc_count") return "% Count";
      return col;
    },
    ForceRedrawTable(){
      this.RedrawingTable = true;
      setTimeout( () => this.RedrawingTable = false, 1);
    },
    ChangeHeatmapType(){
      this.UseSymmetricHeatMap = !this.UseSymmetricHeatMap;
      this.ForceRedrawTable();
    },
    GetValueMale(model, col){
      return this.GetValue(0, model, col);
    },
    GetValueFemale(model, col){
      return this.GetValue(1, model, col);
    },
    GetValue(classIndex, model, col){
      let dtClass = this.DataTable[classIndex];
      let dtModel = dtClass[this.Mindex(model)]

      if(!dtModel){
        this.CurrentLabel = '';
        return -1;
      }
      let val = dtModel[col];
      return val;
    }
    
  },

  computed: {
    Data() {
      let key = this.CurrentTestKey;
      return this.Dumps[key].Data;
    },
    DataTable() {
      let tableM = [];
      let tableF = [];
      let data = this.Data;
      // let models = this.Models;

      // table.push(" ");
      // for( let model in model)
      //   table.push(model);

      for (let i = 0; i < data.length; i++) {
        let m = this.Data[i]["py/tuple"][1];
        let f = this.Data[i]["py/tuple"][2];
        tableM.push(...Object.values(m).filter(xrow => xrow.cat === this.CurrentLabel));
        tableF.push(...Object.values(f).filter(xrow => xrow.cat === this.CurrentLabel));

        // tableM.push(...Object.values(m).map( row => Object.values(row).splice(1) ).filter( xrow => xrow[0] === this.CurrentLabel ));
        // tableF.push(...Object.values(f).map( row => Object.values(row).splice(1) ).filter( xrow => xrow[0] === this.CurrentLabel ));
      }

      return [tableM, tableF];
    },
    KeyedDataM() {
      return this.KVData(1);
    },
    KeyedDataF() {
      return this.KVData(2);
    },
    KeyedDiff() {
      let items = Object.values(this.KeyedDataM);
      let arr = [];

      for (let item of items) {
        arr.push({ Key: item.Key, Value: this.KeyedDataM.find(x => x.Key === item.Key).Value - this.KeyedDataF.find(x => x.Key === item.Key).Value });
      }
      return arr;
    },
    KeyedDiffColors() {
      let items = Object.values(this.KeyedDiff);
      let arr = [];

      for (let item of items) {
        let color = item.Value > 0 ? 'blue' : 'darkmagenta';
        arr.push({ Key: item.Key, Value: color });
      }
      return arr;
    },
    DataM() {
      return this.KVData(1).map(x => x.Value);
    },
    DataF() {
      return this.KVData(2).map(x => x.Value);
    },
    CurrentModelName() {
      return this.Data[this.ModelIdx]["py/tuple"][0];
    },
    Ok() {
      return this.ModelIdx !== -1 && this.ColumnKey !== '';
    },
    Labels() {
      let labels = Object
        .keys(this.Data[0]["py/tuple"][1])
        .filter(x => x !== "[_TOTAL]");
      if (this.RemoveQuestionMarks)
        return labels.filter(x => x !== "[???]");
      return labels;
    },
    Models() {
      return this.Data.map(modelResult => modelResult["py/tuple"][0]);
    },
    Columns() {
      return Object.keys(this.Data[0]["py/tuple"][1]["[???]"])
        .filter(x => !x.includes("py/"))
        .filter(x => x !== "cat");
    },
    VisibleColumns() {
      return this.Columns.filter(col => this.ColumnMeta[col]?.Hide !== true);
    },
    VisibleModels() {
      return this.Models.filter(col => this.ModelMeta[col]?.Hide !== true);
    },
    Tests() {
      return Object.values(this.Dumps);
    },
    CurrentModelLink() {
      return 'https://huggingface.co/' + this.CurrentModelName;
    },
    TableMax() {
      let t = this;
      let map = {};
      t.VisibleColumns.forEach(column => {
        let values = this.VisibleModels.map((model) =>this.GetValueMale(model, column) - this.GetValueFemale(model, column));
        map[column] = Math.max(...values);
      });
      return map;
    },
    TableMin() {
      let t = this;
      let map = {};
      t.VisibleColumns.forEach(column => {
        let values = this.VisibleModels.map((model) => this.GetValueMale(model, column) - this.GetValueFemale(model, column));
        map[column] = Math.min(...values);
      });
      return map;
    },
    // TableMin(){
    //   let t = this;

    //   return t.VisibleColumns.map( column => {
    //     let values = this.Models.map( (model, mIdx) => this.DataTable[0][mIdx][column] - this.DataTable[1][mIdx][column] );
    //     return {[column]: Math.min(...values)}
    //   });
    // },


  }

}
</script>

<style scoped>
/* Override */
.table > :not(caption) > * > * {
    border-bottom-width: 0px;
}
.nav {
  justify-content: center;
  margin-bottom: 10px;
  border-bottom: 2px solid #82c0f7;
}
/* Custom */
.bb1{
  border-bottom: 1px solid black !important;
}
.bb3{
  border-bottom: 3px solid black;
}

.br1{
  border-right: 1px solid black;
}

.br3{
  border-right: 3px solid black;
}

.mb1 {
  margin-bottom: 1px;
}
.mr1 {
  margin-right: 1px;
}

.mr5 {
  margin-right: 5px;
}
.mb5 {
  margin-bottom: 5px;
}
.mb10 {
  margin-bottom: 10px;
}
/* Override */
.table > :not(caption) > * > * {
  border-bottom-width: 0px;
}
.nav {
  justify-content: center;
}
/* Custom */
.bb1 {
  border-bottom: 1px solid black !important;
}
.bb3 {
  border-bottom: 3px solid black;
}

.br1 {
  border-right: 1px solid black;
}

.br3 {
  border-right: 3px solid black;
}

.bt3 {
  border-top: 2px solid black;
}

.b1 {
  border: 1px solid black;
}

.r {
  text-align: right;
}

pre,
h1,
li {
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
  margin-top: 15px !important;
}
</style>