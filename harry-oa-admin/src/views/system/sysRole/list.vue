<template>
  <div class="app-container">
    <!--查询表单-->
    <div class="search-div">
      <el-form label-width="70px" size="small">
        <el-row>
          <el-col :span="24">
            <el-form-item label="Role Name">
              <el-input
                style="width: 100%"
                v-model="searchObj.roleName"
                placeholder="Role Name"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row style="display: flex">
          <el-button
            type="primary"
            icon="el-icon-search"
            size="mini"
            :loading="loading"
            @click="fetchData()"
            >Search</el-button
          >
          <el-button icon="el-icon-refresh" size="mini" @click="resetData"
            >Reset</el-button
          >
        </el-row>
      </el-form>
    </div>
    <!-- tool bar -->

    <!-- role list -->
    <el-table
      v-loading="listLoading"
      :data="list"
      stripe
      border
      style="width: 100%; margin-top: 10px; margin-bottom: 10px"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" />

      <el-table-column label="Index" width="70" align="center">
        <template slot-scope="scope">
          {{ (page - 1) * limit + scope.$index + 1 }}
        </template>
      </el-table-column>

      <el-table-column prop="roleName" label="Role Name" />
      <el-table-column prop="roleCode" label="Role Code" />
      <el-table-column prop="createTime" label="Created Time" width="160" />
      <el-table-column label="Operation" width="200" align="center">
        <template slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="edit(scope.row.id)"
            title="Edit"
          />
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="removeDataById(scope.row.id)"
            title="Delete"
          />
        </template>
      </el-table-column>
    </el-table>
    <div class="tools-div">
      <el-button type="success" icon="el-icon-plus" size="mini" @click="add"
        >Add</el-button
      >
      <el-button class="btn-add" size="mini" @click="batchRemove()"
        >Batch Remove</el-button
      >
    </div>
    <!-- paginate -->
    <el-pagination
      :current-page="page"
      :total="total"
      :page-size="limit"
      style="padding: 30px 0; text-align: center"
      layout="total, prev, pager, next, jumper"
      @current-change="fetchData"
    />

    <!-- pop up window -->
    <el-dialog title="Add/edit" :visible.sync="dialogVisible" width="40%">
      <el-form
        ref="dataForm"
        :model="sysRole"
        label-width="150px"
        size="small"
        style="padding-right: 40px"
      >
        <el-form-item label="Role Name">
          <el-input v-model="sysRole.roleName" />
        </el-form-item>
        <el-form-item label="Role Index">
          <el-input v-model="sysRole.roleCode" />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button
          @click="dialogVisible = false"
          size="small"
          icon="el-icon-refresh-right"
          >Cancel</el-button
        >
        <el-button
          type="primary"
          icon="el-icon-check"
          @click="saveOrUpdate()"
          size="small"
          >Confirm</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import api from "@/api/system/sysRole";

export default {
  data() {
    return {
      list: [], //role list
      page: 1, //recent page
      limit: 3, //records in one page
      total: 0, //total records
      searchObj: {}, //condition object
      dialogVisible: false,
      sysRole: {},
      selections: [], //get selection value
    };
  },
  //before render
  created() {
    this.fetchData();
  },
  methods: {
    handleSelectionChange(selection) {
      this.selections = selection;
      console.log(this.selections);
    },

    batchRemove() {
      if (this.selections.length == 0) {
        this.$message.warning("Please select the records to be deleted");
        return;
      } else {
        this.$confirm(
          "This action will permanently delete the records, do you want to continue?",
          "Warning",
          {
            confirmButtonText: "Confirm",
            cancelButtonText: "Cancel",
            type: "warning",
          }
        )
          .then(() => {
            var idList = [];
            this.selections.forEach((item) => {
              var id = item.id;
              idList.push(id);
            });
            return api.batchRemove(idList);
          })
          .then((response) => {
            this.$message.success(response.message);
            this.fetchData();
          });
      }
    },

    edit(id) {
      this.dialogVisible = true;
      this.fetchDataById(id);
    },

    fetchDataById(id) {
      api.getById(id).then((response) => {
        this.sysRole = response.data;
      });
    },

    add() {
      this.dialogVisible = true;
    },
    saveOrUpdate() {
      if (!this.sysRole.id) {
        this.save();
      } else {
        this.update();
      }
    },
    save() {
      api.saveRole(this.sysRole).then((response) => {
        this.$message.success(response.message || "Successful");
        this.dialogVisible = false;
        this.fetchData(this.page);
      });
    },
    update() {
      api.updateById(this.sysRole).then((response) => {
        this.$message.success(response.message || "Successful");
        this.dialogVisible = false;
        this.fetchData(this.page);
      });
    },
    fetchData(current = 1) {
      this.page = current;
      api
        .getPageList(this.page, this.limit, this.searchObj)
        .then((response) => {
          this.list = response.data.records;
          this.total = response.data.total;
        });
    },
    //delete
    removeDataById(id) {
      this.$confirm(
        "This action will permanently delete the record, do you want to continue?",
        "Warning",
        {
          confirmButtonText: "Confirm",
          cancelButtonText: "Cancel",
          type: "warning",
        }
      )
        .then(() => {
          return api.removeById(id);
        })
        .then((response) => {
          this.fetchData(this.page);
          this.$message.success(response.message || "Delete successfully");
        });
    },
  },
};
</script>
