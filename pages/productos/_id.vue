<template>
  <div>
    <b-form>
      <div class="row mt-3">
        <div class="col-xs-6 col-sm-5 col-md-6">
          <h4>EDICION DE PRODUCTO</h4>
        </div>
      </div>
      <div class="row">
        <div class="col-sm-12">
          <b-form-group label="Imagen:" label-for="imagen">
            <b-form-file
              id="imagen"
              accept="image/*"
              v-model="imageProduct"
              disabled
              placeholder="Cargar Imagen"
              drop-placeholder="Cargar Imagen"
            />
          </b-form-group>

          <b-form-group label="Nombre:" label-for="nombre">
            <b-form-input
              id="nombre"
              type="text"
              required
              v-model="producto.name"
              placeholder="Ingresa el nombre del producto"
            />
          </b-form-group>

          <b-form-group label="Precio:" label-for="precio">
            <b-form-input
              id="precio"
              type="number"
              required
              v-model="producto.price"
              placeholder="Ingresa el precio del producto"
            />
          </b-form-group>

          <b-form-group label="Cantidad:" label-for="cantidad">
            <b-form-input
              id="cantidad"
              type="number"
              required
              v-model="producto.amount"
              placeholder="Ingresa la cantidad disponible del producto"
            />
          </b-form-group>

        </div>
      </div>

      <div class="row">
        <div class="col-xs-12 offset-sm-5">
          <b-spinner variant="primary" v-if="guardando"></b-spinner>
        </div>
      </div>

      <div class="row">
        <div class="col-xs-12 offset-sm-5">
          <b-button-toolbar>
            <b-button-group right class="mx-2">
              <b-button href="/productos">Volver</b-button>
              <b-button
                variant="primary"
                @click="editarProducto(producto.id)"
                :disabled="guardando"
              >Guardar</b-button>
            </b-button-group>
          </b-button-toolbar>
        </div>
      </div>
    </b-form>
  </div>
</template>

<script>
import { async } from "q";
const URL = "http://localhost:4000/api/products/";

export default {
  asyncData({ $axios, params }) {
    return $axios.get(URL + params.id).then(result => {
      return {
        id: params.id,
        producto: result.data.products,
        currentPage: result.data.page,
        categories: result.data
      };
    });
  },
  data() {
    return {
      form: {
        name: "",
        amount: "",
        price: "",
      },
      guardando: false,
      imageProduct: null
    };
  },
  methods: {
    editarProducto(id) {
      this.guardando = true;
      const form = {
        name: this.producto.name,
        amount: this.producto.amount,
        price: this.producto.price
      };
      const URL = `http://localhost:4000/api/products/${id}`;
      this.$axios.put(URL, form).then(res => {
        alert("Producto Actualizado");

        this.$router.push({ path: "/productos" });
      });
    }
  }
};
</script>
