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
        
        <template v-if="Ok">
          <h3>Valores M:</h3>
          <ul><li v-for="(item, idx) in KeyedDataM" :key="idx">{{item.Key}} : {{item.Value}}</li></ul>

          <h3>Valores F:</h3>
          <ul><li v-for="(item, idx) in KeyedDataF" :key="idx">{{item.Key}} : {{item.Value}}</li></ul>
        </template>

        <div class="form-check" v-if="Ok">
        <input class="form-check-input" type="checkbox" v-model="RemoveQuestionMarks" @change="Redraw">
        <label class="form-check-label" for="flexCheckDefault">
          Ocultar [???]
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
          v-if="Ok && !Loading"
          ></Radial>
      </div>

    </div>


  </div>

  <div class="row">
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
import Radial from './Radar.vue';

export default {
  name: 'HelloWorld',
  components: { Radial },
  created(){
    window.app = this;
    window.$data = this.$data;
    this.Models = this.Data.map( modelResult => modelResult["py/tuple"][0] );
    this.Columns = Object.keys(this.Data[0]["py/tuple"][1]["[???]"])
      .filter( x => !x.includes("py/") )
      .filter( x => x !== "cat" );
    this.Loading = false;

  },
  props: {
    msg: String
  },
  data(){
    return {
      Data: DataDump,

      Columns: [],
      Models: [],

      // Run
      ModelIdx: -1,
      ColumnKey: '',

      Loading: true,
      RemoveQuestionMarks: true,
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
    KeyedDataM(){
      return this.KVData(1);
    },
    KeyedDataF(){
      return this.KVData(2);
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
