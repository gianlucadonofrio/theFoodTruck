<template>
  <v-app>
    <v-container>
      <Navbar />
    </v-container>
    <h1 class="tituloCarrito">Carrito</h1>
    <v-container mt-15>
      <v-row>
        <v-col cols="12" class="d-flex mt-1 mb-3 align-center justify-center">
          <v-sheet color="grey" v-if="loader">
            <v-skeleton-loader type="v-card"> </v-skeleton-loader>
          </v-sheet>
          <v-container v-else>
            <v-card
              class="d-lg-flex align-center mt-4"
              v-for="(producto, i) in this.$store.state.cart"
              :key="i"
              color="grey lighten-5"
            >
              <v-img :src="producto.image" width="400" class="mx-auto"> </v-img>
              <v-container>
                <v-card-title style="font-size: 30px"
                  >{{ producto.product }}
                </v-card-title>
                <v-spacer></v-spacer>
                <v-container class="d-flex">
                  <v-container
                    class="d-flex flex-column justify-center"
                    style="max-width: 65%"
                  >
                    <v-row class="mb-2">
                      <v-btn @click="decrease(producto)" class="mr-2" small
                        ><v-icon>mdi-minus-circle</v-icon></v-btn
                      >
                      <h2>{{ producto.quantity }}</h2>
                      <v-btn @click="increase(producto)" class="ml-2" small
                        ><v-icon>mdi-plus-circle</v-icon></v-btn
                      >
                    </v-row>
                    <h3>Disponibles: {{ producto.disponibles }}</h3>
                  </v-container>
                  <v-row>
                    <v-spacer></v-spacer>
                    <v-card-title style="font-size: 28px"
                      >$ {{ productPrice(producto) }}</v-card-title
                    >
                  </v-row>
                </v-container>

                <v-card-actions>
                  <v-btn
                    x-large
                    class="ml-2"
                    color="error"
                    style="border-radius: 15px"
                    @click="removeFromCart(producto)"
                    >Eliminar</v-btn
                  >
                </v-card-actions>
              </v-container>
            </v-card>
            <v-container
              v-if="carritoVacio"
              class="d-flex flex-column align-center"
            >
              <v-img
                :src="require('../assets/carrito-vacio.png')"
                style="max-width: 150px"
              ></v-img>
              <h3 class="mt-4">Tu carrito está vacio</h3>
              <h1 class="mt-1">¡Continúa comprando!</h1>
              <router-link to="/menu" style="text-decoration: none">
                <v-btn
                  color="error"
                  class="mt-2 zoom"
                  x-large
                  style="border-radius: 15px"
                >
                  ¡Comprar!</v-btn
                >
              </router-link>
            </v-container>

            <v-container v-else>
              <v-container ontainer class="d-flex justify-end mb-6">
                <h2>Total: ${{ totalPrice }}</h2>
              </v-container>

              <v-dialog v-model="dialog" max-width="500px">
                <template v-slot:activator="{ on, attrs }">
                  <v-row width="100%">
                    <v-col cols="12" lg="12">
                      <v-container class="d-flex flex-column align-end">
                        <v-btn
                          width="150px"
                          x-large
                          color="error"
                          dark
                          class="mb-4"
                          @click="destroyCart()"
                        >
                          Eliminar carrito
                        </v-btn>
                        <v-btn
                          width="150px"
                          x-large
                          color="primary"
                          dark
                          v-bind="attrs"
                          v-on="on"
                        >
                          Continuar compra
                        </v-btn>
                      </v-container>
                    </v-col>
                  </v-row>
                </template>

                <v-form
                  ref="form"
                  v-if="logueado"
                  v-model="valid"
                  lazy-validation
                >
                  <v-card v-if="loginAdmin">
                    <v-card-title style="text-align: center">
                      ¡Usuarios administradores no pueden realizar pedidos!
                    </v-card-title>
                  </v-card>
                  <v-card v-else>
                    <v-card-title>
                      <span style="font-size: 24px">Tu pedido</span>
                    </v-card-title>

                    <v-card-text>
                      <v-container>
                        <v-row>
                          <v-container v-if="logueado">
                            <p>
                              tip: si te registras, no hace falta que cargues
                              tus datos cada vez que quieras hacer un pedido!
                            </p>
                          </v-container>
                          <v-col cols="12" sm="6" md="6">
                            <v-text-field
                              label="Nombre"
                              v-model="dataUser.nombre"
                              :rules="nameRules"
                            ></v-text-field>
                          </v-col>

                          <v-container>
                            <h3>¿Donde querés recibir tu pedido?</h3>
                          </v-container>
                          <v-col cols="12" sm="12" md="12">
                            <v-text-field
                              label="Dirección"
                              v-model="userAddress.direccion"
                              :rules="addresRules"
                            ></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="6">
                            <v-text-field
                              label="Localidad"
                              v-model="userAddress.localidad"
                              :rules="addresRules"
                            ></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="6">
                            <v-text-field
                              label="Provincia"
                              v-model="userAddress.provincia"
                              :rules="addresRules"
                            ></v-text-field>
                          </v-col>
                          <v-container>
                            <h3>¿Cómo querés pagar?</h3>
                            <v-radio-group v-model="toggle">
                              <v-radio
                                label="Efectivo"
                                color="primary"
                                value="efectivo"
                              ></v-radio>
                              <v-radio
                                label="Tarjeta débito/crédito"
                                color="primary"
                                value="tarjeta"
                              ></v-radio>
                            </v-radio-group>
                          </v-container>
                        </v-row>
                      </v-container>
                    </v-card-text>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="white--text red darken-1" @click="close()">
                        Cancelar
                      </v-btn>
                      <v-btn
                        color="white--text green darken-1"
                        @click="sendOrder()"
                      >
                        <v-icon left> mdi-whatsapp</v-icon>
                        Realizar pedido</v-btn
                      >
                    </v-card-actions>
                  </v-card>
                </v-form>

                <v-form ref="form" v-else v-model="valid" lazy-validation>
                  <v-card>
                    <v-card-title>
                      <span style="font-size: 24px"
                        >Pedido de {{ nameLogin() }}</span
                      >
                    </v-card-title>
                    <v-card-text>
                      <v-container>
                        <v-row>
                          <v-col cols="12" sm="6" md="6">
                            <v-text-field
                              label="Nombre"
                              :value="nameLogin()"
                              :rules="nameRules"
                            ></v-text-field>
                          </v-col>

                          <v-container>
                            <h3>¿Donde querés recibir tu pedido?</h3>
                          </v-container>
                          <v-col cols="12" sm="12" md="12">
                            <v-text-field
                              label="Dirección"
                              :value="direcc()"
                              :rules="addresRules"
                            ></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="6">
                            <v-text-field
                              label="Localidad"
                              :value="localidad()"
                              :rules="addresRules"
                            ></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="6">
                            <v-text-field
                              label="Provincia"
                              :value="provincia()"
                              :rules="addresRules"
                            ></v-text-field>
                          </v-col>
                          <v-container>
                            <h3>¿Cómo querés pagar?</h3>
                            <v-radio-group v-model="toggle">
                              <v-radio
                                label="Efectivo"
                                color="primary"
                                value="efectivo"
                              ></v-radio>
                              <v-radio
                                label="Tarjeta débito/crédito"
                                color="primary"
                                value="tarjeta"
                              ></v-radio>
                            </v-radio-group>
                          </v-container>
                        </v-row>
                      </v-container>
                    </v-card-text>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="white--text red darken-1" @click="close()">
                        Cancelar
                      </v-btn>
                      <v-btn
                        color="white--text green darken-1"
                        @click="sendOrder()"
                      >
                        <v-icon left> mdi-whatsapp</v-icon>
                        Realizar pedido</v-btn
                      >
                    </v-card-actions>
                  </v-card>
                </v-form>
              </v-dialog>
            </v-container>
          </v-container>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import Navbar from "./Navbar.vue";

