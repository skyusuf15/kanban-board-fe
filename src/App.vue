<template>
  <div>
    <base-modal
      v-if="isShowModal"
      @close="toggleModal"
      scrollable
      title="Modal Title"
    >
      <p class="text-sm leading-5 text-gray-500">

      </p>
      <template v-slot:footer>
        <span class="flex w-full rounded-md shadow-sm sm:ml-3 sm:w-auto">
          <button
            type="button"
            class="
              inline-flex
              justify-center
              w-full
              rounded-md
              border border-transparent
              px-4
              py-2
              bg-red-600
              text-base
              leading-6
              font-medium
              text-white
              shadow-sm
              hover:bg-red-500
              focus:outline-none
              focus:border-red-700
              focus:shadow-outline-red
              transition
              ease-in-out
              duration-150
              sm:text-sm
              sm:leading-5
            "
          >
            Deactivate
          </button>
        </span>
      </template>
    </base-modal>

    <div class="container mt-5">
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
                @click="toggleModal"
                :key="element.title"
              >
                {{ element.title }}
              </div>
            </draggable>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
//import draggable
import draggable from "vuedraggable";
import BaseModal from "./components/Modal";

// Import axios
import axios from "axios";

axios.defaults.baseURL = "https://bemo-test-be.herokuapp.com/api/";

export default {
  name: "bemo-board",
  components: {
    draggable,
    BaseModal,
  },
  data() {
    return {
      isShowModal: false,
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
    toggleModal() {
      this.isShowModal = !this.isShowModal;
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
</style>
