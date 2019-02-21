<template>
  <el-container class="flex">
    <div class="authWindow" v-bind:style="{display:activeAuthWindow}">
      <div class="inputText">Login</div>
      <el-input
        placeholder="Please login"
        v-model="user.login"
        @input="filterAllBooks"
        class="searchInput"
        size="small"
      ></el-input>
      <div class="inputText">Password</div>
      <el-input
        placeholder="Please password"
        v-model="user.password"
        @input="filterAllBooks"
        class="searchInput"
        size="small"
      ></el-input>
      <el-button type="primary" class="okButton" @click="requestForUser" size="small">OK</el-button>
      <!-- <el-button
        type="primary"
        class="buttonAdd"
        @click="addBook"
        icon="el-icon-plus"
        size="small"
      >Add book</el-button>-->
    </div>
    <div class="table" v-bind:style="{display: activeTable}">
      <div class="header">
        <el-button
          type="primary"
          class="buttonAdd"
          @click="addBook"
          icon="el-icon-plus"
          size="small"
          v-bind:style="{display: activeButtonAdd}"
        >Add book</el-button>
        <el-input
          placeholder="Please input"
          v-model="serachInput"
          @input="filterAllBooks"
          class="searchInput"
          size="small"
        ></el-input>
      </div>
      <el-table
        :data="tableData"
        border
        style="width: 70%"
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
            <div class="textBlock">
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
            </div>
            <div class="bookImg">
              <!-- <el-upload
                class="avatar-uploader"
                :action="getAvatarBook"
                :show-file-list="false"
                :on-success="handleAvatarChange"
                :data='dataReq'
              >
              </el-upload>-->
              <img v-if="chooseBook.avatar" :src="chooseBook.avatar" class="avatarImage">
              <img v-else src="../assets/story.svg" class="avatarImageDefault">
              <div class="inputText cover" v-bind:style="{display: disabledCoverImg}">Сover
                <el-input placeholder="Please input" v-model="chooseBook.avatar" size="small"></el-input>
              </div>
              <div class="inputText cover" v-bind:style="{display: disabledCoverImg}">Link
                <el-input placeholder="Please input" v-model="chooseBook.link" size="small"></el-input>
              </div>

              <!-- <input type="file" @onchange='handleAvatarChange'>
              <input type="file" :onchange="handleAvatarChange()">-->
            </div>
          </el-main>
          <div class="buttons">
            <el-button
              type="success"
              size="small"
              v-bind:style="{display: activeGetButton}"
              @click="getThisBook"
            >Get this book</el-button>
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
    </div>
  </el-container>
</template>

<script>
import { async, nextTick } from "q";
import axios from "axios";
import { requestAddressValid } from "./../config";
import { requestAddress } from "./../config";
import { requestAddressValidText } from "./../config";
import { getUserAddress } from "./../config";

