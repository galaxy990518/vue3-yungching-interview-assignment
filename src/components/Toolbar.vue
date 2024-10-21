<template>
  <nav class="toolbar">
    <router-link to="/" class="toolbar-link">首頁</router-link>
    <router-link to="/favorites" class="toolbar-link">我的最愛</router-link>
  </nav>
</template>

<script setup></script>

<style lang="scss" scoped>
.toolbar {
  padding: 15px 0;
  text-align: center;
  border-bottom: 1px solid #e0e0e0;

  .toolbar-link {
    color: #333;
    text-decoration: none;
    padding: 10px 20px;
    margin: 0 10px;
    transition: all 0.3s ease;
    font-weight: 500;

    &:hover {
      color: $primary-color;
    }

    &.router-link-active {
      color: $primary-color;
      border-bottom: 2px solid $primary-color;
    }
  }
}
</style>
