  <template>
  <form @submit.prevent="search">
    <div class="row mt-3">
      <div class="col-xs-4">
        <h4>Listado de Productos</h4>
      </div>
      <div class="col-xs-2 ml-2 mr-2">
        <b-button variant="primary" href="/productos/nuevo">Nuevo</b-button>
      </div>

      <div class="col-xs-4">
        <b-form-input v-model="searchText" placeholder="Buscar productos..."></b-form-input>
      </div>

      <div class="col-xs-2">
        <b-button variant="info" type="submit">Buscar</b-button>
      </div>
    </div>

    <div class="row mt-2">
      <div class="col-sm-12">
        <b-table
          responsive
          striped
          hover
          :fields="fields"
          :items="products.data"
          id="products"
          @sort-changed="sortingChanged"
          :sort-by.sync="sortBy"
          :sort-desc.sync="sortDesc"
        >
          <template slot="acciones" slot-scope="row">
            <!--             <b-button variant="success" type="button" @click="row.item.id">Editar</b-button>
            -->
            <b-button variant="success" type="button" :href="`/productos/${row.item.id}`">Editar</b-button>

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
          aria-controls="products"
        ></b-pagination>

        <!-- <div class="modal" id="myModal" tabindex="-1" role="dialog">
          <div class="modal-dialog" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title">Edit Product</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <div class="modal-body">
                <textarea v-model="update" class="form-control">hola</textarea>
              </div>
              <div class="modal-footer">
                <button type="button" class="btn btn-primary">Save changes</button>
                <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
              </div>
            </div>
          </div>
        </div>-->
      </div>
    </div>
  </form>
</template>

<script>
const URL = "http://localhost:4000/api/products";
import { async } from "q";

export default {
  asyncData({ $axios }) {
    return $axios.get(URL).then(async result => {
      let items = [];

      items.forEach(value => {
        items.push({
          id: value.id,

          ...value.data
        });
      });

      return {
        items,
        products: result.data,
        currentPage: result.data.page
      };
    });
  },
  data() {
    return {
      fields: {
        image: {
          label: "Imagen",
          sortable: false
        },
        name: {
          label: "Nombre",
          sortable: true
        },
        price: {
          label: "Precio",
          sortable: true
        },
        amount: {
          label: "Cantidad",
          sortable: true
        },
        acciones: {
          label: "Acciones",
          sortable: true
        }
      },
      sortBy: "name",
      sortDesc: false,
      searchText: ""
    };
  },
  computed: {
    rows() {
      return this.products.total;
    },
    perPage() {
      return this.products.per_page;
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
          this.products = res.data;
        });
    }
  },
  methods: {
    sortingChanged(evt) {
      this.$axios
        .get(
          URL +
            `?page=${this.currentPage}&column_order=${evt.sortBy}
            &type_order=${evt.sortDesc ? "desc" : "asc"}
            &search=${this.searchText}`
        )
        .then(res => {
          this.products = res.data;
        });
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
          this.products = res.data;
        });
    },

    eliminar(id, index) {
      if (confirm("Esta seguro de eliminar el producto?")) {
        this.$axios
          .delete(`http://localhost:4000/api/products/${id}`)
          .then(res => {
            const products = res.data;
            this.items.splice(index, 1);
            alert("Eliminando");
          });
      } else {
        alert("Una sabia decision.");
      }
    }
  }
};
</script>

