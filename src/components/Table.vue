<template>
  <div>
    <div v-if="loading">Loading ...</div>
    <div v-else class="container">
      <button type="button" @click="$refs.addEditModal.openModal(null, 'add-new')"> Add </button>
      <table border=1>
        <thead>
          <tr>
            <th @click="sort('id')">Id</th>
            <th @click="sort('name')">Name</th>
            <th @click="sort('parent')">Parent</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row) in displayTableData" :key="row.id">
            <td>{{row.id}}</td>
            <td :style="{'padding-left': (30 * getParentLevel(row.id))+'px'}">
              <span 
                v-if="hasChildren(row.id)" 
                :class="{ 'mdi-plus-box': !row.childrenOpen, 'mdi-minus-box': row.childrenOpen }"
                class="mdi mdi-24px" 
                @click="toggleChildren(row.id)"
              />{{ row.name}}
            </td>
            <td>{{row.parent}}</td>
            <td class="text-center">
              <span class="mdi mdi-plus" @click="$refs.addEditModal.openModal(row, 'add-child')" />&nbsp;
              <span class="mdi mdi-pencil" @click="$refs.addEditModal.openModal(row, 'edit-child')" />&nbsp;
              <span class="mdi mdi-delete" @click="deleteItem(row.id)"/></td>            
          </tr>
        </tbody>
      </table>
      <p>
        <button @click="prevPage">Previous</button>&nbsp;&nbsp;
        <button @click="nextPage">Next</button>
      </p>
    </div>
  <AddEditModal ref="addEditModal" @addEditData="addEditData" :parents="getParentsArray"/>
  </div>
</template>

