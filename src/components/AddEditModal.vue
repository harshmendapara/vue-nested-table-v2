<template>
  <transition name="fade">
    <div class="modal" v-if="show">
      <div class="modal__backdrop" @click="closeModal()"/>

      <div class="modal__dialog">
        <div class="modal__header">
          {{modalType === "edit" ? 'Edit Data' : 'Add New Data'}}
          <button type="button" class="mdi mdi-close" @click="closeModal()"/>
        </div>

        <div class="modal__body">
            <label>Name:*</label>
            <input label="Name" type="text" v-model="rowData.name" />
            <span v-if="errors && errors['name']" class="error">{{errors['name']}}</span>
            <br/>
            <label v-if="modalType != 'add'">Select Parent(if any): {{modalType}}</label>
            <select v-model="rowData.parentId" class="form-control" @change="setParent" v-if="modalType != 'add'">
                <option v-for="parent in parents" :key="parent.parentId" :value="parent.parentId">
                    {{ parent.parent }}
                </option>
            </select>
          <slot name="body"/>
        </div>

        <div class="modal__footer">
          <button @click="closeModal">Cancel</button>&nbsp;&nbsp;
          <button @click="submitData">{{modalType === "edit" ? 'Save' : 'Add'}}</button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  name: "AddEditModal",
  props: ['parents'],
  data() {
    return {
      show: false,
      rowData: null,
      modalType: 'add',
      errors: { name: ''}
    };
  },
  methods: {
    closeModal() {
      this.show = false;
      document.querySelector("body").classList.remove("overflow-hidden");
    },
    openModal(row = null, type) {
      if(type === 'add-child') {
        this.modalType = "add"
        this.rowData = {
          name: '', 
          parentId: row.id, 
          parent: null, 
          open: false, 
          childrenOpen: false
        }
      } else if(row) {
        this.modalType = "edit"
        this.rowData = JSON.parse(JSON.stringify(row));
      }
      else {
        this.modalType = "add"
        this.rowData = {
            name: '', 
            parentId: null, 
            parent: null, 
            open: true, 
            childrenOpen: false
        }
      }
      this.show = true;
      document.querySelector("body").classList.add("overflow-hidden");
    },
    setParent(){
      this.rowData.parent = this.parents.find((p) => p.parentId === this.rowData.parentId).parent;
      this.rowData.open = false;
    },
    submitData(){
      if(!this.rowData.name.length){
          this.errors['name'] = "This field is required";
          return;
      }
      this.$emit('addEditData', this.rowData);
    }
  }
};
</script>


<style lang="scss" scoped>
.modal {
  overflow-x: hidden;
  overflow-y: auto;
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 9;
  &__backdrop {
    background-color: rgba(0, 0, 0, 0.3);
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 1;
  }
  &__dialog {
    background-color: #ffffff;
    position: relative;
    width: 600px;
    margin: 50px auto;
    display: flex;
    flex-direction: column;
    border-radius: 5px;
    z-index: 2;
    @media screen and (max-width: 992px) {
      width: 90%;
    }
  }
  &__close {
    width: 30px;
    height: 30px;
  }
  &__header {
    padding: 20px 20px 10px;
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
  }
  &__body {
    padding: 10px 20px 10px;
    overflow: auto;
    display: flex;
    flex-direction: column;
    align-items: stretch;
    text-align: left;
  }
  &__footer {
    padding: 10px 20px 20px;
  }
  .error{
    color: red;
    font-size: 12px;
  }
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>