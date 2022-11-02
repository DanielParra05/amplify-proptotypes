<script setup lang="ts">
import { ref } from '@vue/reactivity';
import { API } from 'aws-amplify';
import { onMounted } from 'vue';
import { createTodo } from './graphql/mutations';
import { listTodos } from './graphql/queries';
import { onCreateTodo } from './graphql/subscriptions';

const name = ref('');
const description = ref('');
const todos = ref<any[]>([]);

async function insertTodo() {
  if (!name.value || !description.value) return;
  const todo = { name, description };
  await API.graphql({
    query: createTodo,
    variables: { input: todo }
  });
}

async function getTodos() {
  const todos = await API.graphql({
    query: listTodos
  });
  todos.value = todos.data.listTodos.items;
}

function subscribe() {
  API.graphql({ query: onCreateTodo }).subscribe({
    next: (eventData: any) => {
      let todo = eventData.value.data.onCreateTodo;
      if (todos.value.some((item) => item.name === todo.name)) return; // remove duplications
      todos.value = [...todos.value, todo];
    }
  });
}

onMounted(() => {
  getTodos();
  subscribe();
});
</script>

<template>
  <div id="app">
    <h1>Todo App</h1>
    <input type="text" v-model="name" placeholder="Todo name" />
    <input type="text" v-model="description" placeholder="Todo description" />
    <button v-on:click="insertTodo">Create Todo</button>
  </div>
  <div v-for="item in todos" :key="item.id">
    <h3>{{ item.name }}</h3>
    <p>{{ item.description }}</p>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
