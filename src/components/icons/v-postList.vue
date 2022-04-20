<template>
  <div class="wrapper-header">
    <!-- <button @click="downloud()" class="header-btn">Загрузить</button> -->
    <button @click="addPost()" class="header-btn">Добавление поста</button>
    <button @click="sortPost()" class="header-btn">Сортировать посты</button>
  </div>
  <hr />
  <div class="postList">
    <VPost
      v-for="post in paginatedData"
      :post="post"
      @deletePost="deletePostId = $event"
      @editPost="editPostId = $event"
    />
  </div>
  <div class="navbar">
    <button
      @click="pageNumber = 0"
      :disabled="pageNumber == 0"
      class="form-btn"
    >
      Первая
    </button>
    <button @click="prevPage" :disabled="pageNumber == 0" class="form-btn">
      Предыдущая
    </button>
    <button
      @click="nextPage"
      :disabled="pageNumber >= pageCount - 1"
      class="form-btn"
    >
      Следующая
    </button>
    <button
      @click="pageNumber = pageCount - 1"
      :disabled="pageNumber >= pageCount - 1"
      class="form-btn"
    >
      Последняя
    </button>
  </div>
  <div class="modal" v-if="overlay || editForm || addForm">
    <div class="modal-window" v-if="!editForm && !addForm">
      <h2>Уверен, что надо удалить пост {{ deletePostId }}?</h2>
      <div class="form-btn-wrapper">
        <button @click="deleteSure()" class="form-btn">Да</button>
        <button @click="notDelete()" class="form-btn">Нет</button>
      </div>
    </div>

    <div class="modal-window" v-if="editForm && !addForm">
      <h2>Форма редактрования постa {{ editPostId }}</h2>
      <div>
        <div class="input-wrapper">
          <input type="text" v-model="chengedTitle" class="input-text" />
          <input type="text" v-model="chengedBody" class="input-text" />
        </div>
        <div class="form-btn-wrapper">
          <button
            @click="saveSure()"
            class="form-btn"
            :disabled="chengedTitle == '' || chengedBody == ''"
          >
            Сохранить
          </button>
          <button @click="notSave()" class="form-btn">Не cохранять</button>
        </div>
      </div>
    </div>
    <div class="modal-window" v-if="addForm">
      <h2>Форма добавления</h2>
      <div class="input-wrapper">
        <input type="text" v-model="chengedTitle" class="input-text" />
        <input type="text" v-model="chengedBody" class="input-text" />
      </div>
      <div class="form-btn-wrapper">
        <button
          @click="addSure()"
          :disabled="chengedTitle == '' || chengedBody == ''"
          class="form-btn"
        >
          Добавить
        </button>
        <button @click="notAdd()" class="form-btn">Закрыть</button>
      </div>
    </div>

    <div class="overlay" v-if="overlay || editForm || addForm"></div>
  </div>
</template>
<script>
import VPost from "./v-post.vue";
export default {
  components: { VPost },
  data() {
    return {
      postsList: [],
      sortUp: true,
      deletePostId: "",
      editPostId: "",
      overlay: false,
      editForm: false,
      addForm: false,
      chengedTitle: "",
      chengedBody: "",
      pageNumber: 0,
    };
  },
  created() {
    this.downloud();
  },
  watch: {
    deletePostId() {
      if (this.deletePostId != -1) {
        this.overlay = !this.overlay;
      }
    },
    editPostId() {
      if (this.editPostId != -1) {
        this.chengedTitle = this.vFormEdit.title;
        this.chengedBody = this.vFormEdit.body;
        this.editForm = !this.editForm;
      }
    },
  },
  computed: {
    vFormEdit() {
      return this.postsList.find((item) => item.id === this.editPostId);
    },
    pageCount() {
      let l = this.postsList.length,
        s = 4;
      return Math.ceil(l / s);
    },
    paginatedData() {
      const start = this.pageNumber * 4,
        end = start + 4;

      return this.postsList.slice(start, end);
    },
  },
  methods: {
    nextPage() {
      this.pageNumber++;
    },
    prevPage() {
      this.pageNumber--;
    },
    downloud() {
      fetch("https://jsonplaceholder.typicode.com/posts")
        .then((response) => response.json())
        .then((json) => {
          this.postsList = json;
        });
    },
    addPost() {
      this.addForm = !this.addForm;
    },
    deletePost() {
      fetch(`https://jsonplaceholder.typicode.com/posts/${this.deletePostId}`, {
        method: "DELETE",
      });
    },
    sortPost() {
      this.sortUp = !this.sortUp;
      this.sortUp
        ? this.postsList.sort((a, b) => a.id - b.id)
        : this.postsList.sort((a, b) => b.id - a.id);
    },
    deleteSure() {
      this.deletePost();
      this.postsList = this.postsList.filter(
        (item) => item.id != this.deletePostId
      );
      this.overlay = !this.overlay;
    },
    notDelete() {
      this.overlay = !this.overlay;
      this.deletePostId = -1;
    },
    editPost() {
      const editedPost = {
        title: this.chengedTitle,
        body: this.chengedBody,
        userId: 1,
        id: this.editPostId,
      };
      fetch(`https://jsonplaceholder.typicode.com/posts/${this.editPostId}`, {
        method: "PATCH",
        body: JSON.stringify(editedPost),
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
      })
        .then((response) => response.json())
        .then((json) => {
          const index = this.postsList.findIndex(
            (item) => item.id === this.editPostId
          );
          this.postsList.splice(index, 1, json);
          this.editPostId = -1;
          this.chengedTitle = "";
          this.chengedBody = "";
        });
    },
    saveSure() {
      this.editForm = !this.editForm;
      this.editPost();
    },
    notSave() {
      this.editForm = !this.editForm;
      this.editPostId = -1;
      this.chengedTitle = "";
      this.chengedBody = "";
    },
    addSure() {
      this.addForm = !this.addForm;
      const newPost = {
        title: this.chengedTitle,
        body: this.chengedBody,
        userId: 1,
        id: this.sortUp
          ? this.postsList[this.postsList.length - 1].id + 1
          : this.postsList[0].id + 1,
      };
      fetch("https://jsonplaceholder.typicode.com/posts", {
        method: "POST",
        body: JSON.stringify(newPost),
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
      }).then((response) => response.json());

      this.sortUp
        ? this.postsList.push(newPost)
        : this.postsList.unshift(newPost);

      this.chengedTitle = "";
      this.chengedBody = "";
    },
    notAdd() {
      this.addForm = !this.addForm;
    },
  },
};
</script>
<style>
.postList {
  overflow: auto;
  overflow-x: hidden;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.wrapper-header {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  width: 100%;
  background-color: aquamarine;
  z-index: 5;
}
.header-btn {
  margin: 5px;
  cursor: pointer;
}
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}
.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: black;
  opacity: 0.7;
  z-index: 0;
}
.modal-window {
  width: 550px;
  border-radius: 10px;
  box-shadow: 0 10px 15px black;
  background-color: aliceblue;
  position: relative;
  z-index: 3;
  padding: 20px;
}
.input-wrapper {
  display: flex;
  flex-direction: column;
}
.input-text {
  display: flex;
  flex-direction: column;
  padding: 5px;
  margin: 5px;
}
.form-btn-wrapper {
  display: flex;
  margin: 5px;
  flex-direction: row;
  justify-content: center;
}
.form-btn {
  margin: 5px;
  cursor: pointer;
}
.navbar {
  position: fixed;
  width: 100%;
  z-index: 5;
  bottom: 0;
  background-color: aquamarine;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
