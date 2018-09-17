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
              v-if="scope.row.audio !== null"
              size="mini"
              icon="el-icon-loading"
              type="success"
              @click="playSound(scope.$index, scope.row)"></el-button>
            <el-button
              v-else
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
        var file = {
          name: event.target.files[0].name,
          path: event.target.files[0].path,
          key: null,
          audio: null
        }
        this.tableData.push(file)
        this.$refs.selectFile.value = null

        this.saveChanges()
      },
      playSound (index, row) {
        if (this.dialogVisible) return

        if (!this.active) {
          this.$message.error('The switch is off.')
          return
        }

        // if the sound is playing stop it
        if (this.tableData[index].audio !== null) {
          this.tableData[index].audio.pause()
          this.tableData[index].audio = null
          return
        }

        this.tableData[index].audio = new Audio(this.tableData[index].path)
        this.tableData[index].audio.onended = () => {
          this.tableData[index].audio = null
        }
        this.tableData[index].audio.play()
      },
      removeSound (index, row) {
        // if the sound is playing stop it
        if (this.tableData[index].audio !== null) {
          this.tableData[index].audio.pause()
          this.tableData[index].audio = null
        }

        // if the sound has a key, unregister it
        if (this.tableData[index].key !== null) {
          this.$electron.ipcRenderer.send('streamersb:unregister:shortcut', { accelerator: this.tableData[index].key })
        }
        this.tableData = this.tableData.filter((file, i) => i !== index)

        this.saveChanges()
      },
      editKey (index, row) {
        this.editingIndex = index
        this.dialogVisible = true
        if (this.tableData[index].key) this.$refs.keyPressed.innerHTML = this.tableData[index].key
      },
      saveEdit () {
        if (!isAccelerator(this.$refs.keyPressed.innerHTML)) {
          this.$message.error('Invalid combination.')
          this.cancelEdit()
          return
        }

        if (this.$refs.keyPressed.innerHTML.length === 1) {
          this.$message.error('A modifier is needed.')
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

        this.saveChanges()
      },
      cancelEdit () {
        this.dialogVisible = false
        this.$refs.keyPressed.innerHTML = 'Press a key'
      },
      saveChanges () {
        var sounds = this.tableData.map(sound => {
          return Object.assign({}, sound, {audio: null})
        })
        localStorage.setItem('sounds', JSON.stringify(sounds))
      }
    },
    created () {
      // load data
      var sounds = localStorage.getItem('sounds')
      if (sounds) {
        this.tableData = JSON.parse(sounds)
        for (let i in this.tableData) {
          if (this.tableData[i].key) {
            this.$electron.ipcRenderer.send('streamersb:register:shortcut', { accelerator: this.tableData[i].key, index: i })
          }
        }
      }
  
      window.addEventListener('keydown', (e) => {
        if (this.dialogVisible) {
          var accelerator = ''

          if (e.altKey) accelerator += 'Alt+'
          if (e.ctrlKey) accelerator += 'Ctrl+'
          if (e.shiftKey) accelerator += 'Shift+'

          if (e.keyCode !== 16 && e.keyCode !== 17 && e.keyCode !== 18) {
            if (e.key.length === 1) accelerator += e.key.toUpperCase()
            else if (e.key.includes('Arrow')) accelerator += e.key.replace('Arrow', '')
            else accelerator += e.key
          }

          this.$refs.keyPressed.innerHTML = accelerator
          e.preventDefault()
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
