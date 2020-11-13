<template>
  <div id="students_list">
    <UserDetail />
  </div>
</template>

<script>
import UserDetail from "./UserDetail.vue";

export default {
  name: "UserList",
  data() {
    return {};
  },
  components: {
    UserDetail,
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#students_list {
  margin: auto;
  width: 60%;
  /* margin-top: 60px; */
  background-color: white;
  border-radius: 10px;
  box-shadow: 1px 1px 3px gray;
}
</style>
