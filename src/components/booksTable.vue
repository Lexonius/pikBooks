<template>
  <el-container class="flex">
    <div class="header">
      <el-button
        type="primary"
        class="buttonAdd"
        @click="addBook"
        icon="el-icon-plus"
        size="small"
      >Add book</el-button>
      <el-input
        placeholder="Please input"
        v-model="serachInput"
        @input="filterAllBooks"
        class="searchInput"
        size="small"
      >></el-input>
    </div>
    <el-table
      :data="tableData"
      border
      v-loading="loading"
      style="width: 100%"
      @row-click="clickRowTable"
      size="small"
      :default-sort="{prop: 'author', order: 'ascending'}"
      class="tableBooks"
    >
      <el-table-column prop="author" label="author" sortable></el-table-column>
      <el-table-column prop="name" label="name"></el-table-column>
    </el-table>
    <div v-bind:style="{display: activeDisplay}" class="modalWindow">
      <div class="wrapper">
        <div class="top">
          <i class="el-icon-close" @click="closeModalWindow"></i>
        </div>
        <el-main>
          <div class="inputText">Id*</div>
          <el-input
            placeholder="Please input"
            v-model="chooseBook.id"
            size="small"
            :disabled="disabledFields"
          ></el-input>
          <div class="inputText">Name*</div>
          <el-input
            placeholder="Please input"
            v-model="chooseBook.name"
            size="small"
            :disabled="disabledFields"
          ></el-input>
          <div class="inputText">Author*</div>
          <el-input
            placeholder="Please input"
            v-model="chooseBook.author"
            size="small"
            :disabled="disabledFields"
          ></el-input>
          <div class="inputText">Country</div>
          <el-input
            placeholder="Please input"
            v-model="chooseBook.country"
            size="small"
            :disabled="disabledFields"
          ></el-input>
          <div class="inputText">Language</div>
          <el-input
            placeholder="Please input"
            v-model="chooseBook.language"
            size="small"
            :disabled="disabledFields"
          ></el-input>
          <div class="inputText">Year</div>
          <el-input
            placeholder="Please input"
            v-model="chooseBook.year"
            size="small"
            :disabled="disabledFields"
          ></el-input>
          <div class="inputText">Pages</div>
          <el-input
            placeholder="Please input"
            v-model="chooseBook.pages"
            size="small"
            :disabled="disabledFields"
          ></el-input>
        </el-main>
        <div class="buttons">
          <el-button
            type="primary"
            size="small"
            @click="changeFields"
            v-bind:style="{display: activeButtonChange}"
          >Change</el-button>
          <el-button
            type="warning"
            size="small"
            @click="cancelChangedFields"
            v-bind:style="{display: activeButtonCancel}"
          >Cancel</el-button>
          <el-button
            type="danger"
            size="small"
            v-bind:style="{display: activeButtonDelete}"
            @click="deleteBook"
          >Delete</el-button>
          <el-button
            type="success"
            size="small"
            v-bind:style="{display: activeButtonSave}"
            @click="saveChanges"
          >Save</el-button>
        </div>
      </div>
    </div>
    <div class="block">
      <el-pagination
        layout="prev,pager,next"
        :total="amountBooks"
        :page-size="10"
        @current-change="clickPagPanel"
      ></el-pagination>
    </div>
  </el-container>
</template>

<script>
import { async, nextTick } from "q";
import axios from "axios";
import { requestAddressValid } from "./../config";
import { requestAddress } from "./../config";

