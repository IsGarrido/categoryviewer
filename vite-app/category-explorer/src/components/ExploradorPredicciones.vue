<template>
  <div class="col-2">
    <select v-model="CurrentModel" class="form-select">
      <option selected value="">- Modelo -</option>
      <option v-for="(Model, midx) in Models" :key="midx" :value="Model">
        {{ Model }}
      </option>
    </select>

    <select v-model="CurrentIndex" class="form-select">
      <option selected value="">- Sentence Pair -</option>
      <option v-for="(Index, cidx) in Indexes" :key="cidx" :value="Index">
        {{ Index }} {{ MaleSentences[cidx] }}
      </option>
    </select>

    Cantidad
    <input
      type="number"
      v-model="Quantity"
      class="form-control"
      @change="Redraw"
    />
  </div>
  <div
    class="col"
    v-for="(PredictionGroup, pgidx) in SelectedPredictions"
    :key="pgidx + Quantity"
  >
    <h3>{{ PredictionGroup.type }}</h3>
    <h5>{{ PredictionGroup.sentence }}</h5>
    <table class="table">
      <thead>
        <tr>
          <th>Index</th>
          <th>token_str</th>
          <th>score</th>
          <th>token</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(Prediction, pidx) in PredictionGroup.predictions"
          :key="pidx + Quantity"
        >
          <td class="td">{{ Prediction.index }}</td>
          <td class="td">{{ Prediction.token_str }}</td>
          <td class="td">{{ Prediction.score }}</td>
          <td class="td">{{ Prediction.token }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import predictions from "./../../../StereoES/result_data/FillMask/predictions.json";

export default {
  created() {
    this.Models = [...new Set(this.Predictions.map((x) => x.model))];
    this.Sentences = [...new Set(this.Predictions.map((x) => x.sentence))];
    this.MaleSentences = [
      ...new Set(
        this.Predictions.filter((x) => x.type === "male").map((x) => x.sentence)
      ),
    ];
    this.Types = [...new Set(this.Predictions.map((x) => x.type))];
    this.Indexes = [...new Set(this.Predictions.map((x) => x.sentence_index))];
  },
  data() {
    return {
      Models: [],
      Sentences: [],
      Types: [],
      Indexes: [],
      MaleSentences: [],

      Predictions: predictions,
      CurrentModel: "",
      CurrentSentence: "",
      CurrentType: "",
      CurrentIndex: "",
      Quantity: 10,

      ForceRedraw: false,
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
    SelectedPredictions() {
      if (this.CurrentModel === "" || this.CurrentIndex === "") return [];

      let filtered = this.Predictions.filter(
        (x) =>
          x.model === this.CurrentModel &&
          x.sentence_index === this.CurrentIndex
      );
      let sliced = filtered.map((group) => {
        let copy = JSON.parse(JSON.stringify(group));
        copy.predictions = copy.predictions.slice(0, this.Quantity);
        return copy;
      });
      return sliced;
    },
  },
};
</script>
