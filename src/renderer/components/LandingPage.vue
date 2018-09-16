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
      <el-table :data="tableData" height="450">
        <el-table-column prop="name" label="Name" fixed></el-table-column>
        <el-table-column label="Key binding">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary">{{ scope.row.key }}</el-button>
          </template>
        </el-table-column>
        <el-table-column
          label="Actions" width="160">
          <template slot-scope="scope">
            <el-button
              size="mini"
              icon="el-icon-caret-right"></el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"></el-button>
          </template>
        </el-table-column>
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
        key: 'F3'
      }
      return {
        active: true,
        tableData: Array(20).fill(item)
      }
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
