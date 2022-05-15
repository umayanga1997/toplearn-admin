<template>
  <div></div>
</template>

<script>
export default {
  name: "home_screen",
  data() {
    return {};
  },
  // beforeCreate() {
  //   this.$store.commit("systemUser/findUserData");
  // },
  mounted() {
    this.initialize();
  },
  // computed: {
  //   userData() {
  //     return this.$store.getters["systemUser/userData"];
  //   },
  // },
  methods: {
    initialize() {
      try {
        // this.loading = true;
      } catch (error) {
        console.log(error);
        this.loading = false;
      }
    },
  },
};
</script>

<style lang="scss">
.card-section {
  gap: 15px;
  white-space: normal;
  & .v-card {
    padding: 0px !important;
    margin: 0px !important;
  }
}
// .v-navigation-drawer {
//   max-height: 57px !important;
//   position: absolute;
//   background-color: green !important;
//   top: 40% !important;
// }
</style>