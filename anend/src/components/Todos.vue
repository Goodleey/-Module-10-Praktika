<template>
  <div class="container">
    <div class="col-sm-10">
      <h1>Задачи</h1>
      <h3>Всего задач {{tasksNum}}, выполнено {{numCompleted}},
        осталось {{tasksNum - numCompleted}}</h3>
      <!-- Двустороннее изменение значения visible -->
      <confirmation
        :message="confirmationMessage"
        :visible.sync="showConfirmation">
      </confirmation>
      <button type="button"
        id="task-add"
        class="btn btn-success btn-sm align-left d-block"
        v-b-modal.todo-modify-modal
        @click="addTodo()">
        Добавить задачу
      </button>

      <table class="table table-dark table-stripped table-hover">
        <thead class="thead-light">
          <tr>
            <th>Uid</th>
            <th>Описание</th>
            <th>Выполнена?</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td>{{ todo.description }}</td>
            <td>
              <span v-if="todo.is_completed">Выполнено</span>
              <span v-else>Не выполнено</span>
            </td>
            <td>
              <div class="btn-group" role="group">
                <button type="button" class="btn btn-secondary btn-sm"
                v-b-modal.todo-modify-modal
                @click="updateTodo(todo)">Обновить</button>
                &nbsp;
                <button type="button"
                  class="btn btn-danger btn-sm"
                  @click="deleteTodo(todo)">X</button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Одна модальная форма для добавления и удаления задач -->
      <!-- Лэйблы берутся из computed свойства ModalLabels -->
      <b-modal ref="modifyTodoModal"
         id="todo-modify-modal"
         :title="ModalLabels[0]"
         hide-footer
         @hidden="onHide">
        <b-form @submit="onModifySubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-modify-description-group"
                label="Описание:"
                label-for="form-modify-description-input">
        <b-form-input id="form-modify-description-input"
                  type="text"
                  v-model="modifyTodoForm.description"
                  required
                  :placeholder = "ModalLabels[1]">
        </b-form-input>
        </b-form-group>
        <b-form-group id="form-modify-complete-group">
          <b-form-checkbox-group v-model="modifyTodoForm.is_completed" id="form-modify-checks">
            <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
        <b-button-group>
          <b-button type="submit" variant="primary">{{ModalLabels[2]}}</b-button>
          <b-button type="reset" variant="danger">Сброс</b-button>
        </b-button-group>
        </b-form>
      </b-modal>

      <button type="button"
        id="tasks-clear"
        class="btn btn-danger btn-sm align-left d-block"
        @click="clearTodo()">
        Очистить список
      </button>

    </div>
  </div>
</template>


<style>
button#task-add {
  margin-top: 20px;
  margin-bottom: 20px;
}
h1, h3, td {
  text-align: left;
}
.todo-uid {
  text-align: right;
}
</style>


<script>
import Confirmation from './Confirmation.vue';

export default {
  name: 'Todos',
  data() {
    return {
      todos: [],
      tasksNum: 0,
      modifyTodoForm: {
        uid: 0,
        description: '',
        is_completed: [],
      },
      confirmationMessage: '',
      showConfirmation: false,
      operation: 'add',
    };
  },
  computed: {
    // Обновляет лэйблы для формы
    ModalLabels() {
      const title = this.operation === 'add'
        ? 'Добавление задачи' : 'Обновление задачи';
      const placeholder = this.operation === 'add'
        ? 'Введите задачу' : null;
      const submitButton = this.operation === 'add'
        ? 'Добавить' : 'Обновить';
      return [title, placeholder, submitButton];
    },
    // Вычисляет количество выполненных заданий
    numCompleted() {
      let c = 0;
      for (let i = 0; i < this.todos.length; i += 1) {
        if (this.todos[i].is_completed) c += 1;
      }
      return c;
    },
  },
  methods: {
    getTodos() {
      const tasks = localStorage.getItem('Tasks');
      if (tasks !== '') {
        const parsed = JSON.parse(tasks);
        this.todos = parsed;
        this.tasksNum = parsed.length;
      }
    },
    resetForm() {
      this.modifyTodoForm.description = '';
      this.modifyTodoForm.is_completed = [];
    },
    // Добавляет/изменяет задания
    onModifySubmit(event) {
      event.preventDefault();
      this.$refs.modifyTodoModal.hide();
      if (this.operation === 'add') {
        const requestData = {
          description: this.modifyTodoForm.description,
          is_completed: this.modifyTodoForm.is_completed[0] !== undefined
            ? this.modifyTodoForm.is_completed[0] : false,
          uid: this.todos.length,
        };
        this.todos.push(requestData);
        localStorage.setItem('Tasks', JSON.stringify(this.todos));
        this.confirmationMessage = `Задача "${requestData.description}" добавлена`;
      } else if (this.operation === 'update') {
        this.todos[this.modifyTodoForm.uid].description = this.modifyTodoForm.description;
        this.todos[this.modifyTodoForm.uid].is_completed = this.modifyTodoForm.is_completed[0]
        !== undefined ? this.modifyTodoForm.is_completed[0] : false;
        localStorage.setItem('Tasks', JSON.stringify(this.todos));
        this.confirmationMessage = 'Задача обновлена';
      }
      this.getTodos();
      this.showConfirmation = true;
      this.resetForm();
    },
    onReset(event) {
      event.preventDefault();
      this.$refs.modifyTodoModal.hide();
      this.getTodos();
      this.resetForm();
    },
    addTodo() {
      this.operation = 'add';
    },
    // Проверяет корректность UID
    checkUID(todo) {
      if (this.todos.indexOf(todo) === todo.uid) return true;
      this.confirmationMessage = 'Некорректный UID';
      this.showConfirmation = true;
      this.updateUIDs();
      this.getTodos();
      return false;
    },
    // Подготовка формы для изменения задания
    updateTodo(todo) {
      if (!this.checkUID(todo)) return; // Проверка UID
      this.operation = 'update';
      this.modifyTodoForm.description = todo.description;
      this.modifyTodoForm.is_completed = todo.is_completed ? [true] : [];
      this.modifyTodoForm.uid = todo.uid;
    },
    // Обновляет UIDs
    updateUIDs() {
      for (let i = 0; i < this.todos.length; i += 1) {
        this.todos[i].uid = i;
      }
    },
    deleteTodo(todo) {
      if (!this.checkUID(todo)) return;
      this.todos.splice(this.todos.indexOf(todo), 1);
      this.updateUIDs();
      localStorage.setItem('Tasks', JSON.stringify(this.todos));
      this.getTodos();
      this.confirmationMessage = 'Задача удалена из списка';
      this.showConfirmation = true;
    },
    onHide() {
      this.getTodos();
      this.resetForm();
    },
    clearTodo() {
      localStorage.setItem('Tasks', JSON.stringify([]));
      this.getTodos();
    },
  },
  components: {
    confirmation: Confirmation,
  },
  created() {
    if (localStorage.getItem('Tasks') === null) {
      localStorage.setItem('Tasks', '');
    }
    this.getTodos();
  },
};
</script>
