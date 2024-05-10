<template>
  <div id="app">
    <header>
      <button @click="showPostSection">Post</button>
      <button @click="showTodosSection">Todos</button>
    </header>
    <div>
      <h1>Dashboard</h1>
      <div v-if="currentSection === 'post'">
        <h2>Postingan</h2>
        <select v-model="selectedUserId">
          <option value="" selected disabled>Pilih User</option>
          <option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}</option>
        </select>
        <ul>
          <li v-for="post in posts" :key="post.id">{{ post.title }}</li>
        </ul>
      </div>
      <div v-else-if="currentSection === 'todos'">
        <h2>Daftar Tugas</h2>
        <form @submit.prevent="addActivity">
          <input type="text" v-model="newActivity.name" placeholder="Tambahkan Tugas baru">
          <div class="datetime-container">
            <label for="datetime">Tanggal & Jam:</label>
            <input id="datetime" type="datetime-local" v-model="newActivity.dateTime">
          </div>
          <button type="submit">Tambah</button>
        </form>
        <div>
          <button @click="showAll">Semua Tugas</button>
          <button @click="showCompleted">Tugas Selesai</button>
          <button @click="showIncomplete">Tugas Belum Selesai</button>
        </div>
        <table>
          <thead>
            <tr>
              <th>Tugas</th>
              <th>Tanggal & Jam</th>
              <th>Status</th>
              <th>Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(activity, index) in filteredActivities" :key="index">
              <td>
                <span v-if="!activity.editMode">{{ activity.name }}</span>
                <input type="text" v-model="activity.name" v-else>
              </td>
              <td>
                <span v-if="!activity.editMode">{{ formatDate(activity.dateTime) }}</span>
                <input id="datetime" type="datetime-local" v-model="activity.dateTime" v-else>
              </td>
              <td>
                <input type="checkbox" v-model="activity.completed">
                <span :class="{ 'completed': activity.completed }">{{ activity.completed ? 'Selesai' : 'Belum Selesai' }}</span>
              </td>
              <td>
                <button @click="toggleEditMode(activity)">{{ activity.editMode ? 'Batal' : 'Edit' }}</button>
                <button v-if="activity.editMode" @click="updateActivity(activity)">Simpan</button>
                <button @click="removeActivity(index)">Hapus</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <footer>
      <p>&copy; Anggita Wahyudini Putri 223510804</p>
    </footer>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentSection: 'post',
      users: [],
      posts: [],
      selectedUserId: '',
      newActivity: {
        name: '',
        dateTime: ''
      },
      activities: [],
      filter: ''
    };
  },
  methods: {
    showPostSection() {
      this.currentSection = 'post';
      this.fetchUsers();
    },
    showTodosSection() {
      this.currentSection = 'todos';
    },
    fetchUsers() {
      fetch('https://jsonplaceholder.typicode.com/users')
        .then(response => {
          if (!response.ok) {
            throw new Error('Network response was not ok');
          }
          return response.json();
        })
        .then(users => {
          this.users = users;
        })
        .catch(error => console.error('Error fetching users:', error));
    },
    fetchPosts(userId) {
      fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`)
        .then(response => {
          if (!response.ok) {
            throw new Error('Network response was not ok');
          }
          return response.json();
        })
        .then(posts => {
          this.posts = posts;
        })
        .catch(error => console.error('Error fetching posts:', error));
    },
    addActivity() {
      if (this.newActivity.name.trim() !== '' && this.newActivity.dateTime.trim() !== '') {
        this.activities.unshift({ 
          name: this.newActivity.name, 
          dateTime: this.newActivity.dateTime, 
          completed: false,
          editMode: false,
        });
        this.newActivity.name = '';
        this.newActivity.dateTime = '';
      }
    },
    removeActivity(index) {
      this.activities.splice(index, 1);
    },
    formatDate(dateTime) {
      const options = { year: 'numeric', month: 'short', day: 'numeric', hour: 'numeric', minute: 'numeric' };
      return new Date(dateTime).toLocaleDateString('en-US', options);
    },
    toggleEditMode(activity) {
      activity.editMode = !activity.editMode;
    },
    updateActivity(activity) {
      activity.editMode = false;
    },
    showAll() {
      this.filter = '';
    },
    showCompleted() {
      this.filter = 'completed';
    },
    showIncomplete() {
      this.filter = 'incomplete';
    }
  },
  computed: {
    filteredActivities() {
      if (this.filter === 'completed') {
        return this.activities.filter(activity => activity.completed);
      } else if (this.filter === 'incomplete') {
        return this.activities.filter(activity => !activity.completed);
      } else {
        return this.activities;
      }
    }
  }
};
</script>

<style>
  body {
    font-family: Arial, sans-serif;
    background-color: pink;
    margin: 0;
    padding: 0;
  }

  header {
    background-color: #ff69b4;
    padding: 10px 0;
    text-align: center;
  }

  header button {
    background-color: transparent;
    border: none;
    cursor: pointer;
    font-size: 16px;
    padding: 10px 20px;
  }

  header button:hover {
    background-color: #f5f5f5;
  }

  /* Mengubah tampilan elemen lainnya menjadi berwarna pink */
  nav, ul, li {
    background-color: #ff69b4;
    padding: 0;
    margin: 0;
    list-style: none;
  }

  #app {
    max-width: 800px;
    margin: 20px auto;
    background-color: #fff;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  }

  h1 {
    color: #333;
    text-align: center;
  }

  input[type="text"], input[type="datetime-local"], button {
    margin-bottom: 10px;
    padding: 10px;
    font-size: 16px;
    border: none;
    border-radius: 5px;
  }

  input[type="text"], input[type="datetime-local"] {
    width: calc(100% - 20px);
  }

  button {
    cursor: pointer;
    background-color: #4CAF50;
    color: white;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #45a049;
  }

  table {
    width: 100%;
    border-collapse: collapse;
  }

  th, td {
    padding: 8px;
    text-align: left;
    border-bottom: 1px solid #ddd;
  }

  tr:hover {
    background-color: #f5f5f5;
  }

  .completed {
    text-decoration: line-through;
  }

  .datetime-container {
    margin-top: 10px;
  }

  .datetime-container label {
    display: block;
    margin-bottom: 5px;
  }

  select {
    padding: 10px;
    font-size: 16px;
    border-radius: 5px;
  }

  /* tambahkan class 'hidden' untuk menyembunyikan elemen */
  .hidden {
    display: none;
  }

  footer {
    background-color: #333;
    color: #fff;
    text-align: center;
    padding: 20px 0;
    position: fixed;
    width: 100%;
    bottom: 0;
  }

  footer p {
    margin: 0;
  }
</style>
