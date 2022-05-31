<template>
  <v-app>
    <v-navigation-drawer
      color="blue darken-4"
      v-model="drawer"
      clipped
      dark
      temporary
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar fixed dense color="blue darken-4" dark app elevation="1">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-toolbar-title v-text="'Top-Learn Admin'" />
      <v-spacer />

      <v-btn icon @click="signOut()"
        ><v-icon class="appBar-icon">mdi-logout</v-icon></v-btn
      >
    </v-app-bar>
    <v-main>
      <v-container fluid>
        <Nuxt />
      </v-container>
    </v-main>
    <v-footer absolute app>
      <span
        >&copy; {{ new Date().getFullYear() + " Powered by Apec Lanka" }}</span
      >
    </v-footer>
    <!-- Message -->
    <v-snackbar
      v-model="isAlertShow"
      top
      shaped
      content-class="snack_content"
      centered
      :color="
        alertType == 'error'
          ? 'red darken-4'
          : alertType == 'warning'
          ? 'orange darken-4'
          : 'green darken-4'
      "
    >
      {{ message }}
    </v-snackbar>
  </v-app>
</template>

<script>
import Cookies from "js-cookie";

export default {
  name: "DefaultLayout",
  data() {
    return {
      drawer: false,
      items: [
        {
          icon: "mdi-apps",
          title: "Home",
          to: "/home",
        },
        {
          icon: "mdi-numeric",
          title: "Grades",
          to: "/grades",
        },
        {
          icon: "mdi-file-table",
          title: "Subjects",
          to: "/subjects",
        },
        {
          icon: "mdi-apache-kafka",
          title: "Topics",
          to: "/topics",
        },
        {
          icon: "mdi-video-box",
          title: "Videos",
          to: "/videos",
        },
        {
          icon: "mdi-note-multiple",
          title: "Tests",
          to: "/tests",
        },
        // {
        //   icon: "mdi-video-vintage",
        //   title: "Live Class",
        //   to: "/live_class",
        // },
        {
          icon: "mdi-note",
          title: "Tutes",
          to: "/tutes",
        },
        {
          icon: "mdi-account-multiple",
          title: "Teachers",
          to: "/users/teachers",
        },
        {
          icon: "mdi-account-group",
          title: "Students",
          to: "/users/students",
        },
      ],
    };
  },
  beforeCreate() {
    this.$fire.auth.onAuthStateChanged(async (user) => {
      if (user) {
        this.$store.commit("systemUser/findUserData");
        // This is working with already signin and cookies setted session
        if (this.$store.getters["systemUser/userData"] != null) {
          if (this.$route.path == "/user/auth" || this.$route.path == "/") {
            this.$router.replace("/home").catch(() => {});
          }
        } else {
          this.$router.replace("/user/auth").catch(() => {});
        }
      } else {
        this.$router.replace("/user/auth").catch(() => {});
      }
    });
  },

  computed: {
    isAlertShow() {
      return this.$store.getters["alertState/isAlertShow"];
    },
    alertType() {
      return this.$store.getters["alertState/alert_type"];
    },
    message() {
      return this.$store.getters["alertState/msg"];
    },
  },
  methods: {
    async signOut() {
      try {
        await this.$fire.auth
          .signOut()
          .then(async () => {
            // localStorage.removeItem("systemuser_admin");
            Cookies.remove("access_token_admin");
            this.$store.dispatch("alertState/message", [
              "Sign out successfully.",
              "success",
            ]);
            // this.$router.replace("/user/auth").catch(() => {});
            // this.$router.go().catch(() => {});
            // window.history.go("/"); // Only Web
          })
          .catch((error) => {
            this.$store.dispatch("alertState/message", [error, "error"]);
          });
      } catch (error) {
        this.$store.dispatch("alertState/message", [error, "error"]);
      }
    },
  },
};
</script>