export default {
  name: "booksTable",
  props: {
    msg: String
  },

  data() {
    return {
      tableData: [],
      user: {
        login: "",
        password: "",
        role: "",
        isAuth: ""
      },
      amountBooks: 0,
      activeTable: "none",
      chooseBook: "",
      disabledFields: true,
      activeDisplay: "none",
      activeButtonChange: "inline",
      activeButtonCancel: "none",
      activeButtonDelete: "inline",
      activeButtonSave: "none",
      disabledButtonSave: true,
      serachInput: "",
      pagPage: "",
      activeGetButton: "none",
      disabledCoverImg: "inline",
      activeAuthWindow: "flex",
      activeButtonAdd: "inline"
    };
  },

  methods: {
    createNewUser: async function() {
      this.user.role = "user";
      let newUser = await axios.post(
        "https://directual.com/good/api/v3/struct/WebUser/?appID=d35b5f25-1215-411d-8775-d49cec6b4a3f&appSecret=ubOxrtQiduM",
        {
          role: this.user.role,
          email: this.user.login,
          password: this.user.password
        }
      );
    },

    requestForUser: async function() {
      let getUser = await axios.post(getUserAddress, {
        filters: [],
        fields: "role,email,password",
        pageSize: 10,
        orders: []
      });
      console.log(getUser.data.result.list);
      let adminInfo = getUser.data.result.list[1].obj;
      let userAuth = getUser.data.result.list.filter(
        elem =>
          elem.obj.email === this.user.login &&
          elem.obj.password === this.user.password
      );
      console.log(userAuth);

      if (userAuth.length === 0) {
        
        
        this.activeButtonAdd = 'none'
        this.createNewUser();
        console.log(this.user.role);
      } else if(
        this.user.login === userAuth[0].obj.email &&
        this.user.password === userAuth[0].obj.password        
      ){
          this.user.role = "user";
          console.log(this.user.role);
          this.activeButtonAdd = "none"        
      }

      if (
        adminInfo.email === this.user.login &&
        adminInfo.password === this.user.password
      ) {
        this.user.role = "admin";
        console.log(this.user.role);
        this.activeButtonAdd = 'inline'
      }

      if(this.user.login !== "" && this.user.password !== ""){
        this.activeAuthWindow = 'none'
        this.activeTable = 'flex' 
      }

    },

    getAllBooks: async function(num) {
      const request = await axios.post(requestAddress, {
        filters: [
          {
            field: "id",
            value: "",
            exp: "!="
          }
        ],
        fields: "id,name,author,year,country,language,pages,link,pic",
        pageSize: 10,
        page: `${num}`,
        allObjects: true
      });
      // console.log(request);

      let booksArr = request.data.result.list;
      booksArr.forEach(element => {
        let booksField = {
          id: element.obj.id,
          author: element.obj.author,
          country: element.obj.country,
          language: element.obj.language,
          name: element.obj.name,
          pages: element.obj.pages,
          year: element.obj.year,
          avatar: element.obj.pic,
          link: element.obj.link
        };
        this.tableData.push(booksField);
        console.log(this.tableData);
        if (booksField.link === "") {
          this.activeGetButton = "none";
        }
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
        fields: "id,name,author,year,country,language,pages,link,pic",
        pageSize: 10,
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
          year: element.obj.year,
          avatar: element.obj.pic,
          link: element.obj.link
        };
        this.tableData.splice(0, 0, foundBooks);
        if (foundBooks.link !== "") {
          this.activeGetButton = "none";
        }
      });
      this.amountBooks = searchRequest.data.result.pageInfo.tableSize;
    },

    // handleAvatarChange (file) {
    //   console.log(file);

    // },

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
      this.pagPage = event;
      if (this.serachInput === "") {
        // console.log("pag all");
        // console.log(event);
        this.tableData.splice(0, this.tableData.length);
        this.getAllBooks(event);
      } else {
        // console.log("pag filter");
        this.tableData.splice(0, this.tableData.length);
        this.getFilterBooks(event);
        // console.log(this.tableData.length);
      }
    },

    addBook() {
      if(this.user.role === 'admin'){
        this.activeDisplay = "inline";
        this.disabledFields = false;
        this.chooseBook = {};
        this.activeButtonSave = "inline";
        this.activeButtonCancel = "none";
        this.activeButtonDelete = "none";
        this.activeButtonChange = "none";
        this.activeGetButton = "none";
        this.disabledCoverImg = "inline";
      } 
      // console.log(this.chooseBook.id);
    },

    clickRowTable(event) {
        this.chooseBook = event;
      if(this.user.role === 'admin'){
        // console.log(event);
        this.activeDisplay = "inline";
        // console.log(this.bookObj);
        // console.log(this.tableData);
        this.activeGetButton = "inline";
        this.disabledCoverImg = "none";
        this.activeButtonCancel = "none";
        this.activeButtonChange = "inline";    
           
      } else {
        this.activeDisplay = "inline";
        this.activeGetButton = "none";
        this.disabledCoverImg = "none";
        this.activeButtonCancel = "none";
        this.activeButtonChange = "none";
        this.activeButtonDelete = 'none'
      }
    },

    saveChanges: async function() {
      if (
        this.chooseBook.id &&
        this.chooseBook.name &&
        this.chooseBook.author !== undefined
      ) {
        let updateRequest = await axios
          .post(requestAddressValid, {
            author: this.chooseBook.author,
            country: this.chooseBook.country,
            id: this.chooseBook.id,
            language: this.chooseBook.language,
            name: this.chooseBook.name,
            pages: this.chooseBook.pages,
            year: this.chooseBook.year,
            bookAvatar: this.chooseBook.avatar,
            link: this.chooseBook.link,
            isDelete: false,
            isComplete: ""
          })
          .then(response => {
            this.activeButtonCancel = "none";
            this.activeButtonChange = "inline";
            this.disabledFields = true;
            this.disabledCoverImg = "none";
            // console.log(response);
            const respText = response.data.result.obj.obj.respText;
            // console.log(respText);
            this.tableData.splice(0, this.tableData.length);
            this.getAllBooks(this.pagPage);
            this.activeDisplay = "none";
            this.$notify({
              title: "Успешно",
              message: respText,
              type: "success"
            });
          })
          .catch(e => {
            this.$notify.error({
              title: "Ошибка",
              message: "Сохранение не выполнено"
            });
          });
      } else {
        this.$notify.error({
          title: "Ошибка",
          message: "Заполните необходимые поля"
        });
        return;
      }
    },

    deleteBook: async function() {
      let deleteBook = await axios
        .post(requestAddressValid, {
          id: this.chooseBook.id,
          isDelete: true
        })
        .then(response => {
          // let result = this.tableData.filter(
          //   elem => elem.id !== response.data.result.lastObjectID
          // );
          // console.log(result);
          // this.tableData = result;
          // this.amountBooks = result.length;
          this.tableData.splice(0, this.tableData.length);
          this.getAllBooks(this.pagPage);
          this.activeDisplay = "none";
          const respText = response.data.result.obj.obj.respText;
          this.$notify({
            title: "Успешно",
            message: respText,
            type: "success"
          });
          // console.log(response);
        })
        .catch(e => {
          // console.log(e);
          this.$notify.error({
            title: "Ошибка",
            message: "Удаление не выполнено"
          });
        });
    },

    closeModalWindow() {
      this.activeDisplay = "none";
      this.disabledFields = true;
      this.activeButtonChange = "inline";
      this.activeButtonCancel = "none";
      this.activeButtonDelete = "inline";
      this.activeButtonSave = "none";
      this.disabledCoverImg = "none";
    },

    changeFields() {
      this.disabledFields = false;
      this.activeButtonChange = "none";
      this.activeButtonCancel = "inline";
      this.activeButtonDelete = "none";
      this.activeButtonSave = "inline";
      this.disabledCoverImg = "inline";
    },

    cancelChangedFields() {
      (this.disabledFields = true), (this.activeButtonChange = "inline");
      this.activeButtonCancel = "none";
      this.activeButtonDelete = "inline";
      this.activeButtonSave = "none";
      this.disabledCoverImg = "none";
    },

    getThisBook() {
      window.location = this.chooseBook.link;
    }
  },

  computed: {},

  created: function() {
    this.getAllBooks(0);
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
  top: 9%;
  left: 32%;
  width: 36%;
  border: 0.5px solid #ebeef5;
  background-color: #ffffff;
  border-radius: 5px;
}
.header {
  display: flex;
  padding: 0;
  justify-content: space-around;
  width: 85%;
}
.buttons {
  padding-bottom: 10px;
}
.inputText {
  display: flex;
  font-size: 10px;
  padding: 5px 0px 5px 0px;
}
.flex {
  flex-direction: column;
  align-items: center;
}
.buttonAdd {
  width: 15%;
  margin-bottom: 15px;
}
.searchInput {
  width: 50%;
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
  padding: 0px 0px 15px 15px;
  display: flex;
}
.avatarImage {
  height: 178px;
  display: block;
  padding-left: 50px;
}
.bookImg {
  padding: 20px 0px 0px 10px;
}
.cover {
  display: flex;
  flex-direction: column;
}
.el-table {
  position: initial;
}
.avatarImageDefault {
  height: 178px;
  display: block;
  padding: 0px 5px 0px 30px;
}
.table {
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.authWindow {
    position: fixed;
    top: 30%;
    left: 30%;
    width: 32%;
  border: 0.5px solid #ebeef5;
  background-color: #ffffff;
  border-radius: 5px;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.okButton {
  margin-top: 10px;
}
</style>
