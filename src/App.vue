<template>
  <div id="app" class="container">
    <header class="header">
      <h1>
        <em>Automatic Text Summarization</em>
        Buku Digital Berbahasa Indonesia
      </h1>
    </header>
    <div class="column-input" :class="{ avoid_clicks: inputDisable }">
      <div class="card">
        <h4 class="card-title">Unggah File PDF</h4>
        <div class="card-body">
          <div class="alert-info">
            <b>Informasi!</b>
            File PDF yang akan diunggah, perlu dilakukan proses seleksi, sehingga hanya tersisa teks yang akan diringkas.
          </div>
          <div class="upload-group">
            <label for="input-file-pdf" v-text="labelUpload"></label>
            <input type="file" id="input-file-pdf" ref="file" @change="previewPdf" />
          </div>
          <button
            type="button"
            class="button-submit"
            @click="uploadSummarizing()"
            v-if="showBtnUpload"
          >Meringkas</button>
        </div>
      </div>
      <div class="separator">atau</div>
      <div class="card">
        <h4 class="card-title">Pilih Buku</h4>
        <div class="card-body">
          <div class="accordion" v-for="book in books" v-bind:key="book.id">
            <div class="accordion-button" @click="book.expand = !book.expand">
              <h5>{{ book.title }}</h5>
            </div>
            <ul class="accordion-panel list-chapters" v-if="book.expand">
              <li v-for="chapter in book.chapters" v-bind:key="chapter.id">
                <input
                  type="radio"
                  :id="chapter.id"
                  name="choose"
                  @click="chooseBook(book, chapter)"
                />
                <label :for="chapter.id">{{ chapter.title }}</label>
              </li>
            </ul>
          </div>
          <button
            type="button"
            class="button-submit"
            @click="chooseSummarizing()"
            v-if="showBtnChoose"
          >Meringkas</button>
        </div>
      </div>
    </div>
    <div class="column-output">
      <div id="idle" class="output-idle" v-if="showIdle">
        <img src="./assets/idle.svg" />
      </div>
      <div id="loading" class="output-loading" v-if="showLoading">
        <img src="./assets/loading.gif" />
        <h4>Loading...</h4>
      </div>
      <div class="card" style="margin-bottom: 30px;" v-if="showPreviewPDF">
        <h4 class="card-title title-preview">
          <span>
            Pratinjau
            <span class="filename">{{ fileName }}</span>
          </span>
          <a href="#!" v-on:click="expandPreview = !expandPreview">
            <small>[Lihat/Sembunyikan]</small>
          </a>
        </h4>
        <div class="card-body" v-if="expandPreview">
          <div class="embedded-pdf">
            <embed :src="blobPdf" type="application/pdf" />
          </div>
        </div>
      </div>
      <div class="card" style="margin-bottom: 30px;" v-if="showResultSummary">
        <h4 class="card-title">
          <span>
            Hasil Rangkuman
            <span class="filename">{{ fileName }}</span>
          </span>
        </h4>
        <div class="card-body">
          <div class="panel">
            <div class="panel-header">
              <span id="timeResult" class="text-panel-header">Waktu: {{ time }} detik</span>
              <span class="button-panel-header">
                <button type="button" class="button-action" id="btn-export" @click="exportToPdf()">
                  <span>Jadikan PDF</span>
                </button>
              </span>
            </div>
            <div class="panel-body">
              <ul id="summaryResult">
                <li v-for="sentence in summary" :key="sentence">{{ sentence }}</li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import data from "./data/books.json";

