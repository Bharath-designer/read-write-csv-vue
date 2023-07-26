<template>
  <div class="App">
    <NavBar
      @fileChange="handleFile"
      @exportFile="exportFile"
      @newFile="newFile"
    />
    <div ref="dropListener" class="drop-listener"></div>
    <div class="table-wrapper">
      <table>
        <tbody>
          <tr>
            <td class="head"></td>
            <td
              v-if="data[0]"
              @click="handleColHeadClick"
              :class="`head col_head col_head${ind}`"
              :data-key="ind"
              :key="ind"
              v-for="(el, ind) in Math.max(data[0].length, minColumn)"
            >
              <span>c{{ ind + 1 }}</span>
            </td>
          </tr>
          <tr
            :key="`row${rowInd}`"
            :class="`row row${rowInd}`"
            v-for="(cols, rowInd) in data"
          >
            <td
              :data-key="rowInd"
              @click="handleRowHeadClick"
              :key="rowInd"
              :class="`head row_head row_head${rowInd}`"
            >
              <span>r{{ rowInd + 1 }}</span>
            </td>
            <td
              :data-key="`${rowInd}-${colInd}`"
              :key="`${rowInd}-${colInd}`"
              :class="`col col${colInd}`"
              v-for="(cell, colInd) in Math.max(cols.length, minColumn)"
              @click="handleCellClick"
              @dblclick="handleDblCellClick"
              tabindex="1"
              @focusin="$event.target.click()"
            >
              <input
                :spellcheck="false"
                :data-key="`${rowInd}-${colInd}`"
                :value="cols[colInd]"
                @input="inputChangeHandler"
                readonly
                type="text"
              />
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { toRaw } from "vue";
import NavBar from "./NavBar.vue";

let revertChangeFunc = () => {};

