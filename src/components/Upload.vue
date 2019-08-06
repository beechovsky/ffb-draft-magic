<template>
  <label class='upload-button'>
    <h1>Upload Rankings CSV</h1>
    <input type='file' @change='loadTextFromFile'>
  </label>
</template>

<script>
export default {
  name: 'Upload',
  methods: {
    parseFile (file) {
      let noQuotes = file.replace(/"/g, '')
      let noBlankCols = noQuotes.replace(/,,/g, ',') // for WISD column, requires removal of header in parent as well
      let parsedRankings = noBlankCols.split(/,[\r\n]/)

      let rows = []
      rows = parsedRankings[0].split(/\r\n/)

      this.$emit('load', rows)
    },
    loadTextFromFile (event) {
      if (!event.target.files[0]) {
        return
      }

      let file = event.target.files[0]
      let reader = new FileReader()

      reader.onload = e => this.parseFile(e.target.result)
      reader.readAsText(file)
    }
  }
}
</script>

<style>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.upload-button {
  position: relative;
  overflow: hidden;
  display: inline-block;
  color: orange;

  border: 2px solid black;
  border-radius: 5px;
  padding: 8px 12px;
  cursor: pointer;
}
.upload-button:hover {
  background-color: #f5f5f5;
}
.upload-button input {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
  opacity: 0;
}
</style>