export default {
  components: {
    Navbar,
  },
  data: () => ({
    loginAdmin: false,
    logueado: true,
    valid: true,
    loader: false,
    carritoVacio: false,
    dialog: false,
    toggle: true,
    nuevoIndex: -1,
    dataUser: {
      nombre: "",
    },
    userAddress: {
      direccion: "",
      localidad: "",
      provincia: "",
    },
    nameRules: [
      (v) => !!v || "El nombre es requerido",
      (v) =>
        (v && v.length <= 10) || "El nombre no debe superar los 10 caracteres",
    ],
    addresRules: [(v) => !!v || "La dirección es requerida"],
  }),
  methods: {
    loginStatus() {
      if (localStorage.logueado === "true") {
        this.logueado = false;
      }
      if (localStorage.logueadoAdmin === "true") {
        this.loginAdmin = true;
      }
    },
    removeFromCart(item) {
      this.$store.commit("removeFromCart", item);
      this.cart();
    },
    productPrice(item) {
      return item.price * item.quantity;
    },
    increase(item) {
      if (item.quantity < item.disponibles) {
        return item.quantity++;
      }
    },
    decrease(item) {
      if (item.quantity > 1) {
        return item.quantity--;
      }
    },
    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.nuevoIndex = -1;
      });
    },
    cart() {
      if (this.$store.state.cart.length == 0) {
        this.carritoVacio = true;
      }
    },
    sendOrder() {
      if (
        this.$refs.form.validate() &&
        (this.toggle == "efectivo" || this.toggle == "tarjeta")
      ) {
        let productsString = "";
        let addressMessage = "";
        this.$store.state.cart.forEach((elem) => {
          productsString =
            productsString +
            " " +
            elem.quantity +
            "x" +
            elem.product.replace(" ", "+") +
            "+-+" +
            "%24" +
            elem.price.toString() +
            ".";
          addressMessage =
            (this.userAddress.direccion || this.direcc()) +
            ", " +
            (this.userAddress.localidad || this.localidad()) +
            ", " +
            (this.userAddress.provincia || this.provincia());
        });
        window.location.href = `https://wa.me/5491132686326?text=Hola+,+mi+nombre+es+${
          this.dataUser.nombre || this.nameLogin()
        }%21%0D%0AQuiero+realizar+un+pedido+de+los+siguientes+items%3A${
          productsString +
          " La dirección de envio seria: " +
          addressMessage +
          ". El total seria: $" +
          this.totalPrice +
          ", paga en: " +
          this.toggle
        }`;
      }
    },
    destroyCart() {
      this.$store.commit("clearCart");
      this.cart();
    },
    nameLogin() {
      return localStorage.name;
    },
    direcc() {
      return localStorage.direccion;
    },
    localidad() {
      return localStorage.localidad;
    },
    provincia() {
      return localStorage.provincia;
    },
  },
  computed: {
    totalPrice() {
      let total = 0;

      for (let item of this.$store.state.cart) {
        total += item.price * item.quantity;
      }

      return total;
    },
  },
  mounted() {
    this.cart();
    this.loginStatus();
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
  },
};
</script>
<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap");
* {
  font-family: "Bebas Neue", cursive;
}
.tituloCarrito {
  background: url("../assets/bg-image-cart.jpg");
  color: white;
  display: flex;
  height: 200px;
  justify-content: center;
  align-items: center;
  font-size: 70px;
}
</style>
