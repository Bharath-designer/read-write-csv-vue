<template>
  <div class="wrapper">
    <div class="logo"><img src="./assets/csv.png" /></div>
    <div class="actions">
        <div class="row-1">
      <input class="file-name" :value="fileName" @input="this.$emit('fileNameChange',$event.target.value)" /> 
    </div>
    <div class="row-2">
      <div @click="newFile">Create New Sheet</div>
      <label for="inp">Open csv file</label>
      <input @change="handlefileChange" hidden id="inp" type="file" />
      <button @click="exportFile">Export csv file</button>
    </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ["fileName"],
  methods: {
    handlefileChange(e) {
      this.$emit("fileChange", e);
    },
    exportFile() {
      this.$emit("exportFile");
    },
    newFile() {
      this.$emit("newFile");
    },
  },
};
</script>

<style scoped>
.wrapper {
    display: flex;
    gap: 10px;
    padding: 10px;
    align-items: center;
}



.actions{
    display: flex;
    flex-direction: column;
    gap: 5px;
}

.file-name {
    font-size: 18px;
    color: rgb(60, 60, 60);
    padding: 0 8px;
    width: max-content;
}

.row-2 {
    display: flex;
}
.row-2 > * {
    font-size: 15px;
    border: none;
    background:none;
    cursor: pointer;
    padding: 5px 8px;
    color: rgb(72, 72, 72);
}
.row-2 > *:hover {
    background: rgb(218, 218, 218);
    border-radius: 4px;
}

.logo {
  width: 40px;
  user-select: none;
}
.logo img {
  width: 100%;
  pointer-events: none;
}



</style>
