<template>
  <div class="header">
    <div class="search-container">
      <div class="search-bar">
        <button @click="isOpen = !isOpen" class="search init">
          <i class="fas fa-search"></i>
        </button>
        <form>
        <input v-show="!isOpen" type="search" id="location-search" name="location-search"
          placeholder="Enter a city name..." v-model="userSearch">
          <input type="submit" @click="submit">
        </form>
      </div>
      <button class="search random">
        <i class="fas fa-globe-americas"></i>
      </button>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Header',
    data() {
      return {
        isOpen: true,
        userSearch: ''
      }
    },
    methods: {
      submit(e) {
        e.preventDefault()
        this.$emit("setUserInput", this.userSearch)
        this.userSearch= ''
        this.isOpen= !this.isOpen
      }
    }
  }
</script>


<style scoped>
  .header {
    margin: 5px;
  }

  .search-container {
    display: flex;
    justify-content: space-between;
  }

  form {
    display: flex;
    width: 100%;
  }

  .search-bar {
    display: flex;
    width: 100%;
    margin-top: 5px;
  }

  .search {
    background: none;
    border: none;
    outline: none;
    cursor: pointer;
    margin: 10px;
  }

  .fas {
    color: black;
    font-size: 28px;
  }

  input[type=search] {
    font-size: 16px;
    width: 100%;
    padding-left: 10px;
    transition: width 0.4s ease-in-out;
  }

  input[type=submit] {
    display: none;
  }
</style>