<script>
import AddEditModal from "@/components/AddEditModal";
import _ from 'lodash';

  export default {
    name: "NestedTable",
    components: { AddEditModal },
    data() {
      return {
        loading: true,
        tableData: [],
        currentSort:'',
        currentSortDir:'asc',
        showAddEditModal: false,
        pageSize:20,
        currentPage:1
      };
    },
    created: function() {
      this.tableData = [
        {id: 1, name: 'Transport', parentId: null, parent: null, open: true, childrenOpen: false},
        {id: 2, name: 'Train', parentId: 1, parent: 'Transport', open: false, childrenOpen: false},
        {id: 3, name: 'Gujarat Express', parent: 'Train', parentId: 2, open: false, childrenOpen: false},
        {id: 4, name: 'Gujarat Express', parent: 'Train', parentId: 2, open: false, childrenOpen: false},
        {id: 5, name: 'Gujarat Express', parent: 'Train', parentId: 2, open: false, childrenOpen: false},
        {id: 6, name: 'Gujarat Express', parent: 'Train', parentId: 2, open: false, childrenOpen: false},
        {id: 12, name: 'Gujarat Express Child', parent: 'Gujarat Express', parentId: 6, open: false, childrenOpen: false},
        {id: 7, name: 'Train 1', parentId: 1, parent: 'Transport', open: false, childrenOpen: false},
        {id: 8, name: 'General',  parentId: null, parent: null, open: true, childrenOpen: false},
        {id: 9, name: 'Software', parentId: 8, parent: 'General', open: false, childrenOpen: false},
        {id: 10, name: 'MS Office', parentId: 9, parent: 'Software', open: false, childrenOpen: false},
        {id: 11, name: 'Hardware', parentId: 8, parent: 'General', open: false, childrenOpen: false},
        {id: 13, name: 'CPU', parentId: 11, parent: 'Hardware', open: false, childrenOpen: false}
      ];
      this.loading = false;
    },
    computed:{
      sortedTable() {
        let sortITems = this.tableData.filter((a) => !a.parentId)
        let sort = [];
        // eslint-disable-next-line
        let sorted = sortITems.sort((a,b) => {
          if(a.parentId || b.parentId) return;
          let modifier = 1;
          if(this.currentSortDir === 'desc') modifier = -1;
          if(a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
          if(a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
          return 0;
        });
        sorted.map((s) => {
          sort.push(s);
          if(this.hasChildren(s.id)) this.sortChildren(s.id, sort);    
        })
        return _.uniqBy(sort, 'id');
      },
      displayTableData(){
        return this.sortedTable.filter((t) => !t.parent || t.open)
                .filter((row, index) => {
                  let start = (this.currentPage-1)*this.pageSize;
                  let end = this.currentPage*this.pageSize;
                  if(index >= start && index < end) return true;
                });
      },
      getParentsArray(){
        let data = _.uniqBy(this.tableData, 'id');
        return data.map((d) => ({parent: d.name, parentId: d.id}))
      }
    },
    methods: {
      sort(s) {
        if(s === this.currentSort) {
          this.currentSortDir = this.currentSortDir==='asc'?'desc':'asc';
        }
        this.currentSort = s;
      },
      sortChildren(id, sort){
        let childrenItems = this.tableData.filter((a) => a.parentId === id);
        let sortedChildren = childrenItems.sort((a,b) => {
          let modifier = 1;
          if(this.currentSortDir === 'desc') modifier = -1;
          if(a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
          if(a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
          return 0;
        });
        sortedChildren.map((s) => {        
          sort.push(s);
          if(this.hasChildren(s.id)) sort.push(...this.sortChildren(s.id, sort));  
        })
        return sortedChildren;
      },
      nextPage() {
        if((this.currentPage*this.pageSize) < this.tableData.filter((t) => t.open).length) this.currentPage++;
      },
      prevPage() {
        if(this.currentPage > 1) this.currentPage--;
      },
      hasChildren(id){
        let children = this.tableData.filter((t) => t.parentId === id);
        return children.length > 0;
      },
      toggleChildren(id){
        let match = this.tableData.find((t) => t.id === id);
        match.childrenOpen = !match.childrenOpen;
        if(this.hasChildren(id) && !match.childrenOpen) this.closeNestedChildren(id);
        else{
          let children = this.tableData.filter((t) => t.parentId === id);
          children.map((ch) => ch.open = !ch.open);
        }
      },
      closeNestedChildren(id){
        let children = this.tableData.filter((t) => t.parentId === id);
        children.map((ch) => {
          ch.open = false
          if(this.hasChildren(ch.id)) {          
            ch.childrenOpen = false;
            this.closeNestedChildren(ch.id);
          }
        });
      },
      getParentLevel(id, level = 0){
        let tableItem = this.tableData.find((t) => t.id === id);
        if(tableItem.parentId > 0){        
          level += 1;  
          return this.getParentLevel(tableItem.parentId, level);
        } 
        else{   
          return level;
        }
      },
      addEditData(data){
        if(data.id){
          let matchIndex = this.tableData.findIndex((t) => t.id === data.id);
          this.tableData.splice(matchIndex, 1 ,data)
        }
        else{
          let id = Math.max(...this.tableData.map(row => row.id)) + 1
          data.id = id;
          if(data.parentId){
            data.open = this.tableData.find((t) => t.id === data.parentId).childrenOpen;
            let matchIndex = this.tableData.findIndex((t) => t.id === data.parentId);
            this.tableData.splice((matchIndex + 1), 0, data)
          }
          else this.tableData.push(data);
        }
        this.$refs.addEditModal.closeModal()
        // eslint-disable-next-line no-console
        console.log(this.tableData)
      },
      deleteItem(id){
        let matchIndex = this.tableData.findIndex((t) => t.id === id);
        if(confirm("Do you really want to delete?")){
          if(matchIndex > -1) this.tableData.splice(matchIndex, 1);
          if(this.hasChildren(id)) this.deleteChildren(id);
        }
      },
      deleteChildren(id){
        let children = this.tableData.filter((t) => t.parentId === id);
        children.map((ch) => {
          let matchIndex = this.tableData.findIndex((t) => t.id === ch.id);
          if(matchIndex > -1) this.tableData.splice(matchIndex, 1);
          if(this.hasChildren(ch.id)) this.deleteChildren(ch.id);
        });
      }
    }
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.container {
  margin: 50px 200px;
  display: flex;
  justify-content: center;
  flex-direction: column;
  table{
    flex:0.5;
    tr td{
      text-align: left;
    }
    tr td.text-center{
      text-align: center;
    }
  }
  button{
    margin: 15px 0 15px auto;
    flex:0.25
  }
}
</style>
