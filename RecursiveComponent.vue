<template>
    <div class="node">
      <div>
      {{ node.label }}
      <input type="text" v-model="editableLabel" @blur="updateName(editableLabel, node)" />
      <button v-if="node.type === 'folder'" @click="addChild(node)">Add Child</button>
      <button @click="emitRemoveEvent(node)">Remove</button>
      <input v-if="node.type === 'folder'" type="file" @change="handleFileUpload($event, node)" multiple />
    </div>

      <ul v-if="node.children && node.children.length">
      <li v-for="(child, index) in node.children" :key="index">
        <span v-if="child.type === 'folder'">📁</span>
        <span v-else-if="child.type === 'file'">📄</span>
        {{ child.label }}
        <RecursiveComponent :node="child" :parentNode="node" />
      </li>
    </ul>
    </div>
  </template>
  
  <script>
  export default {
    name: 'RecursiveComponent',
    data() {
  return {
    forceUpdate: 0,
    editableLabel: this.node.label,
  };
},
    props: {
  node: {
    type: Object,
    required: true
  },
  parentNode: Object 
},
    methods: {
      addChild(node) {
    if (node.type === 'folder') {
      // 现在只有当节点类型为'folder'时才会添加子节点
      const newNode = {
        id: this.generateUniqueId(),
        label: 'New Folder',
        type: 'folder', // 设置新节点类型为'folder'
        children: []
      };
      if (!node.children) {
        this.$set(node, 'children', []);
      }
      node.children.push(newNode);
      this.$emit('updateTree');
    } else {
      alert('Cannot add a folder under a file');
    }
  },
  handleFileUpload(event, node) {
    const files = event.target.files;
    // 确保node有children数组
    if (!node.children) {
      this.$set(node, 'children', []);
    }
    for (const file of files) {
      const newNode = {
        id: this.generateUniqueId(),
        label: file.name,
        type: 'file',
        children: []
      };
      node.children.push(newNode);
    }
    this.$emit('updateTree');
  },
    // addFile(node) {
    //   const newNode = { id: this.generateUniqueId(), label: 'New File', children: [] };
    //   node.children.push(newNode);
    //   this.$emit('updateTree');
    // },
    emitRemoveEvent(node) {
      this.$emit('requestRemoveNode', node, this.parentNode || null);
    },
    generateUniqueId() {
      return Date.now().toString(36) + Math.random().toString(36).substr(2);
    },
 
    updateName(newName, node) {
      if (!newName.trim()) {
        alert("名称不能为空！");
        return;
      }
      // 检查名称是否重复
      const isDuplicate = this.findDuplicateName(newName, this.$parent.node);
      if (isDuplicate) {
        alert("已存在相同名称的节点！");
        return;
      }
      node.label = newName;
      this.$emit('updateTree'); // 如果有更新树的需要
    },
    findDuplicateName(name, parentNode) {
      if (!parentNode) {
        return this.treeData.children.some(child => child.label === name);
      }
      return parentNode.children.some(child => child.label === name);
    },
      updateTree() {
        this.forceUpdate++;
        // this.$emit('updateTree');
      }
    }
  };
  </script>
  
  <style scoped>
  .node {
    margin-left: 20px;
  }
  ul{
    list-style: none;
  }
  </style>