<template>
  <div id="todo">
    <h1>My Todo List</h1>
    <div class="todo-form">
      <input
        v-model="todoBaru"
        name="todo"
        placeholder="tambah todo"
        @keyup.enter="TAMBAH_TODO(todoBaru)"
        v-if="!isSearch"
        autocomplete="off"
      />
      <input
        v-model="search"
        type="number"
        min="1"
        name="search"
        v-if="isSearch"
        placeholder="masukkan id"
        autocomplete="off"
      />
      <button @click="CHANGE_BUTTON">Search or Input</button>
    </div>
    <div class="todo-list">
      <template>
        <div v-if="$apollo.queries.listTodo.error">Ups Error</div>

        <div v-else-if="$apollo.queries.listTodo">
          <ul>
            <TodoItem
              v-for="(todo, index) in listTodo"
              :key="index"
              :todo="todo"
              :index="todo.id"
            />
          </ul>
        </div>
        <div v-else class="loading" />
      </template>
    </div>
  </div>
</template>
<script>
import gql from "graphql-tag";
import TodoItem from "@/components/TodoItem.vue";
export default {
  apollo: {
    listTodo: {
      query() {
        if (this.search != "") {
          return gql`
            query ($id: Int) {
              todolist_todo(where: { id: { _eq: $id } }) {
                id
                text
                is_done
              }
            }
          `;
        } else {
          return gql`
            query {
              todolist_todo {
                id
                text
                is_done
              }
            }
          `;
        }
      },
      update: (data) => data.todolist_todo,
      variables() {
        return {
          id: parseInt(this.search),
        };
      },

      subscribeToMore: {
        document: gql(`
                subscription {
                  todolist_todo {
                    id
                    text
                    is_done
                  }
                }
          `),
        updateQuery: (prev, { subscriptionData }) => {
          return {
            todolist_todo: subscriptionData.data.todolist_todo,
          };
        },
      },
    },
  },
  data() {
    return {
      todoBaru: "",
      isSearch: false,
      search: 0,
    };
  },

  components: {
    TodoItem,
  },

  methods: {
    async TAMBAH_TODO(text) {
      if (this.todoBaru != "") {
        await this.$apollo.mutate({
          mutation: gql`
            mutation ($text: String) {
              insert_todolist_todo(objects: { text: $text }) {
                returning {
                  id
                }
              }
            }
          `,
          variables: {
            text,
          },
        });
        this.todoBaru = "";
      }
    },

    CHANGE_BUTTON() {
      this.isSearch = !this.isSearch;
    },
  },
};
</script>
<style scoped>
#todo {
  padding: 10px 0;
}
input {
  border-top-style: hidden;
  border-right-style: hidden;
  border-left-style: hidden;
  border-bottom-style: groove;
  outline: none;
  font-size: 1.3em;
  padding: 15px 0;
  text-align: center;
  margin-bottom: 20px;
}
.todo-form {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

button {
  margin-top: 10px;
  padding: 10px;
  background-color: rgb(180, 199, 211);
  border: none;
  cursor: pointer;
  border-radius: 5px;
  font-weight: bold;
  outline: none;
}

.loading {
  margin: 20px auto 0;
  border: 16px solid #f3f3f3; /* Light grey */
  border-top: 16px solid #3498db; /* Blue */
  border-radius: 50%;
  width: 120px;
  height: 120px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
