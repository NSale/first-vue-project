<template>
    <AddTask v-show="showAddTask" @add-task="addTask" />
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks='tasks'/>
</template>

<script>
    import Tasks from '../components/Tasks'
    import AddTask from '../components/AddTask'
    export default {
        name: 'Home',
        components: {
            Tasks, 
            AddTask
        },
        props: {
            showAddTask: Boolean
        },
        data() {
            return {
                "tasks": []
            }
        },
        methods: {
            async addTask(task) {
            const res = await fetch('api/tasks', {
                method: 'POST',
                headers: {
                'Content-type': 'application/json',
                },
                body: JSON.stringify(task)
            })
            const newTask = await res.json()
            this.tasks = [...this.tasks, newTask]
            },

            async deleteTask(id) {
            if (confirm('Are you sure?')) {
                const res = await fetch(`api/tasks/${id}`, {
                method: 'DELETE'
                })
                res.status === 200 
                ? (this.tasks = this.tasks.filter((task) => task.id !== id)) 
                : alert('Something went wrong with deleting the task')
            }
            },

            async toggleReminder(id) {
            const taskToToggle = await this.fetchTask(id)  
            const updateTask = { ...taskToToggle, reminder: !taskToToggle.reminder }
            const res = await fetch(`api/tasks/${id}`, {
                method: 'PUT',
                headers: {
                'Content-type': 'application/json',
                },
                body: JSON.stringify(updateTask)
            })
            const updatedTask = await res.json()
            this.tasks = this.tasks.map((task) => task.id === id 
                ? {...task, reminder: updatedTask.reminder} 
                : task)
            },

            async fetchTasks() {
            const res = await fetch('api/tasks')
            const tasks = await res.json()
            return tasks
            },

            async fetchTask(id) {
            const res = await fetch(`api/tasks/${id}`)
            const task = await res.json()
            return task
            }
        },
        async created() {
            this.tasks = await this.fetchTasks()
        }
    }
</script>
