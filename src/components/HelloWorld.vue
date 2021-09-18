<template>

  <div class="container">
    <div class="row">

      <div class="col-2">

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
            v-for="(col, colIdx) in Columns"
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

          <h3>Diff M-F:</h3>
          <ul><li v-for="(item, idx) in KeyedDiff" :key="idx"><span :style="{'color':KeyedDiffColors.find(x => x.Key === item.Key).Value}">{{item.Key}}</span> {{item.Value}}</li></ul>

        </template>

        <div class="form-check" v-if="Ok">
        <input class="form-check-input" type="checkbox" v-model="RemoveQuestionMarks" @change="Redraw">
        <label class="form-check-label" for="flexCheckDefault">
          Ocultar [???]
        </label>
        </div>


        <div class="form-check" v-if="Ok">
        <input class="form-check-input" type="checkbox" v-model="CargarDatosNegados" @change="Redraw">
        <label class="form-check-label" for="flexCheckDefault">
          Cargar frases Negadas
        </label>
        </div>

      </div>

      <div class="col-10">
      <h1 v-if="Ok">{{ CurrentModelName }}</h1>
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

  <div class="row" style="margin-top: 75px">
    <div class="col" v-if="Ok"> {{ DataM }}</div>
    <div class="col" v-if="Ok"> {{ DataF }}</div>
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
      Labels
      <pre>{{ Labels }}</pre>
    </div>

  </div>

</div>

</template>

<script>
/* eslint-disable no-debugger */
/* eslint-disable vue/no-unused-components */

import DataDump from './../../../StereoES/result_fillmask/categorias_polaridad_visibilidad/run_result.json'
import DataDumpNegada from './../../../StereoES/result_fillmask/categorias_polaridad_visibilidad_negadas/run_result.json'
import Radial from './Radar.vue';

export default {
  name: 'HelloWorld',
  components: { Radial },
  created(){
  },
  props: {
    msg: String
  },
  data(){
    return {
      NormalData: DataDump,
      NegativeData: DataDumpNegada,

      // Run
      ModelIdx: -1,
      ColumnKey: '',
      ChartType: 'radar',

      Loading: false,
      RemoveQuestionMarks: true,
      CargarDatosNegados: false,
      ChartTypes: ['radar', 'bar', 'line', 'doughnut', 'polarArea']
    }
  }, 
  methods: {
    Redraw(){
      this.Loading = true;
      setTimeout( () => this.Loading = false, 1);
    },
    KVData(idx){
      let modelm = this.Data[this.ModelIdx]["py/tuple"][idx];
      let values = [];
      this.Labels.forEach( label => values.push({ Key: label, Value: modelm[label][this.ColumnKey]}) );
      return values;
    },

  },
  computed: {
    Data(){
      let data = this.CargarDatosNegados ? this.NegativeData : this.NormalData;
      return data;
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
