
<template>
  <div>
    <h1>Todo List</h1>
    <!-- Todo input form -->
    <form @submit.prevent="addTodo">
      <el-input placeholder="Todo" v-model="todo"></el-input>
    </form>
    <el-row :gutter="12">
      <!-- Todo display area -->
      <el-col :span="12" v-for="(todo, index) in todos" :key="'todo-' + index">
        <TodoItem :text="todo" @finish="removeTodo(index)" />
      </el-col>
      <!-- Issue display area -->
      <el-col :span="12" v-for="(issue, index) in issues" :key="'issue-' + index">
        <el-card class="box-card" shadow="hover" style="margin: 5px 0;">
          <el-row :gutter="12">
            <el-col :span="21">{{ issue.title }}</el-col>
            <el-col :span="3">
              <el-button @click="closeIssue(index)" type="success" icon="el-icon-check" circle></el-button>
            </el-col>
          </el-row>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import TodoItem from './TodoItem.vue';
import axios from 'axios';

const client = axios.create({
  baseURL: `${process.env.VUE_APP_GITHUB_ENDPOINT}`,
  headers: {
    'Authorization': `token ${process.env.VUE_APP_GITHUB_TOKEN}`,
    'Accept': 'application/vnd.github.v3+json',
    'Content-Type': 'application/json',
  },
});

export default {
  name: 'TodoListComponent',
  components: {
    TodoItem,
  },
  data() {
    return {
      todo: '',
      todos: [],
      issues: [],
    };
  },
  methods: {
    addTodo() {
      this.todos.push(this.todo);
      this.todo = '';
    },
    removeTodo(index) {
      this.todos.splice(index, 1);
    },
    closeIssue(index) {
      const target = this.issues[index];
      client
        .patch(`/issues/${target.number}`, {
          state: 'closed',
        })
        .then(() => {
          this.issues.splice(index, 1);
        })
        .catch((error) => {
          console.error(error);
        });
    },
    getIssues() {
      client
        .get('issues')
        .then((res) => {
          this.issues = res.data;
        })
        .catch((error) => {
          console.error(error);
        });
    },
  },
  created() {
    this.getIssues();
  },
};
</script>