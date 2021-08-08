<template>
  
    <div class="container mt-5">
      <Modal v-show="isModalVisible" @close="closeModal">
        <template v-slot:header> This is a new modal header. </template>

        <template v-slot:body> This is a new modal body. </template>

        <template v-slot:footer> This is a new modal footer. </template>
      </Modal>

      <div class="row">
        <div class="col form-inline">
          <b-form-input
            id="input-1"
            v-model="newTask"
            required
            placeholder="Enter Task"
            @keyup.enter="addCard"
          ></b-form-input>
          <b-button @click="addCard" variant="primary" class="ml-3"
            >Add</b-button
          >
        </div>
        <div class="col form-inline">
          <b-form-input
            id="input-2"
            v-model="newColumn"
            required
            placeholder="Enter Column"
            @keyup.enter="addColumn"
          ></b-form-input>
          <b-button @click="addColumn" variant="primary" class="ml-3"
            >Create Column</b-button
          >
        </div>
        <div class="col-2 form-inline">
          <b-button @click="downloadQuery" variant="primary" class="ml-3"
            >DownLoad Database Query</b-button
          >
        </div>
      </div>

      <div class="row mt-5">
        <div
          class="col"
          v-for="[index, column] of Object.entries(columns)"
          :key="column.id"
        >
          <div class="p-2 alert alert-secondary">
            <button class="float-right" @click="deleteColumn(index)">❌</button>
            <h3>{{ column.title }}</h3>

            <draggable
              class="list-group kanban-column"
              :list="column.cards"
              @end="dragEnd"
              group="tasks"
            >
              <div
                class="list-group-item"
                v-for="element in column.cards"
                @click="showModal"
                :key="element.title"
              >
                {{ element.title }}
              </div>
            </draggable>
          </div>
        </div>
      </div>
    </div>
  
</template>

<script>
//import draggable
import draggable from "vuedraggable";

// Import axios
import axios from "axios";

import Modal from "./components/Modal.vue";

axios.defaults.baseURL = "https://bemo-test-be.herokuapp.com/api/";

export default {
  name: "bemo-board",
  components: {
    draggable,
    Modal,
  },
  data() {
    return {
      isModalVisible: false,
      newTask: "",
      newColumn: "",
      columns: [],
    };
  },
  methods: {
    getBoardData: function () {
      axios
        .get("column")
        .then((response) => (this.columns = response.data.data));
    },
    addCard: function () {
      if (this.newTask) {
        const cardData = {
          title: this.newTask,
          description: "",
          column_id: this.columns[0].id,
        };

        axios
          .post("card", cardData)
          .then((response) => this.columns[0].cards.push(response.data.data));

        this.newTask = "";
      }
    },
    deleteColumn: function (columnIndex) {
      axios
        .delete(`column/${this.columns[columnIndex].id}`)
        .then((response) => {
          if (response.data.data) {
            this.columns.splice(columnIndex, 1);
            this.$forceUpdate();
          }
        });
    },
    addColumn: function () {
      if (this.newColumn) {
        const data = {
          title: this.newColumn,
          slug: this.newColumn.toLowerCase().split(" ").join("_"),
          cards: [],
        };
        axios
          .post("column", data)
          .then((response) => this.columns.push(response.data.data));

        this.newColumn = "";
      }
    },
    dragEnd: function () {
      axios
        .put("update/all_data", { columns: this.columns })
        .then((response) => console.log(response.data.data));
    },
    downloadQuery: function () {
      axios
        .get("download/query")
        .then((response) => this.downloadAsFile(response.data));
    },
    downloadAsFile: function (data) {
      const url = window.URL.createObjectURL(new Blob([data]));
      const link = document.createElement("a");
      link.href = url;
      link.setAttribute("download", "query.sql");
      document.body.appendChild(link);
      link.click();
    },
    showModal: function () {
      this.isModalVisible = true;
    },
    closeModal: function () {
      this.isModalVisible = false;
    },
  },
  mounted: function () {
    this.getBoardData();
  },
};
</script>

<style>
/* light stylings for the kanban columns */
.kanban-column {
  min-height: 300px;
}

.modal-vue .overlay {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-vue .modal {
  position: relative;
  width: 300px;
  z-index: 9999;
  margin: 0 auto;
  padding: 20px 30px;
  background-color: #fff;
}

.modal-vue .close {
  position: absolute;
  top: 10px;
  right: 10px;
}
</style>
