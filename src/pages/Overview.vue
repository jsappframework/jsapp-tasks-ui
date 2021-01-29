<template>
  <div class="content">
    <div class="container-fluid">
      <div class="row">
        <div class="col-md-12">
          <card>
            <template slot="header">
              <h5 class="title">Task list</h5>
              <p v-if="error" class="text-danger">{{error}}</p>
              <base-input type="text"
                        label="Description"
                        placeholder="Description"
                        v-model="task.description">
              </base-input>
              <button v-if="!editingTask" type="submit" class="btn btn-info btn-fill float-right" @click.prevent="addTask">
                Add Task
              </button>
              <button v-if="editingTask" type="submit" class="btn btn-default btn-fill float-right ml-1" @click.prevent="cancelEditingTask">
                Cancel
              </button>
              <button v-if="editingTask" type="submit" class="btn btn-info btn-fill float-right" @click.prevent="updateTask">
                Update Task
              </button>
            </template>
            <l-table :data="tableData" :columns="tableData.columns">
              <template slot="columns"></template>

              <template slot-scope="{ row }">
                <td width="40">
                  <base-checkbox v-model="row._done"></base-checkbox>
                </td>
                <td>{{ row.description }}</td>
                <td width="40" class="td-actions text-right">
                  <button
                    type="button"
                    class="btn-simple btn btn-xs btn-info"
                    v-tooltip.top-center="editTooltip"
                    @click.prevent="editTask(row.id)"
                  >
                    <i class="fa fa-edit"></i>
                  </button>
                  <button
                    type="button"
                    class="btn-simple btn btn-xs btn-danger"
                    v-tooltip.top-center="deleteTooltip"
                    @click.prevent="deleteTask(row.id)"
                  >
                    <i class="fa fa-times"></i>
                  </button>
                </td>
              </template>
            </l-table>
          </card>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import StatsCard from "src/components/Cards/StatsCard.vue";
import LTable from "src/components/Table.vue";
const URL = 'http://localhost:3000/tasks'

export default {
  components: {
    LTable
  },
  data() {
    return {
      error: null,
      editingTask: false,
      task: {description:''},
      editTooltip: "Edit Task",
      deleteTooltip: "Remove",
      tableData: []
    };
  },
  mounted() {
    this.getTasks()
  },
  methods: {
    getTasks() {
      this.error = null;
      axios.get(URL).then((result) => {
        this.tableData = result.data.fetchResult.items;
      }).catch(err => this.displayError(err))
    },
    addTask() {
      this.error = null;
      axios.post(URL, this.task).then(result => {
        this.task={description:''}
        this.getTasks()}
      ).catch(err => this.displayError(err))
    },
    editTask(id) {
      this.editingTask = true;
      const task = this.tableData.find((item) => item.id === id);
      this.task.id = task.id;
      this.task.description = task.description
    },
    updateTask() {
      this.error = null;
      axios.put(URL+'/'+this.task.id, {description:this.task.description}).then(result => {
        this.task={description:''}
        this.editingTask = false;
        this.getTasks()
      }).catch(err => this.displayError(err))
    },
    deleteTask(id) {
      this.error = null;
      axios.delete(URL+'/'+id).then(result => this.getTasks()).catch(err => this.displayError(err))
    },
    cancelEditingTask() {
        this.task={description:''}
        this.editingTask = false;
    },
    displayError(err) {
      console.log(err.response.data)
      if (err.response && err.response.data) {
        const data = err.response.data;
        this.error = data.errors.join('<br/>')
      }
      else {
        this.error = err
      }
    }
  }
};
</script>
<style></style>
