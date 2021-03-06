<template>
  <div>
    <Navbar 
      :page="currentPage" 
      @addTask="changePage" 
      @logout="changePage">
    </Navbar>
    <LoginPage
      v-if="currentPage == 'Login Page'"
      @registerPage="changePage"
      @mainPage="changePage"
      @loginData="login"
      @loginGoogle="loginGoogle"
    >
    </LoginPage>
    <RegisterPage
      v-if="currentPage === 'Register Page'"
      @loginPage="changePage"
      @registerData="register"
    >
    </RegisterPage>
    <MainBoard 
    v-if="currentPage === 'Main Page'"
    :categories="categories"
    :tasks="tasks"
    @getEdit="getEdit"
    @deleteTask="deleteTask"
    @moveLeft="updateStatus"
    @moveRight="updateStatus"
    >
    </MainBoard>
    <AddTaskPage
      v-if="currentPage === 'Add Task Page'"
      @mainPage="changePage"
      @addTaskData="add"
    ></AddTaskPage>
    <EditTaskPage 
    v-if="currentPage === 'Edit Task Page'"
    :task="task"
    @editTaskData="editPost"
    @mainPage="changePage"
    ></EditTaskPage>
  </div>
</template>

<script>
import axios from "axios";
import Navbar from "./components/Navbar";
import RegisterPage from "./components/RegisterPage";
import LoginPage from "./components/LoginPage";
import MainBoard from "./components/MainBoard";
import AddTaskPage from "./components/AddTaskPage";
import EditTaskPage from "./components/EditTaskPage";

export default {
  name: "App",
  data() {
    return {
      currentPage: "Register Page",
      categories: [
        {
          name: 'backlog',
          color: 'item-1',
          label: 'Backlog'
        },
        {
          name: 'todo',
          color: 'item-2',
          label: 'Todo'
        },
        {
          name: 'doing',
          color: 'item-3',
          label: 'Doing'
        },
        {
          name: 'done',
          color: 'item-4',
          label: 'Done'
        },
      ],
      tasks: [],
      task: {}
    };
  },
  methods: {
    changePage(page) {
      this.currentPage = page;
      if(page == 'Main Page'){
        this.fetchTask()
      }
    },

    login(data) {
      let email = data.email;
      let password = data.password;

      axios({
        method: "post",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/login",
        data: {
          email,
          password,
        },
      })
        .then((response) => {
          localStorage.setItem("access_token", response.data.access_token);
          Swal.fire({
            icon: "success",
            title: "Login Success!",
            showConfirmButton: false,
            timer: 1500,
          });
          this.changePage('Main Page')
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Login Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
        .finally((_) => {
          this.email = "";
          this.password = "";
        });
    },

    loginGoogle(googleToken) {
      axios({
        method: "post",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/googlelogin",
        data: {
          googleToken,
        },
      })
        .then((response) => {
          localStorage.setItem("access_token", response.data.access_token);
          Swal.fire({
            icon: "success",
            title: "Login Success!",
            showConfirmButton: false,
            timer: 1500,
          });
          this.changePage('Main Page')
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Login Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
        .finally((_) => {
          this.email = "";
          this.password = "";
        });
    },

    register(data) {
      let name = data.name;
      let email = data.email;
      let password = data.password;
      axios({
        method: "post",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/register",
        data: {
          name,
          email,
          password,
        },
      })
        .then((response) => {
          Swal.fire({
            icon: "success",
            title: "Registration Success!",
            showConfirmButton: false,
            timer: 1500,
          });
          this.changePage('Login Page')
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Registration Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
        .finally((_) => {
          this.email = "";
          this.password = "";
        });
    },

    add(data) {
      let title = data.title;
      let description = data.description;
      let point = data.point;
      let assignedto = data.assignedto;
      let status = data.status;
      axios({
        method: "post",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/tasks",
        headers: {
          access_token: localStorage.getItem("access_token"),
        },
        data: {
          title,
          description,
          point,
          assignedto,
          status,
        },
      })
        .then((response) => {
          Swal.fire({
            icon: "success",
            title: "Add Task Success!",
            showConfirmButton: false,
            timer: 1500,
          });
          this.changePage('Main Page')
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Add Task Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
        .finally((_) => {
          this.title = "";
          this.description = "";
          this.point = "";
          this.assignedto = "";
          this.status = "";
        });
    },

    getEdit(id){
      axios({
        method: "get",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/tasks/" + id,
        headers: {
          access_token: localStorage.getItem("access_token"),
        },
        data: {
          id
        },
      })
        .then((response) => {
          this.task = response.data.task
          this.changePage('Edit Task Page')
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Get Task Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
    },

    editPost(data) {
      let id = data.id;
      let title = data.title;
      let description = data.description;
      let point = data.point;
      let assignedto = data.assignedto;
      let status = data.status;
      axios({
        method: "put",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/tasks/" + id,
        headers: {
          access_token: localStorage.getItem("access_token"),
        },
        data: {
          title,
          description,
          point,
          assignedto,
          status,
        },
      })
        .then((response) => {
          Swal.fire({
            icon: "success",
            title: "Edit Task Success!",
            showConfirmButton: false,
            timer: 1500,
          });
          this.changePage('Main Page')
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Edit Task Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
        .finally((_) => {
          this.title = "";
          this.description = "";
          this.point = "";
          this.assignedto = "";
          this.status = "";
        });
    },

    updateStatus(id, status, move) {
      let updateStatus 
      for (let i = 0; i < this.categories.length; i++) {
        if(status == this.categories[i].name){
          if(move == 'prev'){
            updateStatus = this.categories[i-1].name
          } else if(move == 'next'){
            updateStatus = this.categories[i+1].name
          }
        }
      }
      axios({
        method: "patch",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/tasks/" + id,
        headers: {
          access_token: localStorage.getItem("access_token"),
        },
        data: {
          status: updateStatus
        },
      })
        .then((response) => {
          Swal.fire({
            icon: "success",
            title: "Move Task Success!",
            text: "Task moved to " + updateStatus,
            showConfirmButton: false,
            timer: 1500,
          });
          this.changePage('Main Page')
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Move Task Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
        .finally((_) => {
          this.title = "";
          this.description = "";
          this.point = "";
          this.assignedto = "";
          this.status = "";
        });
    },

    deleteTask(id){
      axios({
        method: "delete",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/tasks/" + id,
        headers: {
          access_token: localStorage.getItem("access_token"),
        },
        data: {
          id
        },
      })
        .then((response) => {
          Swal.fire({
            icon: "success",
            title: "Delete Task Success!",
            text: response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
          this.changePage('Main Page')
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Delete Task Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
    },

    fetchTask(){
      axios({
        method: "get",
        url: "https://kanban-taufiq-ismail-server.herokuapp.com/tasks",
        headers: {
          access_token: localStorage.getItem("access_token"),
        }
      })
        .then((response) => {
          this.tasks = response.data.task;
        })
        .catch((error) => {
          Swal.fire({
            icon: "error",
            title: "Show Task Failed!",
            text: error.response.data.message,
            showConfirmButton: false,
            timer: 1500,
          });
        })
    }
  },
  created: function () {
    if (localStorage.getItem("access_token")) {
      this.changePage('Main Page')
    } else {
      this.changePage('Login Page')
    }
  },
  components: {
    Navbar,
    RegisterPage,
    LoginPage,
    MainBoard,
    AddTaskPage,
    EditTaskPage,
  },
};
</script>