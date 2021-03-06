<template>
    <div class="users">
        <div class="error" v-if="error">
            {{ error }}

            <p>
                <button @click.prevent="fetchData">
                    Try again
                </button>
            </p>
        </div>

        <ul v-if="users">
            <li v-for="{ id, name, email } in users" :key="id">
                <strong>Name:</strong> {{ name }}
                <strong>Email:</strong> {{ email }}
                | <router-link :to="{ name: 'users.edit', params: { id } }">Edit</router-link>
            </li>
        </ul>

        <div class="pagination">
            <button :disabled="! prevPage" @click.prevent="goToPrev">Previous</button>
            {{ paginationCount }}
            <button :disabled="! nextPage" @click.prevent="goToNext">Next</button>
        </div>

        <p>
            <router-link :to="{ name: 'users.create' }">Add User</router-link>
        </p>
    </div>
</template>

<script>
import users from '~/api/users'

const getUsers = (page, callback) => {
    const params = { page };

    users
        .all(page)
        .then(response => {
            callback(null, response.data);
        })
        .catch(error => {
            callback(error, error.response.data);
        });
};

export default {
    data() {
      return {
          users: null,
          error: null,
          meta: null,
          links: {
              first: null,
              last: null,
              next: null,
              prev: null,
          }
      };
    },
    computed: {
        nextPage() {
           if (! this.meta || this.meta.current_page === this.meta.last_page) {
               return;
           }

           return this.meta.current_page + 1;
        },
        prevPage() {
            if (! this.meta || this.meta.current_page === 1) {
                return;
            }

            return this.meta.current_page - 1;
        },
        paginationCount() {
            if (! this.meta) {
                return;
            }

            const { current_page, last_page } = this.meta;
            return `${current_page} of ${last_page}`;
        }
    },
    beforeRouteEnter(to, from, next) {
        getUsers(to.query.page, (err, data) => {
            next(vm => vm.setData(err, data));
        });
    },
    beforeRouteUpdate(to, from, next) {
        this.users = null;
        this.links = null;
        this.meta = null;

        getUsers(to.query.page, (err, data) => {
           this.setData(err, data);
           next();
        });
    },
    methods: {
        goToNext() {
            this.$router.push({
                name: 'users.index',
                query: {
                    page: this.nextPage,
                }
            });
        },
        goToPrev() {
            this.$router.push({
                name: 'users.index',
                query: {
                    page: this.prevPage,
                }
            });
        },
        setData(err, { data: users, links, meta}) {
            if (err) {
                this.error = err.toString();
                return;
            }

            this.users = users;
            this.links = links;
            this.meta = meta;
        }
    }
};
</script>
