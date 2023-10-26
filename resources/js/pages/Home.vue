<template>
    <div class="w-6/12 p-10 mx-auto">
Home
    </div>
</template>
<script>
import {ref, onMounted} from 'vue'
import {useRouter} from "vue-router";
import {request} from '../helper'
import Loader from '../components/Loader.vue';

export default {
    components: {
        Loader,
    },
    setup() {
        const todo = ref('')
        const todos = ref([])
        const user = ref()
        const isLoading = ref()

        let router = useRouter();
        onMounted(() => {
            authentication()
            handleTodos()
        });

        const authentication = async () => {
            isLoading.value = true
            try {
                const req = await request('get', '/api/user')
                user.value = req.data
            } catch (e) {
                await router.push('/')
            }
        }

        const handleTodos = async () => {
            try {
                const req = await request('get', '/api/todos')
                todos.value = req.data.data
            } catch (e) {
                await router.push('/')
            }
            isLoading.value = false
        }

        const handleNewTodo = async (title) => {
            try {
                const data = {title: title}
                const req = await request('post', '/api/todos', data)
                if (req.data.message) {
                    isLoading.value = false
                    return alert(req.data.message)
                }
                todos.value.push(req.data.data)
            } catch (e) {
                await router.push('/')
            }
            isLoading.value = false
        }

        const handleLogout = () => {
            localStorage.removeItem('APP_DEMO_USER_TOKEN')
            router.push('/')
        }

        const addTodo = () => {
            if (todo.value === "") {
                return alert("Todo cannot be empty");
            }
            isLoading.value = true
            handleNewTodo(todo.value)
            todo.value = ""
        }

        const checked = async (val, index) => {
            try {
                const data = {has_completed: !val.has_completed}
                const req = await request('put', `/api/todos/${val.id}`, data)
                if (req.data.message) {
                    isLoading.value = false
                    return alert(req.data.message)
                }
                todos.value[index].has_completed = !val.has_completed
            } catch (e) {
                await router.push('/')
            }
            isLoading.value = false
        }

        const deleteTodo = async (val, index) => {
            if (window.confirm("Are you sure")) {
                try {
                    const req = await request('delete', `/api/todos/${val.id}`)
                    if (req.data.message) {
                        isLoading.value = false
                        todos.value.splice(index, 1)
                    }
                } catch (e) {
                    await router.push('/')
                }
                isLoading.value = false
            }
        }

        return {
            todo,
            todos,
            user,
            checked,
            addTodo,
            isLoading,
            deleteTodo,
            handleLogout,
        }
    },
}
</script>