<template>
  <v-app>
    <v-container>
      <Navbar />
    </v-container>
    <Menu />
    <Footer />
  </v-app>
</template>

<script>
// @ is an alias to /src
import Navbar from "@/components/Navbar.vue";
import Footer from "@/components/Footer.vue";
import Menu from "@/components/ListadoProductos.vue";

export default {
  name: "menu",
  components: {
    Navbar,
    Menu,
    Footer,
  },
};
</script>
