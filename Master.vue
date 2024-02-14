<template>
     <div id="app">
    <!-- <RecursiveComponent :node="treeData" @removeNode="removeNode" /> -->
    <RecursiveComponent :node="treeData" @requestRemoveNode="removeNode" />
      <input type="file" ref="fileUpload" @change="handleFileUpload" multiple />
    </div>
  </template>
  
  <script>
  import RecursiveComponent from './RecursiveComponent.vue';
  
  export default {
    name: 'App',
    components: {
      RecursiveComponent
    },
    data() {
      return {
        showRootUpload: true, 
        forceUpdate: 0,
        treeData: {
          label: 'Root',
          type: 'folder', 
          children: [
          { id: this.generateUniqueId(), label: 'Child 1', type: 'folder', children: [] },
          // ...
        ]
        }
      };
    },
    methods: {
      updateTree() {
        // 在这里可以执行额外的操作，例如更新后台数据
        this.forceUpdate++;
      },
      addChild(node) {
    const newNode = { id: this.generateUniqueId(), label: 'New Node', children: [] };
    if (!node.children) {
      node.children = []; // 在 Vue 3 中，直接赋值即可
    }
    node.children.push(newNode);
    this.$emit('updateTree');
  },
  generateUniqueId() {
      return Date.now().toString(36) + Math.random().toString(36).substr(2);
  },
    removeNode(nodeToRemove, parentNode) {
      if (parentNode) {
        // 在父节点中删除
        const childIndex = parentNode.children.findIndex(child => child.id === nodeToRemove.id);
        if (childIndex > -1) {
          parentNode.children.splice(childIndex, 1);
        }
      } else {
        // 在根节点中删除
        const childIndex = this.treeData.children.findIndex(child => child.id === nodeToRemove.id);
        if (childIndex > -1) {
          this.treeData.children.splice(childIndex, 1);
        }
      }
      this.forceUpdate++;
      this.updateTree();
    },
    handleFileUpload(event, parentNode) {
  const files = event.target.files;
  // 确保parentNode有children数组
  if (!parentNode.children) {
    this.$set(parentNode, 'children', []);
  }
  for (const file of files) {
    const newNode = {
      id: this.generateUniqueId(),
      label: file.name,
      type: 'file',
      children: []
    };
    parentNode.children.push(newNode); // 将新节点添加到父节点的子节点列表中
  }
  this.$emit('updateTree');
},

  removeNodeRecursive(currentNode, nodeToRemove) {
    if (currentNode.children) {
    currentNode.children = currentNode.children.filter(child => child.id !== nodeToRemove.id);
    currentNode.children.forEach(child => this.removeNodeRecursive(child, nodeToRemove));
  }
  }
  }
  };
  </script>
  