export default {
  name: "app",
  data() {
    return {
      labelUpload: "Telusuri File",
      fileName: "",
      file: "",
      books: data.books,
      inputDisable: false,
      showIdle: true,
      showLoading: false,
      showPreviewPDF: false,
      showResultSummary: false,
      expandPreview: true,
      showBtnUpload: false,
      showBtnChoose: false,
      blobPdf: null,
      summary: [],
      time: null
    };
  },
  methods: {
    changeState(state) {
      switch (state) {
        case "idle":
          this.inputDisable = false;
          this.showIdle = true;
          this.showLoading = false;
          this.showPreviewPDF = false;
          this.showResultSummary = false;
          break;
        case "loading":
          this.inputDisable = true;
          this.showIdle = false;
          this.showLoading = true;
          this.showPreviewPDF = false;
          this.showResultSummary = false;
          break;
        case "preview":
          this.inputDisable = false;
          this.showIdle = false;
          this.showLoading = false;
          this.showPreviewPDF = true;
          this.showResultSummary = false;
          break;
        case "result":
          this.inputDisable = false;
          this.showIdle = false;
          this.showLoading = false;
          this.showPreviewPDF = true;
          this.showResultSummary = true;
          break;
        default:
          break;
      }
    },
    previewPdf(event) {
      this.file = event.target.files[0];
      this.labelUpload = this.file.name;
      this.fileName = this.file.name;
      this.blobPdf = URL.createObjectURL(this.file);
      this.showBtnUpload = true;
      this.changeState("preview");
    },
    chooseBook(b, ch) {
      this.file = ch.pathfile;
      this.showBtnChoose = true;
      this.fileName = b.title + " - " + ch.title;
      this.blobPdf = ch.pathfile;
      this.changeState("preview");
    },
    uploadSummarizing() {
      let formData = new FormData();
      formData.append("pdf", this.file);
      this.showBtnUpload = false;
      this.changeState("loading");
      this.axios
        .post("https://api-textrank.herokuapp.com/summarize", formData)
        .then(response => {
          this.changeState("result");
          this.expandPreview = false;
          this.summary = response.data.summary;
          this.time = response.data.time.toFixed(2);
        })
        .catch(err => console.log(err));
    },
    chooseSummarizing() {
      let formData = new FormData();
      formData.append("pathfile", this.file);
      this.showBtnChoose = false;
      this.changeState("loading");
      this.axios
        .post("https://api-textrank.herokuapp.com/summarize2", formData)
        .then(response => {
          this.changeState("result");
          this.expandPreview = false;
          this.summary = response.data.summary;
          this.time = response.data.time.toFixed(2);
        })
        .catch(err => console.log(err));
    },
    exportToPdf() {
      let formData = new FormData();
      formData.append("summary", this.summary.join(" "));
      this.axios
        .post("https://api-textrank.herokuapp.com/export_file", formData)
        .then(response => {
          const url = window.URL.createObjectURL(new Blob([response.data]));
          const link = document.createElement("a");
          link.href = url;
          link.setAttribute("download", "summary.pdf");
          document.body.appendChild(link);
          link.click();
        })
        .catch(err => console.log(err));
    }
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Overpass:ital,wght@0,400;0,700;1,400;1,700&display=swap");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: "Overpass", sans-serif;
}

body {
  background-color: #eee;
  line-height: 1.4;
}

.container {
  margin: 0 auto;
  width: 95%;
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 100px minmax(200px, auto) 1fr;
  grid-template-areas:
    "header"
    "col-out"
    "col-in";
  column-gap: 30px;
}

.header {
  grid-area: header;
  display: grid;
  min-height: 100px;
  align-items: center;
}

.header h1 {
  font-size: 1.2em;
  text-align: center;
}

.column-input {
  grid-area: col-in;
  margin-bottom: 30px;
}

.avoid_clicks > * {
  pointer-events: none;
}

.column-output {
  grid-area: col-out;
}

.output-idle,
.output-loading {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin-bottom: 30px;
}

.output-idle img {
  width: 75%;
}

.card {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 20px;
  box-shadow: 10px 10px 30px rgba(51, 51, 51, 0.1);
}

.card-title {
  display: flex;
  justify-content: space-between;
  flex-direction: column;
  font-size: 0.8em;
}

.filename {
  color: #10ac84;
}

.card-title a {
  color: #2e86de;
  text-decoration: none;
}

.card-body {
  margin-top: 10px;
}

.alert-info {
  background-color: rgba(72, 219, 251, 0.3);
  border: 2px solid rgba(72, 219, 251, 1);
  border-radius: 10px;
  font-size: 0.8em;
  padding: 5px 10px;
  margin-bottom: 5px;
}

.upload-group {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 75px;
  background-color: rgba(200, 214, 229, 50%);
  border: 2px dashed #8395a7;
  border-radius: 15px;
  margin-bottom: 5px;
  padding: 15px;
}

.upload-group label {
  font-size: 0.8em;
  font-weight: bold;
  color: #8395a7;
  cursor: pointer;
}

.upload-group label:hover {
  color: #60676e;
}

.upload-group input[type="file"] {
  opacity: 0;
  position: fixed;
  width: 0;
}

.button-submit {
  display: block;
  width: 100%;
  min-height: 30px;
  background-color: #1dd1a1;
  border: 2px solid #10ac84;
  border-radius: 50px;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

.button-submit:hover {
  background-color: #10ac84;
}

/* Separator */
.separator {
  display: flex;
  align-items: center;
  text-align: center;
  min-height: 50px;
  color: #8395a7;
  font-weight: bold;
}

.separator::before,
.separator::after {
  content: "";
  flex: 1;
  border-bottom: 2px solid #8395a7;
}

.separator::before {
  margin-right: 0.25em;
}

.separator::after {
  margin-left: 0.25em;
}

.accordion {
  min-height: 40px;
  padding: 10px 15px;
  background-color: rgba(200, 214, 229, 50%);
  border: 2px solid #8395a7;
  border-radius: 10px;
  margin-bottom: 5px;
}

.accordion-button {
  cursor: pointer;
  border-bottom: 2px solid rgba(200, 214, 229, 0%);
}

.accordion-button:hover {
  border-bottom: 2px solid #8395a7;
}

.accordion-button h5 {
  font-size: 14px;
}

.accordion-panel {
  margin-top: 10px;
}

.list-chapters li {
  list-style: none;
}

.list-chapters input[type="radio"] {
  opacity: 0;
  position: fixed;
  width: 0;
}

.list-chapters label {
  display: block;
  background-color: rgba(200, 214, 229, 0.5);
  border: 2px solid rgba(200, 214, 229, 0.5);
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 10px;
  font-weight: bold;
  font-size: 13px;
  margin-bottom: 5px;
}

.list-chapters input[type="radio"]:checked + label {
  background-color: #1dd1a1;
  border: 2px solid #10ac84;
  color: white;
}

.list-chapters label:hover {
  background-color: rgba(200, 214, 229, 0.8);
}

.embedded-pdf embed {
  border-radius: 15px;
  width: 100%;
  height: 400px;
}

.panel {
  background-color: rgba(200, 214, 229, 0.5);
  border: 2px solid #8395a7;
  border-radius: 15px;
}

.panel-header {
  display: flex;
  min-height: 40px;
  padding: 0 20px;
  align-items: center;
  justify-content: space-between;
  background-color: #8395a7;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}

.text-panel-header {
  font-size: 13px;
  font-weight: bold;
  color: white;
}

.button-panel-header button {
  margin: 0 5px;
}

.button-action {
  color: white;
  min-height: 27px;
  background-color: #576574;
  border: 2px solid #222f3e;
  border-radius: 50px;
  padding: 0 10px;
  cursor: pointer;
  font-weight: bold;
}

.button-action:hover {
  background-color: #222f3e;
}

.panel-body {
  padding: 20px;
}

.panel-body li {
  list-style: none;
  padding: 10px 0;
}

#summaryResult li {
  font-size: 0.8em;
}

@media screen and (min-width: 768px) {
  .container {
    width: 80%;
    grid-template-columns: 1fr 2fr;
    grid-template-rows: 100px 1fr;
    grid-template-areas:
      "header header"
      "col-in col-out";
  }

  .header h1 {
    font-size: 1.8em;
    text-align: left;
  }

  .card-title {
    flex-direction: row;
    font-size: 1em;
  }

  .upload-group label {
    font-size: 1em;
  }

  .output-idle,
  .output-loading {
    height: 550px;
  }

  .embedded-pdf embed {
    height: 550px;
  }
}
</style>
