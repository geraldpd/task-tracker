<template>

    <AddTask
        v-show="showAddTask"
        @add-task="addTask"
    />

    <Tasks
      @delete-task="deleteTask"
      @toggle-reminder="toggleReminder"
      :tasks="tasks"
    />
</template>

<script>
  import Tasks from '../components/Tasks.vue'
  import AddTask from '../components/AddTask.vue'

  export default {
        name: 'Home',
        components: {
            Tasks,
            AddTask,
        },
        props: {
            showAddTask: {
                type: Boolean
            }
        },
        data() {
            return {
                tasks: []
            }
        },
            methods: {
            async toggleReminder(id) {
                const taskToToggle = await this.fetchTask(id)

                const updateTask = {...taskToToggle, reminder: !taskToToggle.reminder}

                const res = await fetch(`api/tasks/${id}`, {
                method: 'put',
                headers: {
                    'Content-type': 'application/json',
                },
                body: JSON.stringify(updateTask)
                })

                const data = await res.json()

                this.tasks = this.tasks.map((task) =>
                task.id === id
                ? {...task, reminder: data.reminder}
                : task
                )
            },
            async addTask(task) {
                const res = await fetch('api/tasks', {
                method: 'post',
                headers: {
                    'Content-type': 'application/json',
                },
                body: JSON.stringify(task)
                })

                const id = await res.json()
                task.id = id

                this.tasks = [...this.tasks, task] //pushes the task to the this.tasks
            },
            async deleteTask(id) {
                if(confirm('Are you sure')) {

                const res = await fetch(`api/tasks/${id}`, {
                    method: 'delete'
                })

                res.status === 200
                    ? this.tasks = this.tasks.filter((task) => task.id !== id)
                    : alert('Error deleting task')
                }
            },
            async fetchTasks() {
                const res = await fetch('api/tasks')
                const data = await res.json()
                return data
            },
            async fetchTask(id) {
                const res = await fetch(`api/tasks/${id}`)
                const data = await res.json()
                return data
            }
        },
        async created() {
        this.tasks = await this.fetchTasks();
        }

    }
</script>