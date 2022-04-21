<template>
  <div id="todo">
    <h1>My Todo List</h1>
    <div class="todo-form">
      <input
        v-model="todoBaru"
        name="todo"
        placeholder="tambah todo"
        @keyup.enter="TAMBAH_TODO(todoBaru)"
      />
      <button @click="TAMBAH_TODO(todoBaru)">Add New Todo</button>
    </div>
    <div class="todo-list">
      <ApolloQuery
        :query="
          (gql) =>
            gql(`
        query {
            todolist_todo {
              id
              text
              is_done
            }
          }
        `)
        "
      >
        <template v-slot="{ result: { data, error } }">
          <div v-if="error">Ups Error</div>
          <div v-else-if="data" class="todo-list">
            <ul>
              <TodoItem
                v-for="(todo, index) in data.todolist_todo"
                :key="index"
                :todo="todo"
                :index="todo.id"
              />
            </ul>
          </div>
          <div v-else class="loading" />
        </template>
        <ApolloSubscribeToMore
          :document="
            (gql) =>
              gql(`
                subscription {
                  todolist_todo {
                    id
                    text
                    is_done
                  }
                }
          `)
          "
          :updateQuery="NEW_TODO"
        />
      </ApolloQuery>
    </div>
  </div>
</template>
<script>
import gql from "graphql-tag";
import TodoItem from "@/components/TodoItem.vue";
export default {
  data() {
    return {
      todoBaru: "",
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
    NEW_TODO(prev, { subscriptionData }) {
      return {
        todolist_todo: subscriptionData.data.todolist_todo,
      };
      // console.log(subscriptionData.data.todolist_todo);
    },
  },
};
</script>
<style scoped>
#todo {
  padding: 10px 0;
}
input {
  display: block;
  margin: auto;
  border-top-style: hidden;
  border-right-style: hidden;
  border-left-style: hidden;
  border-bottom-style: groove;
  outline: none;
  font-size: 1.3em;
  padding: 15px 0;
  text-align: center;
}
.todo-form {
  margin-top: 30px;
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
