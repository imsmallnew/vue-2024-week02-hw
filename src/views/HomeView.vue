<template>
    <div id="app" class="container">
        <header>
        <h1>TodoList</h1>
        <ul>
            <li v-if="!loginStatus"><a href="javascript:;"  @click="tabClick(1)" class="btn">註冊帳號</a></li>
            <li v-if="!loginStatus"><a href="javascript:;"  @click="tabClick(2)" class="btn">登入系統</a></li>
            <li v-if="loginStatus"><a href="javascript:;"  @click="signOut" class="btn">登出系統</a></li>
        </ul>
        </header>
        <div v-if="!loginStatus && tablValue === 1">
            <h2>註冊帳號</h2>
            信箱: <input type="text" placeholder="Email" v-model="signUpData.email"><br>
            密碼: <input type="text" placeholder="Password" v-model="signUpData.password"><br>
            暱稱: <input type="text" placeholder="NickName" v-model="signUpData.nickname"><br><br>
            <button type="button" @click="signUp" class="btn2">註冊</button>
            <!-- <p>{{ signUpData }}</p> -->
            <p>{{ signUpResponse }}</p>
            <hr />
        </div>    
        <div v-if="!loginStatus && tablValue === 2">
            <h2>登入系統</h2>
            信箱: <input type="text" placeholder="Email" v-model="signInData.email"><br>
            密碼: <input type="text" placeholder="Password" v-model="signInData.password"><br><br>
            <button type="button" @click="signIn" class="btn2">登入</button>
            <!-- <p>{{ signInData }}</p> -->
            <p>{{ signInResponse }}</p>
            <hr />
        </div>   
        <div>
            <div v-if="user.uid">
                <p>歡迎用戶 {{ user.nickname }} 登入</p>
                <p>UID: {{ user.uid }}</p>
            </div>
            <div v-else>
                <!-- <p>請使用帳號密碼登入或註冊新帳號!!</p> -->
            </div>
        </div>    
        <!-- <div>
            <h2>登出</h2>
            <button type="button" @click="signOut" :disabled="!loginStatus">登出</button>
            <p>結果: {{ signOutResponse }}</p>
            <p>loginstatus:{{ loginStatus }}</p>
            <hr />
        </div>     -->
        <div v-if="loginStatus">
            <!-- <h2>TODO LIST AREA</h2>-->            
            <div v-if="loginStatus">
              <h2 v-if="editorTitle">===={{ editorTitle }}====</h2>
                <table v-if="editItems.length > 0">
                <thead>
                    <tr>
                        <td width="66%">待辦事項</td>
                        <!-- <td width="15%">狀態</td> -->
                        <td width="20%"></td>
                    </tr>
                </thead>
                <tbody v-for="edit in editItems" :key="edit.id">
                    <tr>
                    <td width="66%"><input type="text" v-model="edit.content" value="edit.content" style="width:90%"></td>
                    <!-- <td width="15%">{{ edit.status ? '已完成' : '未完成' }}</td> -->
                    <td width="20%">
                        <button type="button" class="col" @click="cancelItem()" >取消</button>
                        <button type="button" class="col" @click="confirmItem()" v-if="editorTitle === 'Adding Area'" :disabled="!validateInput()">確認新增</button>
                        <button type="button" class="col" @click="updateItem()" v-if="editorTitle === 'Editing Area'" :disabled="!validateInput()">更新</button>
                        <button type="button" class="col" @click="cloneItem()" v-if="editorTitle === 'Editing Area'" :disabled="!validateInput()">複製</button>
                    </td>
                    </tr>
                    <tr style="color:red">
                    <td width="66%">{{ !validateInput() ? errorText1 : "" }}</td>
                    <td width="20%"></td>
                    </tr>
                </tbody>
                </table>
                <hr v-if="editorTitle"/>

                <table>
                    <thead>
                        <tr>
                        <th scope="col" v-for="header in headers" :key="header.id"  style="text-align:left">{{ header.value }}</th>
                        </tr>
                    </thead>
                    <tbody v-for="(item, index) in todoLists" :key="item.id">
                        <tr v-bind:style="{ backgroundColor: item.editing ? '#4FC1F1' : index % 2 === 0 ? '#D0E0EF' : '#FFFFFF' }">
                        <td width="40%">{{ item.content }}</td>
                        <td width="10%" v-show="item.status"><small><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-check-circle-fill" viewBox="0 0 16 16">
                          <path d="M16 8A8 8 0 1 1 0 8a8 8 0 0 1 16 0m-3.97-3.03a.75.75 0 0 0-1.08.022L7.477 9.417 5.384 7.323a.75.75 0 0 0-1.06 1.06L6.97 11.03a.75.75 0 0 0 1.079-.02l3.992-4.99a.75.75 0 0 0-.01-1.05z"/>
                        </svg> 已完成</small></td>
                        <td width="10%" v-show="!item.status"><small><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-x-circle" viewBox="0 0 16 16">
                          <path d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14m0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16"/>
                          <path d="M4.646 4.646a.5.5 0 0 1 .708 0L8 7.293l2.646-2.647a.5.5 0 0 1 .708.708L8.707 8l2.647 2.646a.5.5 0 0 1-.708.708L8 8.707l-2.646 2.647a.5.5 0 0 1-.708-.708L7.293 8 4.646 5.354a.5.5 0 0 1 0-.708"/>
                        </svg> 未完成</small></td>
                        <td width="30%" style="text-align: right;">
                            <button type="button" class="col" @click="editItem(item)" :disabled="editItems.length > 0 || deleteId !==''" v-bind:style="{ display: deleteId === item.id ? 'none' : 'inline-block' }">編輯待辦事項</button>
                            <button type="button" class="col" @click="showDoubleConfirm(item)" :disabled="editItems.length > 0 || deleteId !==''" v-bind:style="{ display: deleteId === item.id ? 'none' : 'inline-block' }">刪除</button>
                            <button type="button" class="col" @click="toggleTodo(item)" :disabled="editItems.length > 0 || deleteId !==''" v-bind:style="{ display: deleteId === item.id ? 'none' : 'inline-block' }">改狀態</button>
                            <button type="button" class="col" @click="cancelDelete(item)" :disabled="editItems.length > 0" v-if="deleteId === item.id">取消刪除</button>
                            <button type="button" class="col" @click="deleteItem(item)" :disabled="editItems.length > 0" v-if="deleteId === item.id">確認刪除</button>
                        </td>
                        </tr>
                    </tbody>
                    </table>
                    <hr />
                    <button type="button" class="col" @click="createItem()" :disabled="editItems.length > 0 || deleteId !==''">新增待辦事項</button>
            </div>
        </div>
    </div>
    <div class="toast-container position-fixed top-0 end-0 p-3">
      <div v-for="(toast, index) in toasts" :key="index" class="toast show" role="alert" aria-live="assertive" aria-atomic="true">
        <div class="toast-body">
          {{ toast.message }}
        </div>
      </div>
    </div>
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted } from 'vue';

