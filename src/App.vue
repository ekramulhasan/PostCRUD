<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const API_URL = "https://myapi.berichtravel.com/api";
const API_KEY = "13|lJDAbnQKrEClAQsjvCiVcylsDA7A5EpfoOojBrop0e241775";

const items = ref([]);
const newItem = ref({ user_id: "", title: "", body_text: "" });
const editingItem = ref(null);



const fetchItems = async () => {
  try {
    const response = await axios.get(`${API_URL}/post`, {
      headers: { Authorization: `Bearer ${API_KEY}` },
    });
    items.value = response.data;
  } catch (error) {
    console.error("Error fetching items:", error);
  }
};

const addItem = async () => {
  if (
    newItem.value.title.trim() &&
    newItem.value.body_text.trim() &&
    newItem.value.user_id.trim()
  ) {
    try {
      const response = await axios.post(`${API_URL}/post`, newItem.value, {
        headers: { Authorization: `Bearer ${API_KEY}` },
      });
      items.value.push(response.data);
      newItem.value = { user_id: "", title: "", body_text: "" };
    } catch (error) {
      console.error("Error adding item:", error);
    }
  }
};

const updateItem = async (item) => {
  try {
    const { post_id, post_title, post_text } = item;
    const response = await axios.put(
      `${API_URL}/post/${post_id}`,
      { title: post_title, body_text: post_text },
      {
        headers: { Authorization: `Bearer ${API_KEY}` },
      }
    );
    const index = items.value.findIndex((i) => i.post_id === post_id);
    if (index !== -1) {
      items.value[index] = { ...response.data };
    }
    editingItem.value = null;
    await fetchItems(); // Refresh the list after update
  } catch (error) {
    console.error("Error updating item:", error);
  }
};

const deleteItem = async (id) => {
  try {
    await axios.delete(`${API_URL}/post/${id}`, {
      headers: { Authorization: `Bearer ${API_KEY}` },
    });
    items.value = items.value.filter((item) => item.post_id !== id);
  } catch (error) {
    console.error("Error deleting item:", error);
  }
};

onMounted(fetchItems);
</script>

<template>
  <div class="container mt-5">
    <h1 class="mb-4">CRUD Operations</h1>

    <div class="mb-3">
      <input v-model="newItem.user_id" class="form-control mb-2" placeholder="User ID" />
      <input v-model="newItem.title" class="form-control mb-2" placeholder="Title" />
      <textarea
        v-model="newItem.body_text"
        class="form-control mb-2"
        placeholder="Body Text"
      ></textarea>
      <button @click="addItem" class="btn btn-primary mt-2">Add Item</button>
    </div>

    <ul class="list-group">
      <li
        v-for="item in items"
        :key="item.post_id"
        class="list-group-item d-flex justify-content-between align-items-center"
      >
        <div v-if="editingItem === item.post_id">
          <input v-model="item.post_title" class="form-control mb-2" placeholder="Title" />
          <textarea
            v-model="item.post_text"
            class="form-control mb-2"
            placeholder="Body Text"
          ></textarea>
        </div>
        <div v-else>
          <h5>{{ item.post_title }}</h5>
          <p>{{ item.post_text }}</p>
        </div>
        <div>
          <button
            v-if="editingItem === item.post_id"
            @click="updateItem(item)"
            class="btn btn-success btn-sm me-2"
          >
            Save
          </button>
          <button
            v-else
            @click="editingItem = item.post_id"
            class="btn btn-warning btn-sm me-2"
          >
            Edit
          </button>
          <button @click="deleteItem(item.post_id)" class="btn btn-danger btn-sm">
            Delete
          </button>
        </div>
      </li>
    </ul>
  </div>
</template>

<style scoped>
/* You can add any additional styles here if needed */
</style>
