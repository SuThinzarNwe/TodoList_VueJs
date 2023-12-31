<template>
    <div id="app" class="h-100 w-full flex items-center justify-center bg-teal-lightest font-sans">
        <div class="bg-white rounded shadow p-6 m-4 w-full lg:w-3/4 lg:max-w-lg mt-20">
            <div class="mb-4">
                <h1 class="text-2xl text-grey-darkest font-semibold">Todo</h1>
                <div class="flex mt-4">
                    <input
                        class="shadow appearance-none border rounded w-full py-3 px-3 mr-4 mb-5 text-grey-darker focus:border-blue-500 focus:outline-none focus:border-2"
                        id="add_item" type="text" placeholder="Add new..." v-model="newTodo" @keyup.enter="addItem" />
                    <button
                        class="flex-no-shrink p-2 border-2 rounded text-teal border-teal bg-blue-500 hover:text-white hover:bg-teal h-12"
                        @click="addItem">
                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                            stroke="white" class="w-5 h-6">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
                        </svg>
                    </button>
                </div>
                <div v-if="isError" class="text-red-500 flex items-center text-lg">
                    <span>Invalid input. Please enter a valid task.</span>
                    <button @click="isError = false" class="ml-2 focus:outline-none">
                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor"
                            class="w-4 h-4">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12">
                            </path>
                        </svg>
                    </button>
                </div>
            </div>

            <!-- Add filter buttons -->
            <div class="flex justify-center mb-4">
                <button class="border-blue-500 flex-1 border hover:bg-blue-50 p-2 shadow-sm mr-2"
                    :class="{ 'bg-blue-50': filterTodo === 'all' }" @click="filterTodo = 'all'">
                    All
                </button>
                <button class="border-blue-500 flex-1 border hover:bg-blue-50 p-2 shadow-sm mr-2"
                    :class="{ 'bg-blue-50': filterTodo === 'active' }" @click="filterTodo = 'active'">
                    Active
                </button>
                <button class="border-blue-500 flex-1 border hover:bg-blue-50 p-2 shadow-sm"
                    :class="{ 'bg-blue-50': filterTodo === 'complete' }" @click="filterTodo = 'complete'">
                    Completed
                </button>
            </div>

            <ul class="mb-4">
                <li v-for="(todo, index) in todoFilter" :key="todo.id" class="flex items-center bg-gray-200 mb-5 p-2">
                    <!-- Add checkbox for completed task -->
                    <input type="checkbox" :id="'checkbox-' + todo.id" v-model="todo.completed"
                        class="mr-2 h-5 w-6 cursor-pointer" v-if="!todo.editItem" />
                    <label :for="'checkbox-' + todo.id" class="px-2 py-1 w-48">
                        <span v-if="!todo.editItem" @dblclick="toggleEdit(todo)" class="cursor-pointer"
                            :class="{ 'line-through': todo.completed }">
                            {{ todo.task }}
                        </span>
                        <input v-else type="text" v-model="todo.task" :ref="'task'" @keyup.enter="doneEdit(todo)"
                            @blur="doneEdit(todo)" class="px-2 py-1" @keyup.esc="doneEdit(todo)" />
                    </label>
                    <button v-if="!todo.editItem"
                        class="flex items-center border-black border-solid border-1 px-2 py-1 ml-auto text-white"
                        @click="deleteTodo(index)">
                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                            class="w-6 h-6 transition-transform duration-300 transform-gpu hover:scale-110" stroke="red">
                            <path stroke-linecap="round" stroke-linejoin="round"
                                d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0" />
                        </svg>
                    </button>
                </li>
            </ul>

            <span class="text-gray-500">{{ remaining }} {{ remaining > 1 ? "items" : "item" }} left.</span>
            <div class="flex justify-center mb-4">
                <button class="border-blue-500 flex-1 border hover:bg-blue-50 p-2 shadow-sm mr-2"
                    :class="{ 'filter-active': filterTodo === 'all' }" @click="toggleCheckAll">
                    {{ checkAllButtonText }}
                </button>
                <button class="border-blue-500 flex-1 border hover:bg-blue-50 p-2 shadow-sm mr-2"
                    :class="{ 'bg-blue-50': filterTodo === 'active' }" @click="clearCompleted">
                    Clear Completed
                </button>
            </div>
        </div>
    </div>
</template>

<style>
@import "./assets/css/index.css";
</style>

<script>
export default {
    name: "DemoProj",
    data() {
        return {
            newTodo: "",
            filterTodo: "all",
            checkAllButtonText: "Check All",
            todos: [],
            isError: false,
        };
    },
    mounted() {
        this.restoreData();
    },
    watch: {
        todos: {
            handler() {
                this.saveData();
            },
            deep: true,
        },
    },//prevent to checkbox to remain the same when the page refreshs
    computed: {
        remaining() {
            if (this.filterTodo === "all") {
                return this.todos.filter((todo) => !todo.completed).length;
            } else if (this.filterTodo === "active") {
                return this.todos.filter((todo) => !todo.completed).length;
            } else if (this.filterTodo === "complete") {
                return this.todos.filter((todo) => todo.completed).length;
            }
            // If the filterTodo value is not recognized, return the default remaining count
            return this.todos.filter((todo) => !todo.completed).length;
        },

        todoFilter() {
            if (this.filterTodo === "active") {
                return this.todos.filter((todo) => !todo.completed);
            } else if (this.filterTodo === "complete") {
                return this.todos.filter((todo) => todo.completed);
            } else {
                return this.todos;
            }
        },
    },
    methods: {
        addItem() {
            const trimmedTodo = this.newTodo.trim();

            if (trimmedTodo === "") {
                // Display error message for empty input
                this.isError = true;
                return false;
            }

            this.isError = false;
            this.todos.push({
                id: this.todoId++,
                task: trimmedTodo,
                completed: false,
                editItem: false,
            });
            this.newTodo = "";

            this.saveData();
        },

        deleteTodo(index) {
            this.todos.splice(index, 1);
        },
        toggleEdit(todo) {
            todo.editItem = !todo.editItem;
            this.$nextTick(() => {
                if (todo.editItem) {
                    const input = this.$refs.task.find(
                        (ref) => ref.id === `task-${todo.id}`
                    );
                    input.focus();
                }
            });
        },
        doneEdit(todo) {
            if (todo.task.trim() === "") {
                this.isError = true;
                return;
            }
            todo.editItem = false; this.isError = false;
            todo.editItem = false;
            this.saveData();
        },
        saveData() {
            localStorage.setItem("todos", JSON.stringify(this.todos));
        },
        saveOnEscape(todo) {
            todo.editItem = false;
            this.isError = false;
            this.saveData();
        },

        restoreData() {
            const storedTodos = localStorage.getItem("todos");
            if (storedTodos) {
                this.todos = JSON.parse(storedTodos);
                this.todoId =
                    this.todos.length > 0 ? this.todos[this.todos.length - 1].id + 1 : 1;
            }
        },
        toggleCheckAll() {
            if (this.checkAllButtonText === "Check All") {
                this.checkAllButtonText = "Unchecked All";
                this.todos.forEach((todo) => {
                    todo.completed = true;
                });
            } else {
                this.checkAllButtonText = "Check All";
                this.todos.forEach((todo) => {
                    todo.completed = false;
                });
            }
        },
        clearCompleted() {
            this.todos = this.todos.filter((todo) => !todo.completed);
        },
    },
};
</script>
