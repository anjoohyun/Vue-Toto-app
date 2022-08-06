<template>
    <div>
        <todo-item v-for="todo in todos" :key="todo.id" :todo="todo" @update-todo="updateTodo"
            @delete-todo="deleteTodo" />
        <hr />
        <todo-creator @create-todo="createTodo" />
    </div>
</template>

<script>
import lowdb from 'lowdb'
import LocalStorage from 'lowdb/adapters/LocalStorage'
import cryptoRandomString from 'crypto-random-string'
import _cloneDeep from 'lodash/cloneDeep'
import _find from 'lodash/find'
import _assign from 'lodash/assign'
import _findIndex from 'lodash/findIndex'
import _remove from 'lodash/remove'
import TodoCreator from './TodoCreator.vue'
import TodoItem from './TodoItem.vue'

export default {
    components: {
        TodoCreator,
        TodoItem
    },
    data() {
        return {
            db: null,
            todos: []
        }
    },
    created() {
        this.initDB()
    },
    methods: {
        initDB() {
            const adapter = new LocalStorage('todo-app')
            this.db = lowdb(adapter)

            const hasTodos = this.db.has('todos').value()
            if (hasTodos) {
                this.todos = _cloneDeep(this.db.getState().todos)
            } else {
                //Local DB 초기화
                this.db.defaults({
                    todos: [] //Collection           
                }).write()
            }

        },
        createTodo(title) {
            const newTodo = {
                id: cryptoRandomString({ length: 10 }),
                title: title,
                createdAt: new Date(),
                updatedAt: new Date(),
                done: false
            }

            //create DB
            this.db
                .get('todos') //lodash
                .push(newTodo) //lodash
                .write() //lowdb

            // Crate client
            this.todos.push(newTodo)
        },
        updateTodo(todo, value) {
            this.db
                .get('todos')
                .find({ id: todo.id })
                .assign(value)
                .write()

            const foundTodo = _find(this.todos, { id: todo.id })
            //Object.assign(foundTodo, value)
            _assign(foundTodo, value)
        },
        deleteTodo(todo) {
            console.log('deleteTodo')
            //디비 삭제
            this.db
                .get('todos')
                .remove({ id: todo.id })
                .write()

            //화면갱신
            const foundIndex = _findIndex(this.todos, { id: todo.id})
            this.$delete(this.todos, foundIndex)
        }
    }

}
</script>