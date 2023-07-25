<template>
  <div class="App">
    <NavBar @fileChange="handleFile" />
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
            tabindex="1"
            :data-key="`${rowInd}-${colInd}`"
            :key="`${rowInd}-${colInd}`"
            :class="`col col${colInd}`"
            v-for="(cell, colInd) in Math.max(cols.length, minColumn)"
            @click="handleCellClick"
            @dblclick="handleDblCellClick"
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
</template>

<script>
import NavBar from "./NavBar.vue";

let revertChangeFunc = () => {};

export default {
  components: { NavBar },
  data() {
    return {
      data: [],
      minColumn : 15
    };
  },

  mounted() {
    fetch("./sheet.csv")
      .then((res) => this.handleFile(undefined, res))
  },

  methods: {
    async handleFile(e,file) {
      
      if (!file)
      file = e.target.files[0];
      
      let data = await file.text();

      let temp = data.split("\r").map((el) => el.replace("\n", ""));
        console.log(temp);

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
          if (temp) inter.push(temp)

          return inter;
        });

        this.data = temp
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
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Arial, Helvetica, sans-serif;
}

table {
  border-spacing: 0;
  overflow: auto;
}

td {
  border: 1px solid rgb(206, 206, 206);
  padding: 2px;
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
