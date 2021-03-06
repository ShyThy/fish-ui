<template>
  <fish-tree-node
      :data="data"
      :multiple="multiple"
      :selected-key="selectedKey"
      :data-key-map="dataKeyMap"
      :expand="expand"
      :edited="edited"
      :delimiter="delimiter"
      :iconCaretRight="iconCaretRight"
      :iconCaretDown="iconCaretDown"
      :on-item-checked="onItemChecked"
      :on-item-dblclick="onItemDblclick"
      :on-item-click="onItemClick">
  </fish-tree-node>
</template>
<script>
  import FishTreeNode from './TreeNode.vue'

  export default {
    components: {FishTreeNode},
    name: 'fish-tree',
    props: {
      defaultSelectedKey: { type: String, default: '' },
      defaultCheckedKeys: { type: Array, default: () => [] },
      multiple: { type: Boolean, default: false },
      expand: { type: Boolean, default: false },
      data: { type: Array, required: true }, // [{title: '', key: '', children: '', checked: false}]
      edited: { type: Boolean, default: false },
      delimiter: { type: String, default: '-' },
      iconCaretRight: { type: String, default: 'fa fa-caret-right' },
      iconCaretDown: { type: String, default: 'fa fa-caret-down' }
    },
    data () {
      return {
        selectedKey: this.defaultSelectedKey,
        dataKeyMap: this.getResetDataKeyMap()
      }
    },
    watch: {
      defaultSelectedKey (nowVal, oldVal) {
        this.selectedKey = nowVal
      },
      defaultCheckedKeys (nowVal, oldVal) {
        this.dataKeyMap = this.getResetDataKeyMap()
      }
    },
    methods: {
      getResetDataKeyMap (checkedKeys) {
        const dataKeyMap = {}
        initDataKeysMap(dataKeyMap, new Set(this.defaultCheckedKeys), this.data)
        return dataKeyMap
      },
      onItemChecked (item) {
        if (this.dataKeyMap[item.key][0] === 'checked') {
          this.setAllChildrenState(item.key, '')
        } else {
          this.setAllChildrenState(item.key, 'checked')
        }
        this.setAllParentState(item.key)
        this.$emit('item-checked', this.getCheckedKeys())
      },
      onItemDblclick (item) {
        this.$emit('item-dblclick', item)
      },
      onItemClick (item) {
        this.selectedKey = item.key
        this.$emit('item-click', item)
      },
      getCheckedKeys () {
        // 获取选择的keys
        let checkedKeys = Object.keys(this.dataKeyMap).filter((key) => {
          return this.dataKeyMap[key][0] === 'checked'
        })
        return checkedKeys.filter((key, index) => {
          for (let k of checkedKeys) {
            if (k !== key && key.startsWith(k)) {
              return false
            }
          }
          return true
        })
      },
      setAllChildrenState (key, state) {
        // 设置所有子节点的状态
        this.dataKeyMap[key].splice(0, 1, state)
        this.dataKeyMap[key][1] && this.dataKeyMap[key][1].forEach((child) => {
          this.setAllChildrenState(child, state)
        })
      },
      setAllParentState (key) {
        // 设置所有父亲节点的状态
        let parentKey = key.substring(0, key.lastIndexOf(this.delimiter))
        if (parentKey === '') return
        if (this.dataKeyMap[parentKey] && this.dataKeyMap[parentKey][1]) {
          let parentAllChildrenChecked = this.dataKeyMap[parentKey][1].every((child) => {
            return this.dataKeyMap[child][0] === 'checked'
          })
          let parentAllChildrenOpen = this.dataKeyMap[parentKey][1].some((child) => {
            return this.dataKeyMap[child][0] === 'open' || this.dataKeyMap[child][0] === 'checked'
          })
          this.dataKeyMap[parentKey].splice(0, 1, parentAllChildrenChecked ? 'checked' : (parentAllChildrenOpen ? 'open' : ''))
        }
        this.setAllParentState(parentKey)
      }
    }
  }

  const getKeyState = (setKeys, key) => {
    if (setKeys.has(key)) return 'checked'
    for (let setKey of setKeys.keys()) {
      if (setKey.startsWith(key)) return 'open'
      if (key.startsWith(setKey)) return 'checked'
    }
    return ''
  }
  const initDataKeysMap = (dataKeysMap, setKeys, items) => {
    items && items.forEach((item) => {
      dataKeysMap[item.key] = [
        getKeyState(setKeys, item.key),
        item.children && item.children.map((child) => child.key)]
      initDataKeysMap(dataKeysMap, setKeys, item.children)
    })
  }
</script>