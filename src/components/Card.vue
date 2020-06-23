<template>
  <div id="card-choose">
    <h4>Choose a book</h4>
    <div class="list-books">
      <div v-for="book in books" :key="book">
        <h5>{{ book.title }}</h5>
        <ul class="list-chapters">
          <li v-for="chapter in book.chapters" :key="chapter">
            {{ chapter.title }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ChooseBook",
  props: ["books"],
};
</script>

<style scoped>
#card-choose {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 20px;
}
</style>
