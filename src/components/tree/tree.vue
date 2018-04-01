<script>
import TTreeItem from './tree-item.vue'

export default {
  components: {
    TTreeItem
  },
  name: 't-tree',

  data () {
    return {
      treeData: [],
      labelKey: 'label',
      childrenKey: 'children'
    }
  },

  props: {
    data: Array,
    defaultProps: {
      default: () => {
        return {
          label: null,
          children: null
        }
      }
    },
    childIndent: {
      type: Number,
      default: 16
    },
    showCheckbox: Boolean,
    nodeClick: Function,
    checkChange: Function,
    load: Function,
    lazy: Boolean,
    nodeKey: String,
    expandNode: Array,
    checkedNode: Array
  },

  created () {
    this.$on('node-click', this.handleNodeClick)
    this.$on('check-change', this.handleCheckChange)
    this.$on('lazy-load', this.lazyLoad)

    this.setPropKeys()
    this.initData()
    this.formatData()
  },

  render (h) {
    let getItems = (h, parent) => {
      const _this = this
      let arr = parent ? parent.children : (this.treeData || [])
      let items = []
      arr.forEach(function (el) {
        let children = []
        if (el.children) {
          children = getItems(h, el)
        }

        let item = h(TTreeItem, {
          props: {
            label: el[_this.labelKey],
            indent: `${_this.childIndent * (el.nodeIndex.split('-').length - 1)}px`,
            hasChildren: !!el.children,
            showCheckbox: _this.showCheckbox,
            lazy: _this.lazy,
            nodeIndex: el.nodeIndex,
            initChecked: el.initChecked,
            initExpand: el.initExpand,
            checkDisabled: el.disabled
          }
        }, children)
        items.push(item)
      })
      return items
    }

    return h('div', {
      class: 't-tree'
    }, getItems(h))
  },

  methods: {
    initData () {
      if (this.lazy) {
        this.dynamicLoad()
      } else {
        this.treeData = this.data
      }
    },
    setPropKeys () {
      if (this.defaultProps['label']) this.labelKey = this.defaultProps['label']
      if (this.defaultProps['children']) this.childrenKey = this.defaultProps['children']
    },

    formatData (parent) {
      const _this = this
      const indexPrefix = parent ? `${parent.nodeIndex}-` : ''
      let data = parent ? parent.children : this.treeData
      data.forEach(function (el, idx) {
        let checked

        if (parent && parent.initChecked) {
          checked = parent.initChecked
        } else {
          checked = !!(_this.nodeKey && _this.checkedNode && _this.checkedNode.indexOf(el[_this.nodeKey]) !== -1)
        }

        el.nodeIndex = `${indexPrefix}${idx + 1}`
        el.initChecked = checked
        if (_this.nodeKey && _this.expandNode && _this.expandNode.indexOf(el[_this.nodeKey]) !== -1) {
          el.initExpand = true
        }
        if (el.children) _this.formatData(el)
      })

      return data
    },

    runLoad (node) {
      return new Promise(resolve => {
        setTimeout(() => {
          this.load(node, resolve)
        })
      })
    },

    async dynamicLoad (node) {
      let result = await this.runLoad(node)
      if (node) {
        this.treeData = this.appendNodeInTreeData(node, result)
        this.$forceUpdate()
      } else {
        this.treeData = result
      }
      return true
    },

    lazyLoad (node) {
      this.dynamicLoad(node).then(() => {
        node.loading = false
        node.loaded = true
      })
    },

    appendNodeInTreeData (node, children, parent) {
      const nodeIndex = node.nodeIndex
      let arr = parent ? parent.children : this.treeData
      const _this = this

      arr.some(function (el) {
        if (el.nodeIndex === nodeIndex) {
          el.initChecked = node.isChecked
          if (children) {
            el.children = children
            _this.formatData(el)
          }
          return true
        }

        if (el.children) {
          el.children = _this.appendNodeInTreeData(node, children, el)
          return true
        }
      })

      return arr
    },

    handleNodeClick (data) {
      this.nodeClick && this.nodeClick(data)
    },

    handleCheckChange (data) {
      this.checkChange && this.checkChange(data)
    }
  },

  watch: {
    defaultProps () {
      this.setPropKeys()
      this.formatData()
    },
    data (val) {
      !this.lazy && (this.treeData = val)
    },
    treeData () {
      this.formatData()
    }
  }
}
</script>

<style scoped>
</style>