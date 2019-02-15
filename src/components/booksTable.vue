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
        @input="searchBook"
        class="searchInput"
        size="small"
      >></el-input>
    </div>
    <el-table
      :data="tableData"
      border
      style="width: 100%"
      @row-click="clickRowTable"
      size="small"
      :default-sort="{prop: 'author', order: 'ascending'}"
    >
      <el-table-column prop="author" label="author" sortable></el-table-column>
      <el-table-column prop="name" label="name"></el-table-column>
    </el-table>
    <div v-bind:style="{display: activeDisplay}" class="modalWindow">
      <div class="wrapper">
        <div class="header">
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
    layout="prev, pager, next"
    :total='amountBooks'
    :page-size="10"
    @prev-click='clickPagPanel'
    @next-click='clickPagPanel'
    @current-change='clickPagPanel'
    >
  </el-pagination>
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
      countPage: 0,
      tableData: [],
      amountBooks: 0,
      activeDisplay: "none",
      chooseBook: "",
      disabledFields: true,
      buttonText: "Отменить",
      buttonType: "info",
      activeButtonChange: "inline",
      activeButtonCancel: "none",
      activeButtonDelete: "inline",
      activeButtonSave: "none",
      disabledButtonSave: true,
      serachInput: ""
    };
  },

  methods: {
    searchBook: async function(event) {
      if(event === ""){
        console.log(this.amountBooks);
        // this.amountBooks = 101;
      } else {
        console.log("input", event);
        this.tableData.splice(0, this.tableData.length);
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
          pageSize: 10,
          // page: 0
        });
        // console.log("responce", searchRequest.data.result.list);
        searchRequest.data.result.list.forEach(element => {
          // console.log(element);
          // console.log(this.booksField);
          let foundBooks = {
            id: element.obj.id,
            author: element.obj.author,
            country: element.obj.country,
            language: element.obj.language,
            name: element.obj.name,
            pages: element.obj.pages,
            year: element.obj.year
          };
          console.log(foundBooks);
          this.tableData.splice(0, 0, foundBooks);
          this.amountBooks = this.tableData.length
        });        
      }

    },
    
    clickPagPanel: async function (event){
      console.log(event);
      this.tableData.splice(0,10)
      let paginationRequest = await axios.post(requestAddress, {
        filters: [
          {
            field: "id",
            value: "0",
            exp: "!="
          }
        ],
        fields: "id,name,author,year,country,language,pages",
        pageSize: 10,
        page: `${event}`,
        allObjects: true
      });

          
    // console.log('booksArr', booksArr);
    // console.log("tableData", this.tableData);

      paginationRequest.data.result.list.forEach(element => {
      // console.log(element.obj);
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
    });
    },

    
    // nextClick(event){
    //   console.log(event);
  
    // },

    // changePage(event){
    //   console.log(event);
      
    // },

    addBook() {
      this.activeDisplay = "inline";
      this.disabledFields = false;
      this.chooseBook = {};
      this.activeButtonSave = "inline";
      this.activeButtonCancel = "none";
      this.activeButtonDelete = "none";
      this.activeButtonChange = "none";
      console.log(this.chooseBook.id);
    },

    clickRowTable(event) {
      console.log(event);
      this.activeDisplay = "inline";
      this.chooseBook = event;
      console.log(this.bookObj);
      console.log(this.tableData);
    },

    saveChanges: async function() {
      console.log("book", this.chooseBook.id);
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
        this.amountBooks = this.tableData.length
        this.activeDisplay = "none";
      } else {
        this.$notify.error({
          title: "Error",
          message: "Please,Enter Starred Values"
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
      this.amountBooks = result.length
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

  created: async function() {
    const request = await axios.post(requestAddress, {
      filters: [
        {
          field: "id",
          value: "0",
          exp: "!="
        }
      ],
      fields: "id,name,author,year,country,language,pages",
      pageSize: 10,
      page: 0,
      allObjects: true
    });

    // console.log('request', request.data.result.pageInfo.tableSize);
    let booksArr = request.data.result.list;
    // console.log('booksArr', booksArr);
    // console.log("tableData", this.tableData);

    booksArr.forEach(element => {
      // console.log(element.obj);
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
  }
};
</script>

<style scoped>
.active {
  display: none;
}
.modalWindow {
  position: fixed;
  top: 10%;
  left: 25%;
  width: 600px;
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
  padding-bottom: 20px;
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
  margin-bottom: 10px;
}
.searchInput {
  width: 50%;
}
</style>
