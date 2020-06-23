<template>
  <div id="app">
    <h2>Upload PDF</h2>
    <input type="file" id="input_pdf" ref="file" v-on:change="handleFileUpload()" />
    <br />
    <button v-on:click="submitFile()">Submit</button>
    <br />
    <div v-if="loading">Loading..</div>
    <div v-if="showResult">
      <h2>Summary</h2>
      <ul>
        <li v-for="sentence in summary" :key="sentence">{{ sentence }}</li>
      </ul>
      <span v-text="time"></span>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      file: "",
      loading: false,
      showResult: false,
      summary: [],
      time: ""
    };
  },
  methods: {
    submitFile() {
      let formData = new FormData();
      formData.append("pdf", this.file);
      this.loading = true;
      this.$http
        .post("http://localhost:5000/api/v1.0/summarize", formData)
        .then(response => {
          this.loading = false;
          this.showResult = true;
          this.summary = response.data.summary;
          this.time = response.data.time;
        })
        .catch(err => console.log(err));
    },

    handleFileUpload() {
      this.file = this.$refs.file.files[0];
    }
  }
};
</script>
