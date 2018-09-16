<template>
  <el-container>
    <el-header>
      <el-row>
        <el-col :span="12">
          <input type="file" ref="selectFile" @change="addSound" accept="audio/*" style="display: none;" />
          <el-button type="primary" icon="el-icon-circle-plus" @click="$refs.selectFile.click()">Sound</el-button>
        </el-col>
        <el-col :span="12" class="header-right">
          <el-switch v-model="active" inactive-text="OFF" active-text="ON"></el-switch>
        </el-col>
      </el-row>
    </el-header>
    
    <el-main>
      <el-table :data="tableData" height="450">
        <el-table-column prop="name" label="Name" fixed></el-table-column>
        <el-table-column label="Key binding">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              @click="editKey(scope.$index, scope.row)">{{ scope.row.key ? scope.row.key : 'Click to edit' }}</el-button>
          </template>
        </el-table-column>
        <el-table-column
          label="Actions" width="160">
          <template slot-scope="scope">
            <el-button
              size="mini"
              icon="el-icon-caret-right"
              @click="playSound(scope.$index, scope.row)"></el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeSound(scope.$index, scope.row)"></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-main>

    <el-dialog
      title="Edit key binding"
      :visible.sync="dialogVisible"
      width="40%"
      style="text-align: center">
      <span ref="keyPressed">Press a key</span>
      <p style="font-size: 12px">To use an existing shortcut you have to unbind it first</p>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cancelEdit">Cancel</el-button>
        <el-button type="primary" @click="saveEdit">Confirm</el-button>
      </span>
    </el-dialog>
  </el-container>
</template>

<script>
  import isAccelerator from 'electron-is-accelerator'

  export default {
    name: 'landing-page',
    data: function () {
      return {
        active: true,
        tableData: [],

        // edit key
        editingIndex: null,
        dialogVisible: false
      }
    },
    methods: {
      addSound (event) {
        var file = { name: event.target.files[0].name, path: event.target.files[0].path, key: null }
        this.tableData.push(file)
        this.$refs.selectFile.value = null
      },
      playSound (index, row) {
        if (this.dialogVisible || !this.active) return

        var audio = new Audio(this.tableData[index].path)
        audio.play()
      },
      removeSound (index, row) {
        this.$electron.ipcRenderer.send('streamersb:unregister:shortcut', { accelerator: this.tableData[index].key })
        this.tableData = this.tableData.filter((file, i) => i !== index)
      },
      editKey (index, row) {
        this.editingIndex = index
        this.dialogVisible = true
        if (this.tableData[index].key) this.$refs.keyPressed.innerHTML = this.tableData[index].key
      },
      saveEdit () {
        if (!isAccelerator(this.$refs.keyPressed.innerHTML)) {
          this.cancelEdit()
          return
        }

        // if another sound has the same key, unregister it
        for (let i in this.tableData) {
          if (this.tableData[i].key === this.$refs.keyPressed.innerHTML) {
            this.$electron.ipcRenderer.send('streamersb:unregister:shortcut', { accelerator: this.tableData[i].key })
            this.tableData[i].key = null
            break
          }
        }

        // if the current sound already has a key, unregister it
        if (this.tableData[this.editingIndex].key) {
          this.$electron.ipcRenderer.send('streamersb:unregister:shortcut', { accelerator: this.tableData[this.editingIndex].key })
        }

        this.dialogVisible = false
        this.tableData[this.editingIndex].key = this.$refs.keyPressed.innerHTML
        this.$electron.ipcRenderer.send('streamersb:register:shortcut', { accelerator: this.$refs.keyPressed.innerHTML, index: this.editingIndex })
        this.$refs.keyPressed.innerHTML = 'Press a key'
      },
      cancelEdit () {
        this.dialogVisible = false
        this.$refs.keyPressed.innerHTML = 'Press a key'
      }
    },
    created () {
      window.addEventListener('keydown', (e) => {
        if (this.dialogVisible) {
          var text = ''

          if (e.ctrlKey) text += 'Ctrl+'
          if (e.shiftKey) text += 'Shift+'
          if (e.altKey) text += 'Alt+'
          if (e.key !== 'Control' && e.key !== 'Shift' && e.key !== 'Alt') text += e.key

          this.$refs.keyPressed.innerHTML = text
        }
      })

      this.$electron.ipcRenderer.on('streamersb:play:sound', (event, arg) => {
        this.playSound(arg, null)
      })
    }
  }
</script>

<style>
  .el-header {
    color: #333;
    font-size: 12px;
    line-height: 60px;
  }

  .header-right {
    text-align: right;
  }
  
  .el-main {
    color: #333;
    background-color: #F5F5F5;
  }
</style>