const api = "https://todolist-api.hexschool.io";

const signUpData = ref({
    email: "",
    password: "",
    nickname: ""
});

const signInData = ref({
    email: "",
    password: ""
});



const user = ref({
    uid: "",
    nickname: "",
});


let tablValue = ref(2); // 預設顯示登入
const signUpStatus = ref(false);
let signUpResponse = ref('');
let signInResponse = ref('');
let checkoutResponse = ref('');
let signOutResponse = ref('');
const token = ref('');
let loginStatus = ref(false);
let cloneStatus = ref(false);
const toasts = ref([]);


const tabClick = (num) => {
    tablValue.value = num;
    signUpResponse.value = "";
    signInResponse.value = "";
    checkoutResponse.value = "";
}


const signUp = async() => {
    try{
        const response = await axios.post(`${api}/users/sign_up`,
            signUpData.value // 記得加上.value
        );
        
        // signUpResponse.value = response.data.uid;
        signUpResponse.value = '註冊成功! 請使用' + signUpData.value.email + '來登入系統';
        setInterval(() =>{
            tabClick(2)
        },2000)
        // 註冊成功
        signUpStatus.value = true;
        // 把信箱寫入登入input
        signInData.value.email = signUpData.value.email;
        // 清空輸入值        
        signUpData.value = {
            email: "",
            password: "",
            nickname: ""
        };
    } catch(err){
        signUpResponse.value = err.response.data.message;
        // 清空輸入值
        signUpData.value = {
            email: "",
            password: "",
            nickname: ""
        };
    };
}

