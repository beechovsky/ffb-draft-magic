<template>
  <div id="app">
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <section class="hero">
      <header>
        <br>
        <h1><b>FFB DraftMagic© by <a href="https://github.com/beechovsky/ffb-draft-magic-vue" target="_blank">Jeff Bucklew</a></b></h1>
        <h3><i>Inspired by <a href="https://jayzheng.com/ff/" target="_blank">Jay Zheng's Draft Aid</a></i></h3>
        <!-- <h3>Steps to draft wizardry:</h3>
        <h3>Download a custom rankings CSV (<i>NOT</i> a cheatsheet) from <a href="https://www.fantasypros.com/nfl/rankings/consensus-cheatsheets.php" target="_blank">Fantasy Pros</a>.</h3>
        <h3>Upload it below.</h3>
        <h3>Click on a player to remove them from Rankings as they are drafted.</h3>
        <h3>If you clicked by accident, click that player on the Drafted table to return them.</h3> -->
        <br>
      </header>
    </section>
    <br>
    <upload v-if="this.showUpload === true" @load="setRows" id="uploadButton" class="visible"></upload>
    <br>
    <br>
    <div class="search">
      <div v-if="this.rankings.length > 0">
        <div class="orange">
          <label><b>Search Rankings:</b></label>
          <input type="text" v-model="search" placeholder="Player Name"/>
        </div>
      </div>
    </div>
    <br>
    <div class="search results">
      <table>
        <tbody>
          <!-- eslint-disable-next-line -->
          <tr v-for="(row, index) in searchList" @click="hideRow(row, index, false)" class="clickable">
            <!-- eslint-disable-next-line -->
            <td v-for="columnData in row.split(',').splice(1, 3)">{{ columnData }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-if="this.rankings.length > 0" class="parent">
      <div class="child rankings">
        <th>
          <tr class="orange">
            Rankings
          </tr>
        </th>
        <table class="rankingsTable">
          <thead>
            <tr>
              <!-- td instead of th for aesthetics -->
              <!-- eslint-disable-next-line -->
              <th v-for="colHeader in columnHeaders" style="background-color: #f5f5f5;">{{ colHeader }}</th>
            </tr>
          </thead>
          <tbody>
            <!-- eslint-disable-next-line -->
            <tr v-for="(row, index) in mergeSort(this.rankings).splice(1, this.rankings.length)" @click="hideRow(row, index, true)" class="clickable">
              <!-- eslint-disable-next-line -->
              <td v-for="rank in row.split(',').splice(0, 1)">{{ rank }}</td><td v-for="columnData in row.split(',').splice(2, colCount)">{{ columnData }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="child rbs">
        <th>
          <tr class="orange">
            Running Backs
          </tr>
        </th>
        <table class="posTable">
          <tbody>
            <!-- eslint-disable-next-line -->
            <tr v-for="(row, index) in mergeSort(rbList)" @click="hideRow(row, index, false)" class="clickable">
              <!-- eslint-disable-next-line -->
              <td v-for="name in row.split(',').splice(2, 1)">{{ name }}</td><td v-for="pos in row.split(',').splice(4, 1)">{{ pos }}</td>
            </tr>
          </tbody>
        </table>
        <br>
        <th>
          <tr class="orange">
            Quarter Backs
          </tr>
        </th>
        <table class="posTable">
          <tbody>
            <!-- eslint-disable-next-line -->
            <tr v-for="(row, index) in mergeSort(qbList)" @click="hideRow(row, index, false)" class="clickable">
              <!-- eslint-disable-next-line -->
              <td v-for="name in row.split(',').splice(2, 1)">{{ name }}</td><td v-for="pos in row.split(',').splice(4, 1)">{{ pos }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="child wrs">
        <th>
          <tr class="orange">
            Wide Receivers
          </tr>
        </th>
        <table class="posTable">
          <tbody>
            <!-- eslint-disable-next-line -->
            <tr v-for="(row, index) in mergeSort(wrList)" @click="hideRow(row, index, false)" class="clickable">
              <!-- eslint-disable-next-line -->
              <td v-for="name in row.split(',').splice(2, 1)">{{ name }}</td><td v-for="pos in row.split(',').splice(4, 1)">{{ pos }}</td>
            </tr>
          </tbody>
        </table>
        <br>
        <th>
          <tr class="orange">
            Tight Ends
          </tr>
        </th>
        <table class="posTable" style="width: 100%">
          <tbody>
            <!-- eslint-disable-next-line -->
            <tr v-for="(row, index) in mergeSort(teList)" @click="hideRow(row, index, false)" class="clickable">
              <!-- eslint-disable-next-line -->
              <td v-for="name in row.split(',').splice(2, 1)">{{ name }}</td><td v-for="pos in row.split(',').splice(4, 1)">{{ pos }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div v-if="this.drafted.length > 0" class="child drafted">
        <th>
          <tr class="orange">
            Drafted
          </tr>
        </th>
        <table>
          <tbody>
            <!-- eslint-disable-next-line -->
            <tr v-for="(row, index) in this.drafted" @click="putBack(row, index)" class="clickable">
              <!-- eslint-disable-next-line -->
              <td v-for="name in row.split(',').splice(2, 1)">{{ name }}</td><td v-for="pos in row.split(',').splice(4, 1)">{{ pos }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import Upload from './components/Upload'

export default {
  name: 'app',
  data: () => ({
    showUpload: true,
    rankings: [],
    drafted: [],
    columnHeaders: [],
    colCount: 0,
    search: ''
  }),
  computed: {
    searchList () {
      if (this.search.length > 3) { // hacky way of keeping results table from displaying persistently and limiting the size of the results eventually displayed
        return this.rankings.filter(player => {
          return player.toLowerCase().includes(this.search.toLowerCase())
        })
      }
    },
    rbList () {
      let rbs = []
      this.rankings.filter(player => {
        if (player.split(',')[4].includes('RB')) {
          rbs.push(player)
        }
      })
      return rbs
    },
    wrList () {
      let wrs = []
      this.rankings.filter(player => {
        if (player.split(',')[4].includes('WR')) {
          wrs.push(player)
        }
      })
      return wrs
    },
    qbList () {
      let qbs = []
      this.rankings.filter(player => {
        if (player.split(',')[4].includes('QB')) {
          qbs.push(player)
        }
      })
      return qbs
    },
    teList () {
      let tes = []
      this.rankings.filter(player => {
        if (player.split(',')[4].includes('TE')) {
          tes.push(player)
        }
      })
      return tes
    }
  },
  methods: {
    setRows (rows) {
      this.showUpload = false

      // populate rankings
      for (var player in rows) {
        this.rankings.splice(rows.indexOf(player), 0, rows[player])
      }

      let headerString = this.rankings[this.rankings.length - 1] // headers magically at the bottom...
      this.columnHeaders = headerString.split(',')
      this.columnHeaders.splice(1, 1) // remove Tier column
      this.columnHeaders.splice(1, 1) // remove WISD column
      this.columnHeaders.splice(1, 1, 'Name') // rename FP's dumb column name 'Overall'
      this.colCount = this.columnHeaders.length
    },
    hideRow (row, index, fromRanks) {
      // from the main table
      if (fromRanks === true) {
        this.drafted.splice(0, 0, row)
        this.rankings.splice(index, 1)
      } else {
        // from positional tables - indexes differ so look for a match (could be any field, but ranking is unique and never more than 3 digits)
        this.rankings.filter((player, index) => {
          if (row.split(',')[0] === player.split(',')[0]) {
            this.drafted.splice(0, 0, row)
            this.rankings.splice(index, 1)
          }
        })
      }

      this.search = ''
    },
    putBack (row, index) {
      // tables are sorted, so no need for intelligence here
      this.rankings.splice(0, 0, row)
      this.drafted.splice(index, 1)
    },
    mergeSort (arr) {
      if (arr.length < 2) {
        return arr
      }

      let middle = parseInt(arr.length / 2)
      let left = arr.slice(0, middle)
      let right = arr.slice(middle, arr.length)

      return this.merge(this.mergeSort(left), this.mergeSort(right))
    },
    merge (left, right) {
      let result = []

      while (left.length && right.length) {
        if (parseInt(left[0].split(',')[0]) <= parseInt(right[0].split(',')[0])) {
          result.push(left.shift())
        } else {
          result.push(right.shift())
        }
      }
      while (left.length) {
        result.push(left.shift())
      }
      while (right.length) {
        result.push(right.shift())
      }
      return result
    }
  },
  components: {
    Upload
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  /* background-color: #A8A8A8; */
}
* {
  box-sizing: border-box;
}
.clickable {
  cursor: pointer;
}
/* banner */
.hero {
  height: auto; /* grows according to text - won't need updating if I move the instructions*/
  background-image: url(./assets/ffb-banner.jpg);
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  color: white;
}
/* flex */
.parent {
  display: flex;
  flex: wrap;
  justify-content: space-evenly;
}
.child {
  margin: .5%;
}
.child rankings {
  flex: 2 2 auto;
}
.child rbs {
  flex: 1; /* equivalent of flex: 1 1 auto */
}
.child wrs {
  flex: 1;
}
/* NOTE: no child class for QB & TE because they share teh width of the tables above them */

.child drafted {
  flex: 1;
}
.orange {
  border: none;
  padding: 0 0 0 5px;
  color: orange;
}
/* Keeping the table header from scrolling with the body breaks the column sizing. Explicitly set them. */
/* Scroll bar also borks things. Width is fudged a bit to keep things close. Proably use Datatables in the future. */
.rankingsTable thead tr {
  width: 99%;
  display: block;
  border: 1px solid black;
}
.rankingsTable tbody {
  width: 101%;
  display: block;
  height: 1000px;
  overflow:auto;
}
.rankingsTable thead tr th {
  width: 7%;
}
.rankingsTable thead tr th:nth-child(2) {
  width: 20%;
}
.rankingsTable tbody tr td {
  width: 7%;
}
.rankingsTable tbody tr td:nth-child(2) {
  width: 20%;
}
.posTable {
  display:block;
  overflow:auto;
  height:300px;
}
.posTable tbody tr td:first-child {
  width: 80%;
}
.search {
  display: flex;
  justify-content: center;
}
.search results{
  min-height: 100px;
}
table, td {
  border-collapse: collapse;
  background-color: #ffffff; /* keep this in case the page bg color is changed */
}
td {
  border: 1px solid black;
  padding: 1px 2px 1px 2px;
}
tr:hover {
  background-color: #f5f5f5;
}
</style>
