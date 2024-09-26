<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const API_URL = 'https://myapi.berichtravel.com/api';
const API_KEY = '13|lJDAbnQKrEClAQsjvCiVcylsDA7A5EpfoOojBrop0e241775';

const items = ref([]);
const newItem = ref('');
const editingItem = ref(null);

const fetchItems = async () => {
  try {
    const response = await axios.get(`${API_URL}/post`, {
      headers: { 'Authorization': `Bearer ${API_KEY}` }
    });
    items.value = response.data;
  } catch (error) {
    console.error('Error fetching items:', error);
  }
};

const addItem = async () => {
  if (newItem.value.trim()) {
    try {
      const response = await axios.post(`${API_URL}/post`, { name: newItem.value }, {
        headers: { 'Authorization': `Bearer ${API_KEY}` }
      });
      items.value.push(response.data);
      newItem.value = '';
    } catch (error) {
      console.error('Error adding item:', error);
    }
  }
};

const updateItem = async (item) => {
  try {
    await axios.put(`${API_URL}/post/${item.id}`, item, {
      headers: { 'Authorization': `Bearer ${API_KEY}` }
    });
    const index = items.value.findIndex(i => i.post_id === item.post_id);
    if (index !== -1) {
      items.value[index] = { ...item };
    }
    editingItem.value = null;
  } catch (error) {
    console.error('Error updating item:', error);
  }
};

const deleteItem = async (id) => {
  try {
    await axios.delete(`${API_URL}/post/${id}`, {
      headers: { 'Authorization': `Bearer ${API_KEY}` }
    });
    items.value = items.value.filter(item => item.post_id !== post_id);
  } catch (error) {
    console.error('Error deleting item:', error);
  }
};

onMounted(fetchItems);
</script>

<template>
  <div class="container mt-5">
    <h1 class="mb-4">CRUD Operations</h1>
    
    <div class="mb-3">
      <input v-model="newItem" @keyup.enter="addItem" class="form-control" placeholder="Add new item">
      <button @click="addItem" class="btn btn-primary mt-2">Add Item</button>
    </div>
    
    <ul class="list-group">
      <li v-for="item in items" :key="item.post_id" class="list-group-item d-flex justify-content-between align-items-center">
        <div v-if="editingItem === item.post_id">
          <input v-model="item.name" @keyup.enter="updateItem(item)" class="form-control">
        </div>
        <span v-else>{{ item.post_title }}</span>
        <div>
          <button v-if="editingItem === item.post_id" @click="updateItem(item)" class="btn btn-success btn-sm me-2">Save</button>
          <button v-else @click="editingItem = item.post_id" class="btn btn-warning btn-sm me-2">Edit</button>
          <button @click="deleteItem(item.post_id)" class="btn btn-danger btn-sm">Delete</button>
        </div>
      </li>
    </ul>
  </div>
</template>

<style scoped>
/* You can add any additional styles here if needed */
</style>
