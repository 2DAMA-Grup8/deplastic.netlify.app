<template>
  <v-card>
    <v-toolbar class="bg-green-darken-1" flat>
      <v-toolbar-title>Product</v-toolbar-title>
      <v-btn variant="outlined" @click="dialog = true"> New Item </v-btn>
    </v-toolbar>
    <v-table>
      <thead>
        <tr>
          <th class="text-left">ID</th>
          <th class="text-left">NAME</th>
          <th class="text-left">IMAGE</th>
          <th class="text-left">DESCRIPTION</th>
          <th class="text-left">PRICE</th>
          <th class="text-left">ACTIONS</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in listProducts" :key="product.id">
          <td class="text-left">{{ product.id }}</td>
          <td class="text-left">{{ product.nom }}</td>
          <v-avatar>
            <v-img :src="product.url"></v-img>
          </v-avatar>
          <td class="text-left" width="500">{{ product.description }}</td>
          <td class="text-left">{{ product.price }}</td>

          <td class="text-left">
            <v-dialog v-model="dialog" width="600" persistent>
              <template v-slot:activator="{ props }">
                <v-btn
                  class="bg-green-darken-1"
                  icon="mdi-pencil"
                  size="small"
                  v-bind="props"
                  @click="inputProduct = product"
                />
              </template>
              <v-card>
                <v-form ref="form">
                  <v-text-field
                    v-model="inputProduct.nom"
                    label="Name"
                  ></v-text-field>
                  <v-text-field
                    v-model="inputProduct.url"
                    label="Image"
                  ></v-text-field>
                  <v-text-field
                    v-model="inputProduct.description"
                    label="Description"
                  ></v-text-field>
                  <v-text-field
                    v-model="inputProduct.price"
                    label="Price"
                  ></v-text-field>
                  <v-btn color="success" class="mr-4" @click="editProduct">
                    Submit
                  </v-btn>
                  <v-btn
                    color="blue-darken-1"
                    variant="text"
                    @click="closeDialog"
                  >
                    Close
                  </v-btn>
                </v-form>
              </v-card>
            </v-dialog>
            <v-btn
              class="bg-green-darken-1"
              icon="mdi-trash-can-outline"
              size="small"
              @click="deleteProduct(product.id)"
            />
            <v-snackbar :timeout="3000" v-model="snackbar">
              <label color="red">ERROR </label>{{ errorText }}
              <template v-slot:actions>
                <v-btn variant="text" @click="snackbar = false"> Close </v-btn>
              </template>
            </v-snackbar>
          </td>
        </tr>
      </tbody>
    </v-table>
  </v-card>
</template>

<script>
export default {
  data() {
    return {
      snackbar: false,
      errorText: "Something went wrong!",
      listProducts: [],
      inputProduct: {},
      dialog: false,
    };
  },
  methods: {
    async getData() {
      const res = await fetch("/.netlify/functions/api/products");
      const finalRes = await res.json();
      this.listProducts = finalRes;
    },
    deleteProduct(id) {
      fetch("/.netlify/functions/api/products", {
        method: "DELETE",
        body: JSON.stringify({ id: id }),
        headers: { "Content-Type": "application/json" },
      })
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          console.log(data);
          window.location.reload();
        });
    },
    editProduct() {
      if (this.inputProduct.id) {
        fetch("/.netlify/functions/api/products", {
          method: "PUT",
          body: JSON.stringify({
            id: this.inputProduct.id,
            nom: this.inputProduct.nom,
            url: this.inputProduct.url,
            description: this.inputProduct.description,
            price: this.inputProduct.price,
          }),
          headers: { "Content-Type": "application/json" },
        })
          .then((response) => {
            return response.json();
          })
          .then((data) => {
            console.log(data);
            if (data.success) {
              this.closeDialog();
            } else {
              this.snackbar = true;
            }
          });
      } else {
        this.createProduct();
      }
    },
    createProduct() {
      fetch("/.netlify/functions/api/products", {
        method: "POST",
        body: JSON.stringify({
          nom: this.inputProduct.nom,
          url: this.inputProduct.url,
          description: this.inputProduct.description,
          price: this.inputProduct.price,
        }),
        headers: { "Content-Type": "application/json" },
      })
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          if (data.success) {
            this.closeDialog();
          } else {
            this.snackbar = true;
          }
        });
    },
    closeDialog() {
      this.inputProduct = {};
      this.dialog = false;
      window.location.reload();
    },
  },
  mounted() {
    this.getData();
  },
};
</script>