const signIn = async() => {
    try{
        const response = await axios.post(`${api}/users/sign_in`,
            signInData.value // 記得加上.value
        );
        
        signInResponse.value = response.data.token;
        // 寫入cookie
        document.cookie = `myToken=${response.data.token};  path=/`;
        // 重整網頁
        location.reload(); 
    } catch(err){
        signInResponse.value = err.response.data.message;
    };
}

const checkOut = async () =>{
  // 讀出 cookie
  token.value = document.cookie.replace(
        /(?:(?:^|.*;\s*)myToken\s*=\s*([^;]*).*$)|^.*$/,
        "$1",
    );
    try{
        const response = await axios.get(`${api}/users/checkout`,{
            headers: {
                Authorization: `${token.value}`,
            }
        }
        );
        
        checkoutResponse.value = response.data;
        user.value = response.data;
        loginStatus.value = response.data?.status
        if(loginStatus.value){
            getTodoList()
        }
       
    } catch(err){
        checkoutResponse.value = err.response.data.message;
        loginStatus.value = err.response.data?.status
    };
}

onMounted(() => {
    token.value = document.cookie.replace(
        /(?:(?:^|.*;\s*)myToken\s*=\s*([^;]*).*$)|^.*$/,
        "$1",
    );

    if(token.value != "") {
      loginStatus.value = true
      checkOut()
    }
})



const signOut= async() => {
    try{
        const response = await axios.post(`${api}/users/sign_out`,
            {}, //不需要傳參數
            {
                headers: {
                    Authorization: `${token.value}`,
                }
            }
        );
        
        signOutResponse.value = response.data.message;
        document.cookie = `myToken=; max-age=0; path=/;`
        // 重整網頁
        location.reload();
        tablValue.value = 2 
    } catch(err){
        signOutResponse.value = err.response.data.message;
    };
}

const todoLists = ref([])

const toastPopup = (message) => {
  toasts.value.push(
    {
      message: message,
    }
  );

  setTimeout(() => {
    toasts.value.shift()
  }, 3000)
}

const clearEditor = () => {
  editItems.value = [];
  clearText()
}

const getTodoList = async() => {
    try{
        const response = await axios.get(`${api}/todos`,{
            headers: {
                Authorization: `${token.value}`,
            }
        }
        );
        
        todoLists.value = response.data.data.length > 0 && response.data.data;
        todoLists.value = response.data.data.map(v=>({
               id: v.id,
               createTime: v.createTime,
               content: v.content,
               status: v.status,
               editing: false,
            })
        )
    } catch(err){
        toastPopup(err.response.data.message)
    };
}


