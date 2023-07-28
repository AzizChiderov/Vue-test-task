<template>
  <div class="container">
    <div v-for="user in users" :key="user.id" class="user-column">
      <h3>{{ user.username }}</h3>
      <p>Количество постов: {{ getPostCount(user.id) }}</p>
      <div
        v-for="post in getPostsByUserId(user.id)"
        :key="post.id"
        class="post"
      >
        <h4>{{ post.title }}</h4>
        <p>{{ post.body }}</p>
        <button @click="openEditPostModal(post.id)">Редактировать</button>
        <button @click="deletePost(post.id)">Удалить</button>
      </div>
      <button @click="showAddPostModal(user.id)">Добавить пост</button>
    </div>

    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <h3>Добавить пост</h3>
        <label for="userId">User ID:</label>
        <select id="userId" v-model="newPost.userId">
          <option v-for="user in users" :key="user.id" :value="user.id">
            {{ user.id }}
          </option>
        </select>
        <label for="title">Заголовок:</label>
        <input type="text" id="title" v-model="newPost.title" />
        <label for="description">Описание:</label>
        <textarea id="description" v-model="newPost.body"></textarea>
        <button @click="addPostToUser()">Добавить</button>
        <button @click="hideAddPostModal()">Отмена</button>
      </div>
    </div>

    <div v-if="showEditPostModal" class="modal">
      <div class="modal-content">
        <h3>Редактировать пост</h3>
        <label for="editTitle">Заголовок:</label>
        <input type="text" id="editTitle" v-model="editPostData.title" />
        <label for="editDescription">Описание:</label>
        <textarea id="editDescription" v-model="editPostData.body"></textarea>
        <button @click="editPost">Редактировать</button>
        <button @click="hideEditPostModal">Отмена</button>
      </div>
    </div>

    <div v-if="showDeleteConfirmationModal" class="modal">
      <div class="modal-content">
        <h3>Пост удален</h3>
        <p>Пост успешно удален!</p>
        <button @click="hideDeleteConfirmationModal">Закрыть</button>
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, computed, ref } from "vue";
import { createStore } from "vuex";

const store = createStore({
  state: {
    users: [],
    posts: [],
  },
  mutations: {
    setUsers(state, users) {
      state.users = users;
    },
    setPosts(state, posts) {
      state.posts = posts;
    },
    addPost(state, post) {
      state.posts.push(post);
    },
    editPost(state, updatedPost) {
      const index = state.posts.findIndex((post) => post.id === updatedPost.id);
      if (index !== -1) {
        state.posts.splice(index, 1, updatedPost);
      }
    },
    deletePost(state, postId) {
      state.posts = state.posts.filter((post) => post.id !== postId);
    },
  },
  actions: {
    fetchUsers(context) {
      fetch("https://jsonplaceholder.typicode.com/users")
        .then((response) => response.json())
        .then((data) => context.commit("setUsers", data))
        .catch((error) => console.error("Error fetching users:", error));
    },
    fetchPosts(context) {
      fetch("https://jsonplaceholder.typicode.com/posts")
        .then((response) => response.json())
        .then((data) => context.commit("setPosts", data))
        .catch((error) => console.error("Error fetching posts:", error));
    },
  },
  getters: {
    getUserById: (state) => (id) => {
      return state.users.find((user) => user.id === id);
    },
    getPostsByUserId: (state) => (userId) => {
      return state.posts.filter((post) => post.userId === userId);
    },
  },
});

export default {
  setup() {
    const state = reactive({});
    const users = computed(() => store.state.users);
    const posts = computed(() => store.state.posts);
    const getPostsByUserId = (userId) => store.getters.getPostsByUserId(userId);

    const getPostCount = (userId) => {
      return posts.value.filter((post) => post.userId === userId).length;
    };

    store.dispatch("fetchUsers");
    const savedPosts = localStorage.getItem("posts");
    if (savedPosts) {
      store.commit("setPosts", JSON.parse(savedPosts));
    } else {
      store.dispatch("fetchPosts");
    }

    const savePostsToLocalStorage = () => {
      localStorage.setItem("posts", JSON.stringify(store.state.posts));
    };

    const showModal = ref(false);
    const newPost = reactive({
      userId: null,
      title: "",
      body: "",
    });

    const showAddPostModal = (userId) => {
      newPost.userId = userId;
      showModal.value = true;
    };

    const hideAddPostModal = () => {
      showModal.value = false;
    };

    const addPostToUser = () => {
      store.commit("addPost", {
        ...newPost,
        id: store.state.posts.length + 1,
      });
      hideAddPostModal();
      savePostsToLocalStorage();
    };

    const showEditPostModal = ref(false);
    const editPostData = reactive({
      postId: null,
      title: "",
      body: "",
    });

    const openEditPostModal = (postId) => {
      const post = posts.value.find((post) => post.id === postId);
      if (post) {
        editPostData.postId = post.id;
        editPostData.title = post.title;
        editPostData.body = post.body;
        showEditPostModal.value = true;
      }
    };

    const editPost = () => {
      const updatedPost = {
        id: editPostData.postId,
        title: editPostData.title,
        body: editPostData.body,
      };
      const index = store.state.posts.findIndex(
        (post) => post.id === updatedPost.id
      );
      if (index !== -1) {
        store.state.posts[index].title = updatedPost.title;
        store.state.posts[index].body = updatedPost.body;
      }
      hideEditPostModal();
      savePostsToLocalStorage();
    };

    const hideEditPostModal = () => {
      showEditPostModal.value = false;
    };

    const showDeleteConfirmationModal = ref(false);

    const deletePost = (postId) => {
      if (postId !== null && typeof postId !== "undefined") {
        const index = store.state.posts.findIndex((post) => post.id === postId);
        if (index !== -1) {
          store.state.posts.splice(index, 1);
          showDeleteConfirmationModal.value = true;
        }
      }
      savePostsToLocalStorage();
    };

    const hideDeleteConfirmationModal = () => {
      showDeleteConfirmationModal.value = false;
    };

    store.dispatch("fetchUsers");
    store.dispatch("fetchPosts");

    return {
      state,
      users,
      getPostsByUserId,
      getPostCount,
      showModal,
      newPost,
      showAddPostModal,
      hideAddPostModal,
      addPostToUser,
      showEditPostModal,
      editPostData,
      openEditPostModal,
      editPost,
      hideEditPostModal,
      showDeleteConfirmationModal,
      deletePost,
      hideDeleteConfirmationModal,
    };
  },
};
</script>

<style>
.container {
  display: flex;
  width: 100%;
}

.user-column {
  flex: 1 0 10%;
  border: 1px solid #ccc;
  padding: 10px;
  margin: 10px;
}

.post {
  border: 1px solid #ddd;
  padding: 5px;
  margin-bottom: 5px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  max-width: 400px;
  width: 90%;
}

.modal-content label {
  display: block;
  margin-bottom: 5px;
}

.modal-content input,
.modal-content textarea,
.modal-content select {
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
}

.modal-content button {
  padding: 8px 12px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.modal-content button:not(:last-child) {
  margin-right: 10px;
}

.modal-content button:hover {
  background-color: #0056b3;
}
</style>
