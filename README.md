# vue-task

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


<style>
.app {
  display: flex;
  justify-content: center;
  margin-top: 50px;
}

.users-container {
  display: flex;
}

.user-column {
  border: 1px solid #ccc;
  padding: 10px;
  margin: 10px;
  min-width: 300px;
}

.user-header {
  font-weight: bold;
  margin-bottom: 10px;
}

.posts {
  margin-bottom: 10px;
}

.post {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
}

.post-actions {
  margin-top: 10px;
}

button {
  margin-right: 5px;
}
</style>



Также нужно прописать функции для кнопок "Редактировать", "Удалить", "Добавить пост".
Редактировать - при редактировании должно всплывать модальное окно с "Юзернейм", "Титул", "Описания", "Добавить"
Удалить- всплывает модальное окно с подтверждением и отменой удаления, в случае удаления поста, должно всплывать модальное окно с подтверждением удаления поста.
Добавление поста- аналогично Редактированию

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
