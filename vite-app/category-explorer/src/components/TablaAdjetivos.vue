<template>
  <div class="col-4" style="border-right: 1px solid grey">
    <h3 class="text-right">Tipos/Clases</h3>
    <template v-for="(Type, midx) in Types" :key="midx">
      <a
        class="btn text-right"
        @click="Type.Enabled = !Type.Enabled"
        style="display: block"
      >
        {{ Type.Name }}
        <span v-if="Type.Enabled">🟩</span>
        <span v-if="!Type.Enabled">🟥</span>
      </a>
    </template>

    <h3 class="text-right">Columnas</h3>
    <template v-for="(Column, cidx) in Columns" :key="cidx">
      <a
        class="btn text-right"
        @click="Column.Enabled = !Column.Enabled"
        style="display: block"
      >
        {{ Column.Name }}
        <span v-if="Column.Enabled">🟩</span>
        <span v-if="!Column.Enabled">🟥</span>
      </a>
    </template>

    <h3 class="text-right">
      Modelos <small @click="ModelosInvertir()">(invertir selección)</small>
    </h3>
    <template v-for="(Model, midx) in Models" :key="midx">
      <a
        class="btn text-right"
        @click="Model.Enabled = !Model.Enabled"
        style="display: block"
      >
        {{ Model.Name }}
        <span v-if="Model.Enabled">🟩</span>
        <span v-if="!Model.Enabled">🟥</span>
      </a>
    </template>
  </div>
  <div class="col-8">
    <table class="table table-hover">
      <thead>
        <th
          v-for="(Column, cidx) in Cols"
          :key="cidx"
          @click="ToggleSort(Column.Name)"
        >
          {{ Column.Name }}
          <span v-if="SortBy == Column.Name && SortDirection == 'DSC'">
            🔽</span
          >
          <span v-if="SortBy == Column.Name && SortDirection == 'ASC'">
            🔼</span
          >
        </th>
      </thead>
      <tbody>
        <tr v-for="(Stat, sidx) in Rows" :key="sidx">
          <td v-for="(Column, cidx) in Cols" :key="cidx">
            <span v-if="typeof Stat[Column.Name] == 'number'">
              {{ Cut(Stat[Column.Name]) }}
            </span>
            <span v-else>{{ Stat[Column.Name] }}</span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import stats from "./../../../StereoES/result_data/FillMask/stats_by_model.json";

export default {
  created() {
    let model_names = [...new Set(this.Stats.map((x) => x.model))];
    let type_names = [...new Set(this.Stats.map((x) => x.type))];
    this.Models = model_names.map((x) => {
      return { Name: x, Enabled: true };
    });
    this.Types = type_names.map((x) => {
      return { Name: x, Enabled: true };
    });
    this.Columns = Object.keys(this.Stats[0]).map((x) => {
      return { Name: x, Enabled: true };
    });
    this.Columns.shift();
    //this.Columns = [...tCols, ...this.Columns];

    //this.Types.find(x => x.Name === "all" ).Enabled = false;
    //this.Types.find(x => x.Name === "global" ).Enabled = false;
    this.SortBy = this.Columns[1].Name;
  },
  data() {
    return {
      Stats: stats,
      SortBy: "",
      SortDirection: "DSC",
      Models: [],
      Types: [],
      Columns: [],
    };
  },
  methods: {
    Cut(item) {
      return (item + "").slice(0, 6);
    },
    ToggleSort(name) {
      if (this.SortBy == name) {
        if (this.SortDirection === "DSC") this.SortDirection = "ASC";
        else this.SortDirection = "DSC";
      } else {
        this.SortBy = name;
      }
    },
    ModelosInvertir() {
      this.Models.forEach((model) => (model.Enabled = !model.Enabled));
    },
  },
  computed: {
    Rows() {
      let enabled_types = this.Types.filter((x) => x.Enabled).map(
        (x) => x.Name
      );
      let enabled_models = this.Models.filter((x) => x.Enabled).map(
        (x) => x.Name
      );

      let data = this.Stats.filter((stat) =>
        enabled_types.includes(stat.type)
      ).filter((stat) => enabled_models.includes(stat.model));

      data.sort((a, b) => (a[this.SortBy] > b[this.SortBy] ? 1 : -1));
      if (this.SortDirection == "DSC") data = data.reverse();

      return data;
    },
    Cols() {
      return this.Columns.filter((x) => x.Enabled);
    },
  },
};
</script>

<style>
.text-right {
  text-align: right;
}
</style>
