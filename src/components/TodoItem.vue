<template>
  <li class="todo">
    <div class="content">
      <h3 v-if="!isEdit && !todo.is_done">{{ todo.text }}</h3>
      <h3 v-if="!isEdit && todo.is_done" class="is-completed">
        {{ todo.text }}
      </h3>
      <input
        v-if="isEdit"
        v-model="editTodo"
        @keyup.enter="EDIT_TODO_VALUE(index, editTodo)"
      />
    </div>
    <div class="content">
      <i
        class="fas fa-check-circle fa-2x"
        @click="EDIT_DONE(index, !todo.is_done)"
      />
      <i class="fas fa-edit fa-2x" @click="changeIsEdit" />
      <i class="fas fa-trash fa-2x" @click="DELETE_TODO(index)" />
    </div>
  </li>
</template>

<script>
import gql from "graphql-tag";
export default {
  props: {
    todo: Object,
    index: Number,
  },

  data() {
    return {
      editTodo: "",
      isEdit: false,
    };
  },

  created() {
    this.changeEdit();
  },

  methods: {
    changeEdit() {
      this.editTodo = this.todo.text;
    },
    changeIsEdit() {
      this.isEdit = !this.isEdit;
      this.changeEdit();
    },
    async EDIT_TODO_VALUE(index, value) {
      await this.$apollo.mutate({
        mutation: gql`
          mutation ($value: String, $index: Int) {
            update_todolist_todo(
              where: { id: { _eq: $index } }
              _set: { text: $value }
            ) {
              returning {
                id
              }
            }
          }
        `,
        variables: {
          index,
          value,
        },
      });
      this.changeIsEdit();
    },
    async EDIT_DONE(index, value) {
      await this.$apollo.mutate({
        mutation: gql`
          mutation ($value: Boolean, $index: Int) {
            update_todolist_todo(
              where: { id: { _eq: $index } }
              _set: { is_done: $value }
            ) {
              returning {
                id
              }
            }
          }
        `,
        variables: {
          index,
          value,
        },
      });
      if (this.isEdit) {
        this.changeIsEdit();
      }
    },
    async DELETE_TODO(id) {
      await this.$apollo.mutate({
        mutation: gql`
          mutation ($id: Int) {
            delete_todolist_todo(where: { id: { _eq: $id } }) {
              returning {
                id
              }
            }
          }
        `,
        variables: {
          id,
        },
      });
    },
  },
};
</script>

<style scoped>
.todo {
  margin-bottom: 20px;
  display: flex;
  flex-direction: row;
  align-items: center;
}
h3 {
  margin: 0px;
  padding: 0px;
}
li {
  list-style-type: none;
  width: 50%;
  margin: auto;
  padding: 15px 0;
  background-color: rgb(216, 216, 216);
  border-radius: 15px;
}
.content {
  flex: 1;
}
.content > i {
  margin: 0 6px;
}
.content > input {
  font-size: 1em;
  border-style: hidden;
  padding: 5px 0;
  text-align: center;
  border-radius: 5px;
}
i {
  cursor: pointer;
}
.fa-trash {
  color: red;
}
.fa-check-circle {
  color: rgb(38, 180, 38);
}
.fa-edit {
  color: rgb(237, 177, 65);
}
.is-completed {
  text-decoration: line-through;
  color: brown;
}
</style>
