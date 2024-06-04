<template>
  <div class="container">
    <h1 class="title">Data Hewan Qurban</h1>
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br>
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br>
      <button type="submit" class="button">Save</button>
    </form>
    <ul class="articles">
      <li v-for="article in articles" :key="article.id" class="article">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <div class="buttons">
          <button @click="edit(article)" class="button button-icon">
            <i class="fas fa-edit"></i> Edit
          </button>
          <button @click="confirmDelete(article.id)" class="button button-danger button-icon">
            <i class="fas fa-trash"></i> Delete
          </button>
        </div>
      </li>
    </ul>
    <button @click="load" class="button">Load</button>

    <!-- Delete Confirmation Modal -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <p>Are you sure you want to delete this article?</p>
        <button @click="deleteArticle" class="button button-danger">Yes, Delete</button>
        <button @click="closeModal" class="button">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });

    const articles = ref([]);
    const showModal = ref(false);
    const articleToDelete = ref(null);

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
        const url = form.id ? `http://localhost:3000/articles/${form.id}` : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (method === 'put') {
          const index = articles.value.findIndex(article => article.id === response.data.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
        }

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

    function confirmDelete(id) {
      articleToDelete.value = id;
      showModal.value = true;
    }

    function deleteArticle() {
      remove(articleToDelete.value);
      closeModal();
    }

    function closeModal() {
      showModal.value = false;
      articleToDelete.value = null;
    }

    onMounted(load);

    return { form, articles, save, edit, confirmDelete, deleteArticle, closeModal, showModal };
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Roboto', sans-serif;
}

.title {
  font-size: 2em;
  text-align: center;
  margin-bottom: 20px;
  font-weight: 700;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.input, .textarea {
  padding: 10px;
  font-size: 1em;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-family: 'Roboto', sans-serif;
}

.button {
  padding: 10px 20px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  font-family: 'Roboto', sans-serif;
}

.button:hover {
  background-color: #0056b3;
}

.button-danger {
  background-color: #dc3545;
}

.button-danger:hover {
  background-color: #c82333;
}

.articles {
  list-style: none;
  padding: 0;
}

.article {
  background-color: #000000;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 5px;
  margin-bottom: 10px;
}

.buttons {
  display: flex;
  gap: 10px;
}

.button-icon i {
  margin-right: 5px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}

.modal-content p {
  margin-bottom: 20px;
}
</style>