export default {
  components: { NavBar },
  data() {
    return {
      data: [],
      minColumn: 15,
      minRow: 15,
      sheetName: "",
    };
  },

  mounted() {
    fetch("./sheet.csv").then((res) => this.handleFile(undefined, res));

    let dropListener = this.$refs["dropListener"];

    document.addEventListener("dragenter", (e) => {
      e.preventDefault();
      dropListener.classList.add("active");
    });
    dropListener.addEventListener("dragleave", (e) => {
      dropListener.classList.remove("active");
    });
    document.addEventListener("dragover", (e) => {
      e.preventDefault();
    });
    document.addEventListener("drop", (e) => {
      e.preventDefault();
      dropListener.classList.remove("active");

      let files = e.dataTransfer.files;

      if (files.length != 1) {
        alert("Please upload one file");
        return;
      }

      this.handleFile(undefined, files[0]);
    });
  },

  methods: {
    makeFit() {
      if (this.data.length < this.minRow) {
        let diff = this.minRow - this.data.length;
        for (let i = 0; i < diff; i++) {
          this.data.push([]);
        }
      }
    },
    async handleFile(e, file) {
      if (!file) file = e.target.files[0];

      let data = await file.text();

      let temp = data.split("\r").map((el) => el.replace("\n", ""));

      temp = temp.map((str) => {
        let inter = [];
        let temp = "";
        let splitComma = true;

        for (let i = 0; i < str.length; i++) {
          if (splitComma && str[i] == ",") {
            inter.push(temp);
            temp = "";
          } else if (splitComma && str[i] == '"') {
            splitComma = false;
          } else if (!splitComma && str[i] == '"') {
            splitComma = true;
          } else {
            temp += str[i];
          }
        }
        if (temp) inter.push(temp);

        return inter;
      });

      this.data = temp;
      this.makeFit();
    },
    handleRowHeadClick(e) {
      revertChangeFunc();
      const key = e.currentTarget.getAttribute("data-key");
      const row = document.querySelector(`.row${key}`);
      row.style.setProperty("background", "#7C9D96");

      revertChangeFunc = () => {
        row.style.setProperty("background", "white");
      };
    },

    handleColHeadClick(e) {
      revertChangeFunc();
      const key = e.currentTarget.getAttribute("data-key");
      const cols = document.querySelectorAll(`.col${key}`);
      cols.forEach((col) => col.classList.add("col-head-selected"));
      revertChangeFunc = () => {
        cols.forEach((col) => col.classList.remove("col-head-selected"));
      };
    },

    handleCellClick(e) {
      revertChangeFunc(e.target);
      const [ri, ci] = e.currentTarget.getAttribute("data-key").split("-");
      const cell = document.querySelector(`[data-key="${ri}-${ci}"]`);
      const input = cell.firstChild;

      function handleKeyDown(event) {
        if (input.readOnly)
          if (
            (event.keyCode >= 65 && event.keyCode <= 90) || // A-Z
            (event.keyCode >= 48 && event.keyCode <= 57) || // 0-9
            (event.keyCode >= 186 && event.keyCode <= 192) || // Symbols on US keyboard layout
            (event.keyCode >= 219 && event.keyCode <= 222) // More symbols on US keyboard layout
          ) {
            input.setSelectionRange(0, input.value.length);
            input.removeAttribute("readonly");
            input.focus();
          }
      }

      document.addEventListener("keydown", handleKeyDown);

      revertChangeFunc = (newEle) => {
        if (newEle != input) input.setAttribute("readonly", "true");
        document.removeEventListener("keydown", handleKeyDown);
      };
    },
    handleDblCellClick(e) {
      const [ri, ci] = e.currentTarget.getAttribute("data-key").split("-");
      const cell = document.querySelector(`[data-key="${ri}-${ci}"]`);
      const input = cell.firstChild;
      input.focus();
      input.setSelectionRange(input.value.length, input.value.length);
      input.removeAttribute("readonly");
    },

    inputChangeHandler(e) {
      const [ri, ci] = e.currentTarget.getAttribute("data-key").split("-");
      this.data[ri][ci] = e.currentTarget.value;
    },
    exportFile() {
      let csvContent =
        "data:text/csv;charset=utf-8," +
        this.data
          .map((e) =>
            e.map((col) => {
              if (col.includes(",")) {
                return '"' + col + '"';
              }
              return col;
            })
          )
          .join("\r\n");
      let encodedUri = encodeURI(csvContent);
      let link = document.createElement("a");
      link.setAttribute("href", encodedUri);
      link.setAttribute("download", "my_data.csv");
      link.click();
    },
    newFile() {
      this.data = [[]];

      let r = parseInt(prompt("Number of Row"));
      let c = parseInt(prompt("Number of Column"));
      this.minRow = isNaN(r) ? 1 : r;
      this.minColumn = isNaN(c) ? 1 : c;
      this.makeFit();
    },
  },
};
</script>

<style>
.drop-listener {
  position: absolute;
  width: 100%;
  height: 100vh;
  height: 100svh;
  background-color: red;
  z-index: 1000;
  visibility: hidden;
}

.drop-listener.active {
  visibility: visible;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Arial, Helvetica, sans-serif;
}

.App {
  height: 100vh;
  height: 100svh;
  display: flex;
  flex-direction: column;
}

.table-wrapper {
  width: 100%;
  overflow: auto;
  flex: 1;
}

table {
  border-spacing: 0;
  overflow: auto;
}

td {
  border: 1px solid rgb(206, 206, 206);
}

td input {
  display: block;
  border: none;
  outline: none;
  cursor: text;
  background: transparent;
  width: 100%;
}

td input:read-only {
  cursor: default;
}

td input:read-only::selection {
  color: black;
}

.col.col-head-selected {
  background-color: rgba(175, 169, 255, 0.514);
  border-left: 1px solid black;
  border-right: 1px solid black;
}

.head {
  background-color: black;
  color: white;
  text-align: center;
  border: 1px solid white;
  cursor: pointer;
  position: sticky;
  top: 0;
  left: 0;
}

.col_head {
  min-width: 200px;
}

.col_head span {
  display: block;
  resize: horizontal;
  overflow: auto;
  min-width: 100%;
  min-height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.row_head span {
  min-height: 30px;
  resize: vertical;
  overflow: auto;
  min-width: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.head span::-webkit-resizer {
}

td:focus-within {
  outline: 2px solid rgb(73, 73, 73);
  outline-offset: -1px;
}
</style>
