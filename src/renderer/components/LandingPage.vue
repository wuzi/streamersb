<template>
  <el-container>
    <el-header>
      <el-row>
        <el-col :span="12">
          <el-button type="primary" icon="el-icon-circle-plus">Sound</el-button>
        </el-col>
        <el-col :span="12" class="header-right">
          <el-switch v-model="active" inactive-text="OFF" active-text="ON"></el-switch>
        </el-col>
      </el-row>
    </el-header>
    
    <el-main>
      <el-table :data="tableData">
        <el-table-column prop="name" label="Name"></el-table-column>
        <el-table-column prop="key" label="Key binding"></el-table-column>
        <el-table-column prop="actions" label="Actions" width="160"></el-table-column>
      </el-table>
    </el-main>
  </el-container>
</template>

<script>
  export default {
    name: 'landing-page',
    data: function () {
      const item = {
        name: 'Intense music',
        key: 'F3',
        actions: 'N/A'
      }
      return {
        active: true,
        tableData: Array(20).fill(item)
      }
    }
  }
</script>

<style>
  .el-container {
    height: 100vh;
  }

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
