<template>
  <div class="post">
    <div>
      <h2>Пост {{ post.id }}</h2>
      <h3 :title="post.title">{{ shortBody(post.title) }}</h3>
    </div>
    <div>
      <h2>Содержание</h2>
      <h3 :title="post.body">
        {{ openFullState ? shortBody(post.body) : post.body }}
        <button @click="openFull()">
          {{ openFullState ? shortText : longText }}
        </button>
      </h3>
    </div>
    <div class="wrapper-postbtn">
      <button @click="editPost()" class="post-btn">Редактирование поста</button>
      <button @click="deletePost()" class="post-btn">Х</button>
    </div>
    <hr />
  </div>
</template>
<script>
export default {
  props: ["post"],
  data() {
    return {
      shortText: "Полный текст",
      longText: "Скрыть ",

      openFullState: true,
    };
  },
  computed: {},
  methods: {
    openFull() {
      this.openFullState = !this.openFullState;
    },
    shortBody(text) {
      return text.length > 65 ? text.slice(0, 65) + "..." : text;
    },
    deletePost() {
      console.log(this.post.id);
      this.$emit("deletePost", this.post.id);
    },
    editPost() {
      this.$emit("editPost", this.post.id);
    },
  },
};
</script>
<style scoped>
.post {
  position: relative;
  max-width: 600px;
  min-width: 600px;
  margin-bottom: 15px;
}
.wrapper-postbtn {
  position: absolute;
  top: 15px;
  right: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}
.post-btn {
  margin: 5px;
  cursor: pointer;
}
</style>