const createTodo = async() => {
    try{
        let detail;
        if(cloneStatus.value){
          const index = todoLists.value.findIndex(i => i.editing === true);
          detail = todoLists.value[index].content
        }else{
          detail = editItems.value[0].content
        }  
      
        const response = await axios.post(`${api}/todos`,
            {
                "content": detail
            },
            {
                headers: {
                    Authorization: `${token.value}`,
                }
            }           
        );
        let message
        if(response.data.status == true){
          // 新增成功
          message= `${cloneStatus ? '複製' : '新增'}待辦事項成功: ${detail}`
          // 因為response的item無提供id, 所以需要call getTodoList() 取得新清單資訊
          clearEditor();
          getTodoList();
          cloneStatus = false;
          // {
          //   "status": true,
          //   "newTodo": {
          //     "createTime": 1723710684,
          //     "content": "買晚餐333",
          //     "status": false
          //   }
          // }
           
          // let item = editItems.value[0];
          // todoLists.value.push(item);
          
        }else{
          // 新增失敗
          message = '新增待辦事項失敗,請稍後再試'
        }
        toastPopup(message);   
    } catch(err){
        // 新增失敗
        let message = `新增待辦事項失敗, ${err.response.data.message}`;
        toastPopup(message);
    };
}

const updateTodo = async(item) => {
    let id = item.id
    // 更新用 axios.put
    try{
        const response = await axios.put(`${api}/todos/${id}`,
            {
                "content": editItems.value[0].content
            },
            {
                headers: {
                    Authorization: `${token.value}`,
                }
            }           
        );
        let message
        if(response.data.status == true){
            // 更新成功
            message= '更新待辦事項成功';
            const index = todoLists.value.findIndex(i => i.id === item.id);
            todoLists.value[index] = item;
            clearEditor();
        }else{
            // 更新失敗
            message = '更新待辦事項失敗,請稍後再試:'+ response.data.message;
        }
        toastPopup(message);
    } catch(err){
        // 更新失敗
        message = `更新待辦事項失敗, ${err.response.data.message}`;
        toastPopup(message);
    };
}

const deleteTodo = async(item) => {
    let id = item.id
    // 刪除用 axios.delete
    try{
        const response = await axios.delete(`${api}/todos/${id}`,
            {
                headers: {
                    Authorization: `${token.value}`,
                }
            }           
        );
        let message
        if(response.data.status == true){
            //刪除成功
            message= '刪除待辦事項成功';
            const index = todoLists.value.findIndex(i => i.id === item.id);
            todoLists.value.splice(index, 1);
            deleteId = "";
            clearText()
        }else{
            //刪除失敗
            message= '刪除待辦事項失敗,請稍後再試:'+ response.data.message
        }
        toastPopup(message);    
    } catch(err){
        // 刪除失敗
        message = `刪除待辦事項失敗, ${err.response.data.message}`;
        toastPopup(message);
    };
}

const toggleTodo = async(item) => {
    let id = item.id
    // 更新狀態用 axios.patch
    try{
        const response = await axios.patch(`${api}/todos/${id}/toggle`,
            {}, //不需要傳參數    
            {
                headers: {
                    Authorization: `${token.value}`,
                }
            }           
        );
        let message
        if(response.data.status == true){
            //更新狀態成功
            message= '待辦事項更新狀態成功';
            const index = todoLists.value.findIndex(i => i.id === item.id);
            todoLists.value[index].status = !todoLists.value[index].status;
            clearText()
        }else{
            //更新狀態失敗
            message= '待辦事項更新狀態失敗,請稍後再試:'+ response.data.message
        }
        toastPopup(message);    
    } catch(err){
        // 更新狀態失敗
        message = `待辦事項更新狀態失敗, ${err.response.data.message}`;
        toastPopup(message);
    };
}

//GET請求
// axios.get('https://randomuser.me/api/')
//     .then( (response) => console.log(response))
//     .catch( (error) => console.log(error))

// //POST請求
// axios.post('https://todolist-api.hexschool.io/users/sign_up/',{
//     "email": "example@gmail.com",
//     "password": "example",
//     "nickname": "example"
// })
//     .then( (response) => console.log(response))
//     .catch( (error) => console.log(error))


let editorTitle = ref('')
let deleteId = ref('')
let errorText1 = ref('')
let errorText2 = ref('')
let errorText3 = ref('')
let errorText4 = ref('')

// Header
const headers = ref([
  {
    id: 1,
    value:'待辦事項'
  },
  {
    id: 2,
    value:'狀態'
  }
])

const editItems = ref([])