export default {
  name: "booksTable",
  props: {
    msg: String
  },

  data() {
    return {
      loading: true,
      tableData: [],
      amountBooks: 0,
      activeDisplay: "none",
      chooseBook: "",
      disabledFields: true,
      activeButtonChange: "inline",
      activeButtonCancel: "none",
      activeButtonDelete: "inline",
      activeButtonSave: "none",
      disabledButtonSave: true,
      serachInput: ""
    };
  },

  methods: {
    getAllBooks: async function(num) {
      const request = await axios.post(requestAddress, {
        filters: [
          {
            field: "id",
            value: "0",
            exp: "!="
          }
        ],
        fields: "id,name,author,year,country,language,pages",
        pageSize: 14,
        page: `${num}`,
        allObjects: true
      });
      let booksArr = request.data.result.list;
      booksArr.forEach(element => {
        let booksField = {
          id: element.obj.id,
          author: element.obj.author,
          country: element.obj.country,
          language: element.obj.language,
          name: element.obj.name,
          pages: element.obj.pages,
          year: element.obj.year
        };
        this.tableData.push(booksField);
        this.amountBooks = request.data.result.pageInfo.tableSize;
      });
    },

    getFilterBooks: async function(num) {
      let searchRequest = await axios.post(requestAddress, {
        filters: [
          {
            operator: "OR",
            filters: [
              {
                value: `${this.serachInput}`,
                field: "author",
                exp: "like"
              },
              {
                value: `${this.serachInput}`,
                field: "name",
                exp: "like"
              }
            ]
          }
        ],
        fields: "id,name,author,year,country,language,pages",
        pageSize: 14,
        allObjects: true,
        page: `${num}`
      });
      this.paginationAllBooks === false;
      searchRequest.data.result.list.forEach(element => {
        let foundBooks = {
          id: element.obj.id,
          author: element.obj.author,
          country: element.obj.country,
          language: element.obj.language,
          name: element.obj.name,
          pages: element.obj.pages,
          year: element.obj.year
        };
        this.tableData.splice(0, 0, foundBooks);
      });
      this.amountBooks = searchRequest.data.result.pageInfo.tableSize;
    },

    filterAllBooks: async function() {
      if (this.serachInput === "") {
        this.tableData.splice(0, this.tableData.length);
        this.getAllBooks(0);
      } else {
        this.tableData.splice(0, this.tableData.length);
        this.getFilterBooks(0);
      }
    },

    clickPagPanel: async function(event) {
      if (event > 0) {
        event = event - 1;
      }

      if (this.serachInput === "") {
        console.log("pag all");
        // console.log(event);
        this.tableData.splice(0, this.tableData.length);
        this.getAllBooks(event);
      } else {
        console.log("pag filter");
        this.tableData.splice(0, this.tableData.length);
        this.getFilterBooks(event);
        console.log(this.tableData.length);
      }
    },

    addBook() {
      this.activeDisplay = "inline";
      this.disabledFields = false;
      this.chooseBook = {};
      this.activeButtonSave = "inline";
      this.activeButtonCancel = "none";
      this.activeButtonDelete = "none";
      this.activeButtonChange = "none";
      // console.log(this.chooseBook.id);
    },

    clickRowTable(event) {
      // console.log(event);
      this.activeDisplay = "inline";
      this.chooseBook = event;
      // console.log(this.bookObj);
      // console.log(this.tableData);
    },

    saveChanges: async function() {
      // console.log("book", this.chooseBook.id);
      if (
        this.chooseBook.id &&
        this.chooseBook.name &&
        this.chooseBook.author !== undefined
      ) {
        let updateRequest = await axios.post(requestAddressValid, {
          author: this.chooseBook.author,
          country: this.chooseBook.country,
          id: this.chooseBook.id,
          language: this.chooseBook.language,
          name: this.chooseBook.name,
          pages: this.chooseBook.pages,
          year: this.chooseBook.year,
          isDelete: false,
          isComplete: ""
        });
        // console.log("request", updateRequest.config.data);
        let newObj = JSON.parse(updateRequest.config.data);
        console.log(newObj);
        this.tableData.push(newObj);
        this.amountBooks = this.tableData.length;
        this.activeDisplay = "none";
      } else {
        this.$notify.error({
          title: "Error",
          message: "Please,Enter starred values"
        });
        return;
      }
    },

    deleteBook: async function() {
      let deleteBook = await axios.post(requestAddressValid, {
        id: this.chooseBook.id,
        isDelete: true
      });
      console.log(deleteBook.data.result.lastObjectID);
      let result = this.tableData.filter(
        elem => elem.id !== deleteBook.data.result.lastObjectID
      );
      console.log(result);
      this.tableData = result;
      this.amountBooks = result.length;
      this.activeDisplay = "none";
    },

    closeModalWindow() {
      this.activeDisplay = "none";
      this.disabledFields = true;
      this.activeButtonChange = "inline";
      this.activeButtonCancel = "none";
      this.activeButtonDelete = "inline";
      this.activeButtonSave = "none";
    },

    changeFields() {
      console.log("45");
      this.disabledFields = false;
      this.activeButtonChange = "none";
      this.activeButtonCancel = "inline";
      this.activeButtonDelete = "none";
      this.activeButtonSave = "inline";
    },

    cancelChangedFields() {
      (this.disabledFields = true), (this.activeButtonChange = "inline");
      this.activeButtonCancel = "none";
      this.activeButtonDelete = "inline";
      this.activeButtonSave = "none";
    }
  },

  computed: {},

  created: function() {
    this.getAllBooks(0);
    this.loading = false;
  }
};
</script>

<style scoped>
.active {
  display: none;
}
body {
  margin: 0;
}
.modalWindow {
  position: fixed;
  top: 10%;
  left: 35%;
  width: 30%;
  border: 0.5px solid #ebeef5;
  background-color: #ffffff;
  border-radius: 5px;
}
.header {
  display: flex;
  padding: 0;
  justify-content: space-between;
}
.buttons {
  padding-bottom: 10px;
}
.inputText {
  display: flex;
  font-size: 12px;
  padding: 5px 0px 5px 0px;
}
.flex {
  flex-direction: column;
}
.buttonAdd {
  width: 15%;
  margin-bottom: 15px;
}
.searchInput {
  width: 50%;
}
.tableBooks {
  position: initial;
}
.top {
  display: flex;
  justify-content: flex-end;
}
.el-icon-close {
  padding-top: 10px;
  padding-right: 10px;
}
.el-main {
  padding: 15px;
}
</style>
