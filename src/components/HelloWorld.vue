<script setup>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get('http://localhost:3000/articles');
    articles.value = response.data;
  } catch (error) {
    console.error('Error loading articles:', error);
  }
}

async function save() {
  try {
    const url = form.id
      ? `http://localhost:3000/articles/${form.id}`
      : 'http://localhost:3000/articles';
    const method = form.id ? 'put' : 'post';
    const response = await axios[method](url, form);

    articles.value = form.id
      ? articles.value.map((article) =>
          article.id === response.data.id ? response.data : article
        )
      : [...articles.value, response.data];

    form.id = null;
    form.title = '';
    form.content = '';
  } catch (error) {
    console.error('Error saving article:', error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

onMounted(load);
</script>

<template>
  <div>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" /><br/>
      <textarea v-model="form.content"></textarea>
      <button type="submit">Save</button>
    </form>
    <ul>
      <li v-for="article in articles" :key="article.id">
        {{ article.title }}<br />
        {{ article.content }}<br />
        <button @click="edit(article)">Edit</button>
      </li>
    </ul>
    <button @click="load">Load Articles</button>
  </div>
</template>
