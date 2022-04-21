<template>
  <div>
    <h1>My Todo List</h1>
    <div class="todo-form">
      <input v-model="newTodo" name="todo" placeholder="tambah todo" />
      <button>Add New Todo</button>
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
            }
          }
        `)
        "
      >
        <template v-slot="{ result: { data, error } }">
          <div v-if="error">Ups Error</div>
          <div v-else-if="data" class="todo-list">
            <ul>
              <li
                v-for="todo in data.todolist_todo"
                :key="todo.id"
                class="todo"
              >
                <div class="content">
                  <h3>{{ todo.text }}</h3>
                </div>
                <div class="content">
                  <i class="fas fa-check-circle fa-2x"></i>
                </div>
              </li>
            </ul>
          </div>
          <div v-else class="loading" />
        </template>
      </ApolloQuery>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      newTodo: "",
    };
  },
};
</script>
<style scoped>
body {
  padding-top: 1em;
  padding-bottom: 1em;
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
.todo {
  margin-bottom: 20px;
  display: flex;
  flex-direction: row;
  align-items: center;
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

h3 {
  margin: 0px;
  padding: 0px;
  text-transform: capitalize;
}
li {
  list-style-type: none;
  width: 50%;
  margin: auto;
  padding: 15px 0;
  background-color: rgb(216, 216, 216);
  border-radius: 20px;
}
.content {
  flex: 1;
}
i {
  cursor: pointer;
  color: rgb(45, 163, 45);
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
