<template>
  <div>
    <h1>Products</h1>
    <ul>
      <li v-for="product in products" :key="product.id">
        <template v-if="editProductId === product.id">
          <input v-model="editProduct.name" placeholder="Name" />
          <input v-model="editProduct.description" placeholder="Description" />
          <input v-model="editProduct.price" placeholder="Price" />
          <input v-model="editProduct.stocks" placeholder="Stocks" />
          <input v-model="editProduct.branch" placeholder="Branch" />
          <button @click="updateProduct(product.id)">Save</button>
          <button @click="cancelEdit">Cancel</button>
        </template>
        <template v-else>
          {{ product.name }} - {{ product.price }}
          <button @click="startEdit(product)">Edit</button>
          <button @click="deleteProduct(product.id)">Delete</button>
        </template>
      </li>
    </ul>
    <div>
      <input v-model="newProduct.name" placeholder="Name" />
      <input v-model="newProduct.description" placeholder="Description" />
      <input v-model="newProduct.price" placeholder="Price" />
      <input v-model="newProduct.stocks" placeholder="Stocks" />
      <input v-model="newProduct.branch" placeholder="Branch" />
      <button @click="addProduct">Add Product</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ProductsComponent",
  data() {
    return {
      products: [],
      newProduct: {
        name: "",
        description: "",
        price: "",
        stocks: "",
        branch: "",
      },
      editProduct: {
        name: "",
        description: "",
        price: "",
        stocks: "",
        branch: "",
      },
      editProductId: null,
    };
  },
  mounted() {
    this.fetchProducts();
  },
  methods: {
    fetchProducts() {
      axios
        .get("http://localhost:3000/products")
        .then((response) => {
          //this.products = response.data; //non-hateoas
          this.products = response.data._embedded.products; //hateoas
        })
        .catch((error) => {
          console.error("There was an error fetching the products:", error);
        });
    },
    addProduct() {
      axios
        .post("http://localhost:3000/products", this.newProduct)
        .then((response) => {
          this.products.push(response.data);
          this.newProduct = {
            name: "",
            description: "",
            price: "",
            stocks: "",
            branch: "",
          }; // Reset form
        })
        .catch((error) => {
          console.error("There was an error adding the product:", error);
        });
    },
    deleteProduct(id) {
      axios
        .delete(`http://localhost:3000/products/${id}`)
        .then(() => {
          this.products = this.products.filter((product) => product.id !== id);
        })
        .catch((error) => {
          console.error("There was an error deleting the product:", error);
        });
    },
    startEdit(product) {
      this.editProductId = product.id;
      this.editProduct = {
        name: product.name,
        description: product.description,
        price: product.price,
        stocks: product.stocks,
        branch: product.branch,
      };
    },
    updateProduct(id) {
      axios
        .put(`http://localhost:3000/products/${id}`, this.editProduct)
        .then((response) => {
          const index = this.products.findIndex((product) => product.id === id);
          this.products.splice(index, 1, response.data);
          this.cancelEdit(); // Reset edit mode
        })
        .catch((error) => {
          console.error("There was an error updating the product:", error);
        });
    },
    cancelEdit() {
      this.editProductId = null;
      this.editProduct = {
        name: "",
        description: "",
        price: "",
        stocks: "",
        branch: "",
      }; // Reset edit form
    },
  },
};
</script>
