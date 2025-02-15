<template>
    <div>
        <h5 class="mt-2 mb-3">Category Favorites</h5>
        <ul class="list-group">
            <li class="list-group-item" v-for="(category, index) in getCategoriesAndFavorites" v-on:click="favoriteIndex = index" v-bind:key="category.id">
                <span href="#">{{category.name | capitalize}}</span>
                 <div v-show="favoriteIndex == index">
                    <table class="table table-bordered">
                    <thead>
                        <tr>
                            <th scope="col">Title</th>
                            <th scope="col">Description</th>
                            <th scope="col">More Info</th>
                            <th scope="col">Ranking</th>
                            <th scope="col">Update</th>
                            <th scope="col">Delete</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="favorite in favorites(category)" v-bind:key="favorite.id">
                            <th scope="row">{{favorite.title}}</th>
                            <td v-if="favorite.description">{{favorite.description}}</td>
                            <td v-else class="empty-description">empty</td>
                            <td>
                                <a href="#" v-on:click="showMetadata(JSON.parse(favorite.objectMetadata))">View</a>
                            </td>
                            <td>{{favorite.ranking}}</td>
                            <td><router-link :to="`/favorites/update/${favorite.id}`" class="text-warning">Update</router-link></td>
                            <td><a href="#" class="text-danger" v-on:click="deleteFavorite(favorite.id)">Delete</a></td>
                        </tr>
                    </tbody>
                </table>
                </div>
            </li>
        </ul>
        <p class="no-favorite-text" v-if="getCategoriesAndFavorites && !getCategoriesAndFavorites.length">You have not added any favorite thing yet.<br> Click the plus sign to add one.</p>
    </div>
</template>

<script>
// eslint-disable-next-line
import gql from 'graphql-tag';
import Swal from 'sweetalert2';
import { SET_APP_ERROR_MESSAGE } from '../mutationTypes';

const swalWithBootstrapButtons = Swal.mixin({
  customClass: {
    confirmButton: 'btn btn-danger',
    cancelButton: 'btn btn-danger',
  },
  buttonsStyling: false,
});

const allFavoritesByCategory = gql`query {
  getCategoriesAndFavorites {
    id
    name
    favoriteThings {
      id
      title
      description
      objectMetadata
      ranking
      createdDate
    }
  }
}
`;

const deleteFavorite = gql`mutation  ($id: Int!){
  deleteFavoriteThing(id: $id) {
    favoriteThing {
      id
      title
      ranking
    }
  }
}
`;
export default {
  data() {
    return {
      getCategoriesAndFavorites: [],
      favoriteIndex: -1,
    };
  },
  apollo: {
    getCategoriesAndFavorites: {
      query: allFavoritesByCategory,
    },
  },
  async created() {
    try {
      await this.$apollo.queries.getCategoriesAndFavorites.refetch();
    } catch (err) {
      this.$store.dispatch(SET_APP_ERROR_MESSAGE, err.message.split(':')[1]);
    }
  },
  methods: {
    favorites(category) {
      return category.favoriteThings;
    },
    async deleteFavorite(id) {
      const self = this;
      const result = await Swal.fire({
        title: 'Are you sure?',
        type: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes',
      });
      if (result.value) {
        try {
          await self.$apollo.mutate({
            mutation: deleteFavorite,
            variables: {
              id,
            },
          });
          Swal.fire(
            'Deleted!',
            'Favorite thing deleted',
            'success',
          );
        } catch (err) {
          swalWithBootstrapButtons.fire(
            'Cancelled',
            'Favorite thing could not be deleted',
            'error',
          );
          return;
        }
        try {
          await self.$apollo.queries.getCategoriesAndFavorites.refetch();
        } catch (err) {
          this.$store.dispatch(SET_APP_ERROR_MESSAGE, err.message.split(':')[1]);
        }
      }
    },
    showMetadata(metadata) {
      let html = '';
      let swalHtml = '';
      const keys = Object.keys(metadata);
      for (let i = 0; i < keys.length; i++) {
        html += `<tr><td>${keys[i]}</td>
                    <td>${metadata[keys[i]]}</td></tr>`;
      }
      if (!keys.length) {
        swalHtml = 'no metadata';
      } else {
        swalHtml = `
                <table class="table">
                <thead class="thead-dark">
                    <tr>
                        <th scope="col">key</th>
                        <th scope="col">value</th>
                    </tr>
                </thead>
                <tbody>
                    ${html}
                </tbody>
                </table>`;
      }
      Swal.fire({
        title: '<h6>More Info</h6>',
        html: swalHtml,
        focusConfirm: false,
        confirmButtonText:
                    'Ok',
        confirmButtonAriaLabel: 'Thumbs up, great!',
        showCancelButton: false,
      });
    },
  },
  filters: {
    capitalize(value) {
      if (!value) return '';
      value = value.toString();
      return value.charAt(0).toUpperCase() + value.slice(1);
    },
  },
};
</script>

<style lang="scss" scoped>
    li {
        list-style: none;
    }
    ul {
        li {
            cursor: pointer;
            span {
                color: #61B7FF;
                float: left;
            }
        }
    }
    .empty-description {
      color: #ccc;
      font-style: italic;
    }
</style>
