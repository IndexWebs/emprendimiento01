<template>
  <div class="relative overflow-x-auto shadow-md sm:rounded-lg">
    <table class="w-full text-sm text-left text-gray-500">
      <thead class="text-xs text-white uppercase bg-secondary">
        <tr>
          <th scope="col" class="px-6 py-3">Product name</th>
          <th scope="col" class="px-6 py-3">Category</th>
          <th scope="col" class="px-6 py-3">Price</th>
          <th scope="col" class="px-6 py-3">Description</th>
          <th scope="col" class="px-6 py-3">Action</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="product in filteredProducts"
          :key="product.id"
          class="bg-white border-b"
        >
          <th
            scope="row"
            class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap"
          >
            {{ product.name }}
          </th>
          <td class="px-6 py-4">{{ product.category }}</td>
          <td class="px-6 py-4">${{ product.price }}</td>
          <td class="px-6 py-4">{{ product.description }}</td>
          <td class="px-6 py-4 flex items-center space-x-2">
            <nuxt-link
              :to="'admin/' + product.handle"
              class="font-medium text-secondary hover:underline"
              ><EditIcon class="w-5 h-5"
            /></nuxt-link>
            <span
              @click="deleteDocument(product.id)"
              class="font-medium text-primary hover:underline"
              ><TrashIcon class="w-5 h-5"
            /></span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import EditIcon from "~/components/shared/icons/EditIcon.vue";
import TrashIcon from "~/components/shared/icons/TrashIcon.vue";
import { mapState, mapActions } from 'vuex';

export default {
  components: { EditIcon, TrashIcon },
  middleware: "auth",
  layout: "admin",
  data() {
    return {
      products: [],
    };
  },
  computed: {
    ...mapState(['filteredProducts']),
  },
  created() {
    this.fetchProducts();
  },
  methods: {
    ...mapActions(['fetchProducts', 'filterProducts']),
  
    deleteDocument(id) {
      const confirmDelete = window.confirm(
        "¿Estás seguro de que quieres eliminar este documento?"
      );
      if (!confirmDelete) {
        return; 
      }
      const ref = db.collection("products").doc(id);
      ref
        .get()
        .then((doc) => {
          if (doc.exists) {
            const data = doc.data();
            console.log("data", data);
            const imageUrl = data.image;
            ref
              .delete()
              .then(() => {
                const storageRef = firebase.storage().refFromURL(imageUrl);
                storageRef
                  .delete()
                  .then(() => {
                    console.log("Imagen eliminada de Firebase Storage.");
                  })
                  .catch((error) => {
                    console.error("Error al eliminar la imagen:", error);
                  });
              })
              .catch((error) => {
                console.error("Error al eliminar el documento:", error);
              });
          } else {
            console.log("No se encontró el documento.");
          }
        })
        .catch((error) => {
          console.error("Error al obtener el documento:", error);
        });
      this.getDocuments();
    },
  },
};
</script>