// Create btn
const createItem = () => {
  editorTitle = "Adding Area";
  editItems.value.push({
      id: new Date().getTime(),
      createTime: new Date().getTime(),
      content: "",
      status: false,
      editing: false,
  })
}

// Edit btn
const editItem = (item) => {
  editorTitle = "Editing Area";
  if(editItems.value.length > 0){
    toastPopup("已有待辦事項在編輯!");
    return
  }else{
    const index = todoLists.value.findIndex(i => i.id === item.id);
    todoLists.value[index].editing = true;
    editItems.value.push({
      id: item.id,
      createTime: item.createTime,
      content: item.content,
      status: item.status,
      editing: false,
    })
  }
}

// Delete btn
const deleteItem= (item) => {
  const index = todoLists.value.findIndex(i => i.id === item.id);
  if(index !== -1){
    deleteTodo(item)
  }
}

const clearText = () => {
    editorTitle = "";
    errorText1 = ""
    errorText2 = ""
    errorText3 = ""
    errorText4 = ""
}

const validateInput = () => {
  let item = editItems.value[0];
  errorText1 = item.content === "" ? "此欄位不可為空" : "";

  if( errorText1 !==""){
    return false
  }else{
    return true
  }
}

// Update btn
const updateItem= () => {
  let item = editItems.value[0];
  if(!validateInput()){
    toastPopup("有錯誤無法更新!");
    return
  }else{
    updateTodo(item)
  }
}

// Cancel btn
const cancelItem = () => {
  if(editorTitle === "Adding Area"){
    editItems.value = [];
    clearText()
  }else{
    console.log("editItems.value.length:",editItems.value.length)
    if(editItems.value.length === 0){
      toastPopup("已無暫存資料可取消!");
      return
    }else{
      const index = todoLists.value.findIndex(i => i.editing === true)
      todoLists.value[index].editing = false;
      // clear edit item
      editItems.value = [];
    }
    clearText()
  }
}

// clone btn
const cloneItem= () => {
  // let item = editItems.value[0];
  // editItems.value[0].id = new Date().getTime();
  // const index = todoLists.value.findIndex(i => i.editing === true);
  // todoLists.value[index].editing = false;
  // copy item to items
  // todoLists.value.push(item);
  // clear edit item
  cloneStatus = true
  createTodo();
  getTodoList();
  editItems.value = [];
  clearText()
}

// Confirm btn
const confirmItem = () => {
  if(!validateInput()){
    toastPopup("有錯誤無法更新")
    return
  }else{
    createTodo()
  }  
}

const cancelDelete = (item) => {
  const index = todoLists.value.findIndex(i => i.id === item.id);
  todoLists.value[index].editing = false;
  deleteId = "";
}

const showDoubleConfirm = (item) => {
  const index = todoLists.value.findIndex(i => i.id === item.id);
  todoLists.value[index].editing = true;
  deleteId = todoLists.value[index].id;
}

</script>

<style>
.loginInfo {
    display: inline-block;
    padding-right: 10;
}
.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 16px;
}

.row {
  display: flex;
  flex-wrap: wrap;
  margin: -8px;
}

.col {
  flex: 0 0 50%;
  padding: 8px;
  box-sizing: border-box;
}

.btn {
  display: inline-block;
  padding: 8px 16px;
  background-color: #007bff;
  color: #fff;
  text-align: center;
  border-radius: 4px;
  text-decoration: none;
}

.btn2 {
  display: inline-block;
  padding: 8px 16px;
  background-color: #007bff;
  color: #fff;
  text-align: center;
  border-radius: 4px;
  text-decoration: none;
  border:0;
  font-size: 1rem;
}

.btn:hover, .btn2:hover {
  background-color: #0868cf;
  cursor: pointer;
}

header {
  background-color: aliceblue;
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 15px;
  padding: 5px 10px;
}

header ul {
  display: flex;
}

header li {
  list-style: none;
  padding: 0 15px;
}
</style>