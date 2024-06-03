<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button save-button">{{ form.id ? 'Update' : 'Save' }}</button>
    </form>
    <ul class="articles-list">
      <li v-for="article in articles" :key="article.id" class="article-item" :class="{ 'new-article': article.id === newArticleId }">
        <div class="article-content">
          <strong>{{ article.title }}</strong><br />
          {{ article.content }}<br />
          <button @click="edit(article)" class="button edit-button">Edit</button>
          <button @click="remove(article.id)" class="button delete-button">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });

    const articles = ref([]);
    const newArticleId = ref(null);

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
        const response = await axios[method](url, { title: form.title, content: form.content });

        if (form.id) {
          const index = articles.value.findIndex(article => article.id === form.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
          newArticleId.value = response.data.id;
          setTimeout(() => {
            newArticleId.value = null;
          }, 2000);
        }

        // Reset the form after saving
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load, newArticleId };
  }
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.form {
  margin-bottom: 20px;
}

.input, .textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.save-button {
  background-color: #28a745;
  color: white;
}

.load-button {
  background-color: #007bff;
  color: white;
  margin-top: 20px;
}

.edit-button {
  background-color: #ffc107;
  color: white;
}

.delete-button {
  background-color: #dc3545;
  color: white;
}

.articles-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
  transition: transform 0.5s ease;
}

.article-content {
  background-color: #ffffff;
  padding: 15px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.new-article {
  transform: translateX(100%);
}
</style>
