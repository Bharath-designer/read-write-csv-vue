<template>
    <div class="wrapper">
        <div class="logo"><img src="./assets/csv.png" ></div>
        <div @click="newFile" class="nav-item">Create New Sheet</div>
        <label for="inp" class="nav-item">Open csv file</label>
        <input @change="handlefileChange" hidden id="inp" type="file">
        <button @click="exportFile" class="nav-item">Export csv file</button>
    </div>
</template>

<script>
    export default {
        methods:{
            handlefileChange(e) {
                this.$emit("fileChange",e)
            },
            exportFile() {
                this.$emit("exportFile")
            },
            newFile() {
                this.$emit("newFile")
            }
        }
    }
</script>

<style scoped>

.wrapper {
    display: flex;
    padding: 10px;
    align-items: center;
    gap: 20px;
}

.logo {
    width: 40px;
}
.logo img {
    width: 100%;
    pointer-events: none;
}

.nav-item{
    background:none;
    border: none;
    font-size: 14px;
    cursor: pointer;
    color: black;
}

</style>