<template>
  <div id="app">
    <DragSelect
      question="一二三四五六七八九十"
      @selectedChanged="selectedChanged"
    />
    <div>
      <span v-for="select in selects" :key="select">{{ select }}</span>
    </div>
  </div>
</template>

<script>
import DragSelect from "./components/DragSelect.vue";

export default {
  name: "App",
  components: {
    DragSelect,
  },
  data() {
    return {
      selects: [],
    };
  },
  methods: {
    selectedChanged(selects) {
      this.selects = selects.map(it => it.innerText);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
