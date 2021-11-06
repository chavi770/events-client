<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center"></div>
      <span class="ml-2">My calendar</span>
      <span class="material-icons-outlined"> event </span>
    </v-app-bar>
    <v-main>
      <Calendar />
    </v-main>
    <Footer />
  </v-app>
</template>

<script>
import Calendar from "./components/Calendar";
import Footer from "./components/Footer";

export default {
  name: "App",
  components: {
    Calendar,
    Footer,
  },

  data: () => ({
    //
  }),
};
</script>
