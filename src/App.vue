<template>
  <div id="app" class="container">
    <header class="header">
      <h1>
        <em>Automatic Text Summarization</em>
        Buku Digital Berbahasa Indonesia
      </h1>
    </header>
    <div class="column-input">
      <div class="card">
        <h4 class="card-title">Unggah Dokumen PDF</h4>
        <div class="card-body">
          <div class="upload-group">
            <label for="input-file-pdf" v-text="labelUpload"></label>
            <input type="file" id="input-file-pdf" ref="file" @change="previewPdf" />
          </div>
          <button
            type="button"
            class="button-submit"
            v-on:click="uploadSummarizing()"
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
          <button type="button" class="button-submit" v-if="showBtnChoose">Meringkas</button>
        </div>
      </div>
    </div>
    <div class="column-output">
      <div id="idle" class="output-idle" v-if="showIdle">
        <img src="./assets/idle.svg" />
      </div>
      <div id="loading" class="output-loading" v-if="showLoading">
        <img :src="animLoading" />
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
            <embed :src="blobPdf" type="application/pdf" width="100%" height="550px" />
          </div>
        </div>
      </div>
      <div class="card" v-if="showResultSummary">
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
                <button type="button" class="button-action" id="btn-export">
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
import imageIdle from "./assets/idle.svg";
import animLoading from "./assets/loading.gif";

export default {
  name: "app",
  data() {
    return {
      imageIdle: imageIdle,
      animLoading: animLoading,
      labelUpload: "Browse File",
      fileName: "",
      file: "",
      books: [
        {
          id: "b1",
          title: "Artificial Intelligence",
          chapters: [
            {
              id: "c1",
              title: "Pendahuluan",
              pathfile:
                "https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf"
            },
            {
              id: "c2",
              title: "Logic",
              pathfile: ""
            }
          ],
          expand: false
        },
        {
          id: "b2",
          title: "Algoritma & Pemrograman Dasar",
          chapters: [
            {
              id: "c3",
              title: "Pendahuluan",
              pathfile: ""
            },
            {
              id: "c4",
              title: "Flowchart & Pseudocode",
              pathfile: ""
            },
            {
              id: "c5",
              title: "Pengenalan Bahasa C++",
              pathfile: ""
            }
          ],
          expand: false
        },
        {
          id: "b3",
          title: "Multimedia Pembelajaran Interaktif",
          chapters: [
            {
              id: "c6",
              title: "Multimedia",
              pathfile: ""
            },
            {
              id: "c7",
              title: "Prinsip Multimedia Pembelajaran",
              pathfile: ""
            }
          ],
          expand: false
        }
      ],
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
          this.showIdle = true;
          this.showLoading = false;
          this.showPreviewPDF = false;
          this.showResultSummary = false;
          break;
        case "loading":
          this.showIdle = false;
          this.showLoading = true;
          this.showPreviewPDF = false;
          this.showResultSummary = false;
          break;
        case "preview":
          this.showIdle = false;
          this.showLoading = false;
          this.showPreviewPDF = true;
          this.showResultSummary = false;
          break;
        case "result":
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
        .post("https://api-textrank.herokuapp.com/api/v1.0/summarize", formData)
        .then(response => {
          this.changeState("result");
          this.expandPreview = false;
          this.summary = response.data.summary;
          this.time = response.data.time.toFixed(2);
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
  width: 80%;
  display: grid;
  grid-template-columns: 1fr 2fr;
  grid-template-rows: 100px 1fr;
  grid-template-areas:
    "header header"
    "col-in col-out";
  column-gap: 30px;
}

.header {
  grid-area: header;
  display: grid;
  min-height: 100px;
  align-items: center;
}

.column-input {
  grid-area: col-in;
  margin-bottom: 30px;
}

.column-output {
  grid-area: col-out;
}

.output-idle,
.output-loading {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 630px;
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

.upload-group {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 75px;
  background-color: rgba(200, 214, 229, 50%);
  border: 2px dashed #8395a7;
  border-radius: 15px;
  margin-bottom: 5px;
}

.upload-group label {
  font-size: 16px;
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
</style>
