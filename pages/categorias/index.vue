  <template>
  <div>
    <div>
      <b-modal
        id="Model-categorias"
        ref="modal"
        title="Nueva Categoria"
        @show="resetModal"
        @hidden="resetModal"
        @ok="handleOk"
        cancel-title="Cancelar"
        ok-title="Guardar"
      >
        <form ref="form" id="myModal" @submit.prevent="guardarCategoria">
          <b-form-group label="Imagen:" label-for="imagen">
            <b-form-input @click="pickFile" v-model="form.name_image" placeholder="Cargar imagen"></b-form-input>

            <input
              type="file"
              style="display: none"
              ref="image"
              accept="image/*"
              @change="onFilePicked"
            >
          </b-form-group>

          <b-form-group
            label="Nombre"
            label-for="nombre-input"
            invalid-feedback="Nombre is required"
          >
            <b-form-input
              id="categorias"
              placeholder="Ingresa la Categoria"
              v-model="form.name"
              required
            ></b-form-input>
          </b-form-group>
        </form>
      </b-modal>
    </div>

    <div class="row mt-3">
      <div class="col-xs-4">
        <h4>Listado de Categorias</h4>
      </div>
      <div class="col-xs-2 ml-2 mr-2">
        <b-button v-b-modal.Model-categorias>Nueva Categoria</b-button>
      </div>

      <div class="col-xs-4">
        <b-form-input v-model="searchText" placeholder="Buscar categorias..."></b-form-input>
      </div>

      <div class="col-xs-2">
        <b-button variant="info" @click="search">Buscar</b-button>
      </div>
    </div>

    <div class="row mt-2">
      <div class="col-sm-12">
        <b-table
          responsive
          striped
          hover
          :fields="fields"
          :items="categories.data"
          id="categories"
          @sort-changed="sortingChanged"
          :sort-by.sync="sortBy"
          :sort-desc.sync="sortDesc"
        >
          <template slot="acciones" slot-scope="row">
            <b-button variant="success" :href="`/categorias/${row.item.id}`">Editar</b-button>

            <b-button
              variant="danger"
              type="button"
              @click="eliminar(row.item.id,row.index)"
            >Eliminar</b-button>
          </template>
          <template slot="image" slot-scope="data">
            <b-img width="220" :src="`http://localhost:4000/images/${data.item.image}`"></b-img>
          </template>
        </b-table>

        <b-pagination
          v-model="currentPage"
          :total-rows="rows"
          :per-page="perPage"
          aria-controls="categories"
        ></b-pagination>
      </div>
    </div>
  </div>
</template>

<script>
const URL = "http://localhost:4000/api/categories";

export default {
  asyncData({ $axios }) {
    return $axios.get(URL).then(result => {
      return {
        categories: result.data,
        currentPage: result.data.page
      };
    });
  },
  data() {
    return {
      form: {
        image: "",
        name_image: "",
        name: ""
      },
      fields: {
        image: {
          label: "Imagen",
          sortable: false
        },
        name: {
          label: "Nombre",
          sortable: true
        },
        acciones: {
          label: "Acciones",
          sortable: false
        }
      },
      sortBy: "name",
      sortDesc: false,
      searchText: ""
    };
  },
  computed: {
    rows() {
      return this.categories.total;
    },
    perPage() {
      return this.categories.per_page;
    }
  },
  watch: {
    currentPage(value) {
      this.$axios
        .get(
          URL +
            `?page=${value}&column_order=${this.sortBy}
            &type_order=${this.sortDesc ? "desc" : "asc"}
            &search=${this.searchText}`
        )
        .then(res => {
          this.categories = res.data;
        });
    }
  },
  methods: {
    guardarCategoria() {
      // Exit when the form isn't valid
      if (!this.checkFormValidity()) {
        return;
      }

      // Push the name to submitted names
      /* this.items.push(this.form); */
      // Hide the modal manually
      this.$nextTick(() => {
        this.$refs.modal.hide(
          this.$axios
            .post("http://localhost:4000/api/categories", this.form)
            .then(res => {
              this.$router.push({ path: "/categorias" });
            })
        );
      });
    },
    checkFormValidity() {
      const valid = this.$refs.form.checkValidity();
      this.name = valid ? "valid" : "invalid";
      return valid;
    },
    resetModal() {
      /* this.nombre = '' */
      this.name = null;
    },
    handleOk(bvModalEvt) {
      // Prevent modal from closing
      bvModalEvt.preventDefault();
      // Trigger submit handler
      this.guardarCategoria();
    },
    sortingChanged(evt) {
      this.$axios
        .get(
          URL +
            `?page=${this.currentPage}&column_order=${evt.sortBy}
            &type_order=${evt.sortDesc ? "desc" : "asc"}
            &search=${this.searchText}`
        )
        .then(res => {
          this.categories = res.data;
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
    },
    search() {
      this.$axios
        .get(
          URL +
            `?page=${this.currentPage}&column_order=${this.sortBy}
            &type_order=${this.sortDesc ? "desc" : "asc"}
            &search=${this.searchText}`
        )
        .then(res => {
          this.categories = res.data;
        });
    },
    eliminar(id, index) {
      if (confirm("Esta seguro de eliminar la categoria?")) {
        this.$axios
          .delete(`http://localhost:4000/api/categories/${id}`)
          .then(res => {
            const categorias = res.data;
            this.items.splice(index, 1);
            alert("Eliminando");
          });
      } else {
        alert("Una sabia decision.");
      }
    },
    editar(id) {
      const form = {
        name: this.categories.name
      };
      const URL = `http://localhost:4000/api/categories/${id}`;
      this.$axios.put(URL, form).then(res => {
        alert("Categoria Actualizada");
        this.$router.push({ path: "/categorias" });
      });
    }
  }
};
</script>

