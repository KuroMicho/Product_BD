<template>
  <b-form @submit.prevent="guardarProducto">
    <div class="row mt-3">
      <div class="col-xs-6 col-sm-5 col-md-6">
        <h4>Nuevo Producto</h4>
      </div>
    </div>
    <div class="row">
      <div class="col-sm-12">
        <b-form-group label="Imagen:" label-for="imagen">
          <b-form-input
            @click="pickFile"
            required
            v-model="form.name_image"
            placeholder="Cargar imagen"
          ></b-form-input>

          <input
            type="file"
            style="display: none"
            ref="image"
            accept="image/*"
            @change="onFilePicked"
          >
        </b-form-group>

        <b-form-group label="Nombre:" label-for="nombre">
          <b-form-input
            id="nombre"
            type="text"
            required
            v-model="form.name"
            placeholder="Ingresa el nombre del producto"
          />
        </b-form-group>

        <b-form-group label="Precio:" label-for="precio">
          <b-form-input
            id="precio"
            type="number"
            required
            v-model="form.price"
            placeholder="Ingresa el precio del producto"
          />
        </b-form-group>

        <b-form-group label="Cantidad:" label-for="cantidad">
          <b-form-input
            id="cantidad"
            type="number"
            required
            v-model="form.amount"
            placeholder="Ingresa la cantidad disponible del producto"
          />
        </b-form-group>

        <b-form-group label="Categoria:" label-for="categoria">
          <b-form-select
            v-model="form.category_id"
            required
            :options="categories.data"
            value-field="id"
            text-field="name"
          ></b-form-select>
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
            <b-button variant="primary" type="submit" :disabled="guardando">Guardar</b-button>
          </b-button-group>
        </b-button-toolbar>
      </div>
    </div>
  </b-form>
</template>

<script>
const URL_CATEGORIES = "http://localhost:4000/api/categories?pagination=false";

export default {
  asyncData({ $axios }) {
    return $axios.get(URL_CATEGORIES).then(categories => {
      return {
        categories: categories.data
      };
    });
  },
  data() {
    return {
      form: {
        name: "",
        amount: "",
        price: "",
        name_image: "",
        image: "",
        category_id: ""
      },
      guardando: false
    };
  },
  methods: {
    guardarProducto() {
      this.guardando = true;
      this.$axios
        .post("http://localhost:4000/api/products", this.form)
        .then(res => {
          alert("Se ha agregado un producto");
          this.$router.push({ path: "/productos" });
        });
    },
    pickFile() {
      this.$refs.image.click();
    },
    onFilePicked(e) {
      const files = e.target.files;
      if (files[0] !== undefined) {
        this.form.name_image = files[0].name;
        if (this.form.name_image.lastIndexOf(".") <= 0) {
          return;
        }
        const fr = new FileReader();
        fr.readAsDataURL(files[0]);
        fr.addEventListener("load", () => {
          this.form.image = fr.result;
        });
      }
    }
  }
};
</script>
