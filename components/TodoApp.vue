<template>
    <div class="todo-app">
        <div class="todo-app__actions">
            <div class="filters">
                <button :class=" {active:filter === 'all' }" @click="changeFilter('all')">
                    모든 항목({{ total }})
                </button>
                <button :class=" {active:filter === 'active' }" @click="changeFilter('active')">
                    해야 할 항목 ({{ activeCount }})
                </button>
                <button :class=" {active:filter === 'completed' }" @click="changeFilter('completed')">
                    완료된 항목 ({{ completedCount }})
                </button>
            </div>
            <div class="actions">
                <input v-model="allDone" type="checkbox"/>
                <button @click="clearCompleted">
                    완료된 항목 삭제
                </button>
            </div>
        </div>
        <div class="todo-app__list">
            <todo-item v-for="todo in filteredTodos" :key="todo.id" :todo="todo" @update-todo="updateTodo"
                @delete-todo="deleteTodo" />
        </div>
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
import _forEachRight from 'lodash/forEachRight'
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
            todos: [],
            filter: 'all'
        }
    },
    created() {
        this.initDB()
    },
    computed: {
        filteredTodos() {
            switch (this.filter) {
                case 'all':
                    default:
                    return this.todos
                case 'active': //해야할 항목
                    return this.todos.filter(todo => !todo.done)
                case 'completed': //완료된 항목
                    return this.todos.filter(todo => todo.done)
            }
        },
        total (){
            return this.todos.length
        },
        activeCount () {
            return this.todos.filter(todo => !todo.done).length
        },
        completedCount () {
            return this.total -this.activeCount
        },
        allDone :{
            get () {
                return this.total === this.completedCount && this.total > 0
            },
            set (checked) {
                this.completeAll(checked)
            }
        }
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
            const foundIndex = _findIndex(this.todos, { id: todo.id })
            this.$delete(this.todos, foundIndex)
        },
        changeFilter (filter) {
            this.filter = filter
        },
        completeAll(checked) {
            // DB
           const newTodos = this.db
            .get('todos')
            .forEach(todo => {
                todo.done = checked
            })
            .write()

            // Local todos
            // this.todos.forEach(todo => {
            //     todo.done = checked
            // })

            this.todos = _cloneDeep(newTodos)

        },
        clearCompleted () {
            
            // 배열을 앞에서 부터 삭제하면 뒤에 삭제되지 않는 문제 발생
            // this.todos.forEach(todo => {
            //     if(todo.done) {
            //         this.deleteTodo(todo)
            //     }
            // })

            // 배열 뒤에서 부터 삭제하는 네이티브 코드
            // this.todos.reduce((list, todo, index) => {
            //     if(todo.done) {
            //         list.push(index)
            //     }
            //     return list
            // }, [])
            // .reverse()
            // .forEach(index => {
            //     this.deleteTodo(this.todos[index])
            // })

            //lodash 플러그인 사용해서 간단하게 처리
            _forEachRight (this.todos, todo=> {
                if(todo.done) {
                    this.deleteTodo(todo)
                }
            })
        }
    }

}
</script>

<style scoped lang="scss">
    button.active {
        font-weight:bold;
        background:#006eff;
        color:#fff;
        border-color:#006eff
    }
</style>