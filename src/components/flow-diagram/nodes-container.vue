<template>
  <div class="container">
    <links @select="linkSelect($event)" :lines="lines" />
    <node
      v-for="node in nodes"
      :key="node.id"
      v-bind.sync="node"
      :options="optionsForChild"
      @update="updateDiagram"
      @linkingStart="linkingStart(node, $event)"
      @linkingStop="linkingStop(node, $event)"
      @linkingBreak="linkingBreak(node, $event)"
      @select="elementSelect(node)"
      @delete="nodeDelete(node)"
    />
  </div>
</template>

<script>
import merge from 'deepmerge'
import mouseHelper from '../../helpers/mouse'

import Node from './node'
import Links from './links'

export default {
  name: 'NodeContainer',
  components: {
    Node,
    Links
  },
  props: {
    MMNodes: {
      type: Array,
      default () {
        return []
      }
    },
    diagram: {
      type: Object,
      default () {
        return { nodes: [], links: [], container: {} }
      }
    },
    options: {
      type: Object
    }
  },
  mounted () {
    document.documentElement.addEventListener('mousemove', this.handleMove, true)
    document.documentElement.addEventListener('mousedown', this.handleDown, true)
    document.documentElement.addEventListener('mouseup', this.handleUp, true)
    document.documentElement.addEventListener('wheel', this.handleWheel, true)

    this.centerX = this.$el.clientWidth / 2
    this.centerY = this.$el.clientHeight / 2

    this.importMetaModel()
    this.importDiagram()
  },
  beforeDestroy () {
    document.documentElement.removeEventListener('mousemove', this.handleMove, true)
    document.documentElement.removeEventListener('mousedown', this.handleDown, true)
    document.documentElement.removeEventListener('mouseup', this.handleUp, true)
    document.documentElement.removeEventListener('wheel', this.handleWheel, true)
  },
  created () {
    this.mouseX = 0
    this.mouseY = 0

    this.lastMouseX = 0
    this.lastMouseY = 0

    this.minScale = 0.2
    this.maxScale = 5

    this.linking = false
    this.linkStartData = null

    this.inputSlotClassName = 'inputSlot'

    this.defaultDiagram = {
      nodes: [],
      links: [],
      container: {}
    }
  },
  data () {
    return {
      dragging: false,
      //
      centerX: 0,
      centerY: 0,
      scale: 1,
      //
      internalMMNodes: [],
      nodes: [],
      links: [],
      //
      tempLink: null,
      selectedElement: null,
      hasDragged: false
    }
  },
  computed: {
    optionsForChild () {
      return {
        width: 200,
        titleHeight: 20,
        scale: this.scale,
        inputSlotClassName: this.inputSlotClassName,
        center: {
          x: this.centerX,
          y: this.centerY
        }
      }
    },
    container () {
      return {
        centerX: this.centerX,
        centerY: this.centerY,
        scale: this.scale
      }
    },
    // Links calculate
    lines () {
      let lines = []

      for (let link of this.links) {
        let originNode = this.nodes.find(node => {
          return node.id === link.originID
        })

        let targetNode = this.nodes.find(node => {
          return node.id === link.targetID
        })

        if (!originNode || !targetNode) {
          console.log('Remove invalid link', link)
          this.removeLink(link.id)
          continue
        }

        if (originNode.id === targetNode.id) {
          console.log('Loop detected, remove link', link)
          this.removeLink(link.id)
          continue
        }

        let originLinkPos = this.getConnectionPos(originNode, link.originSlot, false)
        let targetLinkPos = this.getConnectionPos(targetNode, link.targetSlot, true)

        if (!originLinkPos || !targetLinkPos) {
          console.log('Remove invalid link (slot not exist)', link)
          this.removeLink(link.id)
          continue
        }

        let x1 = originLinkPos.x
        let y1 = originLinkPos.y

        let x2 = targetLinkPos.x
        let y2 = targetLinkPos.y

        lines.push({
          id: link.id,
          x1: x1,
          y1: y1,
          x2: x2,
          y2: y2,
          style: {
            stroke: '#F85',
            strokeWidth: 4 * this.scale,
            fill: 'none'
          },
          outlineStyle: {
            stroke: '#666',
            strokeWidth: 6 * this.scale,
            strokeOpacity: 0.6,
            fill: 'none'
          }
        })
      }

      if (this.tempLink) {
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        this.tempLink.style = {
          stroke: '#8f8f8f',
          strokeWidth: 4 * this.scale,
          fill: 'none'
        }

        lines.push(this.tempLink)
      }

      return lines
    }
  },
  methods: {
    // Events
    /** @param e {MouseEvent} */
    handleMove (e) {
      let mouse = mouseHelper.getMousePosition(this.$el, e)
      this.mouseX = mouse.x
      this.mouseY = mouse.y

      if (this.dragging) {
        let diffX = this.mouseX - this.lastMouseX
        let diffY = this.mouseY - this.lastMouseY

        this.lastMouseX = this.mouseX
        this.lastMouseY = this.mouseY

        this.centerX += diffX
        this.centerY += diffY

        this.hasDragged = true
      }

      if (this.linking && this.linkStartData) {
        let linkStartPos = this.getConnectionPos(this.linkStartData.node, this.linkStartData.slotNumber, false)
        this.tempLink = {
          x1: linkStartPos.x,
          y1: linkStartPos.y,
          x2: this.mouseX,
          y2: this.mouseY
        }
      }
    },
    handleDown (e) {
      const target = e.target || e.srcElement
      if ((target === this.$el || target.matches('svg, svg *')) && e.which === 1) {
        this.dragging = true

        let mouse = mouseHelper.getMousePosition(this.$el, e)
        this.mouseX = mouse.x
        this.mouseY = mouse.y

        this.lastMouseX = this.mouseX
        this.lastMouseY = this.mouseY

        this.deselectAll()
        if (e.preventDefault) e.preventDefault()
      }
    },
    handleUp (e) {
      const target = e.target || e.srcElement

      if (this.dragging) {
        this.dragging = false

        if (this.hasDragged) {
          this.updateDiagram()
          this.hasDragged = false
        }
      }

      if (this.$el.contains(target) && (typeof target.className !== 'string' || target.className.indexOf(this.inputSlotClassName) === -1)) {
        this.linking = false
        this.tempLink = null
        this.linkStartData = null
      }
    },
    handleWheel (e) {
      const target = e.target || e.srcElement
      if (this.$el.contains(target)) {
        if (e.preventDefault) e.preventDefault()

        let deltaScale = Math.pow(1.1, e.deltaY * -0.01)
        this.scale *= deltaScale

        if (this.scale < this.minScale) {
          this.scale = this.minScale
          return
        } else if (this.scale > this.maxScale) {
          this.scale = this.maxScale
          return
        }

        let zoomingCenter = {
          x: this.mouseX,
          y: this.mouseY
        }

        let deltaOffsetX = (zoomingCenter.x - this.centerX) * (deltaScale - 1)
        let deltaOffsetY = (zoomingCenter.y - this.centerY) * (deltaScale - 1)

        this.centerX -= deltaOffsetX
        this.centerY -= deltaOffsetY

        this.updateDiagram()
      }
    },
    // Processing
    getConnectionPos (node, slotNumber, isInput) {
      if (!node || slotNumber === -1) {
        return undefined
      }

      let x = 0
      let y = 0

      x += node.x
      y += node.y

      y += this.optionsForChild.titleHeight

      if (isInput && node.inputs.length > slotNumber) {
      } else if (!isInput && node.outputs.length > slotNumber) {
        x += this.optionsForChild.width
      } else {
        console.error('slot ' + slotNumber + ' not found, is input: ' + isInput, node)
        return undefined
      }

      // (height / 2 + nodeBorder + padding)
      y += (16 / 2 + 1 + 2)
      //  + (height * slotNumber)
      y += (16 * slotNumber)

      x *= this.scale
      y *= this.scale

      x += this.centerX
      y += this.centerY

      return { x: x, y: y }
    },
    // Linking
    linkingStart (node, slotNumber) {
      this.linkStartData = { node: node, slotNumber: slotNumber }
      let linkStartPos = this.getConnectionPos(this.linkStartData.node, this.linkStartData.slotNumber, false)
      this.tempLink = {
        x1: linkStartPos.x,
        y1: linkStartPos.y,
        x2: this.mouseX,
        y2: this.mouseY
      }

      this.linking = true
    },
    linkingStop (targetNode, slotNumber) {
      if (this.linkStartData && targetNode && slotNumber > -1) {
        /*
        // this prevents to have several links into one input slot
        this.links = this.links.filter(value => {
          return !(value.targetID === targetNode.id && value.targetSlot === slotNumber)
        }) */

        let maxID = Math.max(0, ...this.links.map(function (o) {
          return o.id
        }))

        // skip if looping
        if (this.linkStartData.node.id !== targetNode.id) {
          this.links.push({
            id: maxID + 1,
            originID: this.linkStartData.node.id,
            originSlot: this.linkStartData.slotNumber,
            targetID: targetNode.id,
            targetSlot: slotNumber
          })
          this.updateDiagram()
        }
      }

      this.linking = false
      this.tempLink = null
      this.linkStartData = null
    },
    linkingBreak (targetNode, slotNumber) {
      if (targetNode && slotNumber > -1) {
        let findLink = this.links.find(value => {
          return value.targetID === targetNode.id && value.targetSlot === slotNumber
        })

        if (findLink) {
          let findNode = this.nodes.find(value => {
            return value.id === findLink.originID
          })

          this.links = this.links.filter(value => {
            return !(value.targetID === targetNode.id && value.targetSlot === slotNumber)
          })

          this.linkingStart(findNode, findLink.originSlot)

          this.updateDiagram()
        }
      }
    },
    removeLink (linkID) {
      this.links = this.links.filter(value => {
        return !(value.id === linkID)
      })
    },
    // Nodes
    addNewNode (MMNodeName, x, y) {
      let maxID = Math.max(0, ...this.nodes.map(function (o) {
        return o.id
      }))

      let MMNode = this.internalMMNodes.find(n => {
        return n.type === MMNodeName
      })

      if (!MMNode) {
        return
      }
      let node = this.createNode(MMNode, maxID + 1)

      // if x or y not set, place node to center
      if (x === undefined || y === undefined) {
        x = (this.$el.clientWidth / 2 - this.centerX) / this.scale
        y = (this.$el.clientHeight / 2 - this.centerY) / this.scale
      } else {
        x = (x - this.centerX) / this.scale
        y = (y - this.centerY) / this.scale
      }

      node.x = x
      node.y = y
      this.nodes.push(node)

      this.updateDiagram()
    },
    createNode (MMNode, id) {
      let inputs = []
      let outputs = []
      let values = {}

      MMNode.fields.forEach(field => {
        if (field.attr === 'input') {
          inputs.push({
            name: field.name,
            label: field.label || field.name
          })
        } else if (field.attr === 'output') {
          outputs.push({
            name: field.name,
            label: field.label || field.name
          })
        } else {
          if (!values[field.attr]) {
            values[field.attr] = {}
          }

          let newField = merge({}, field)
          delete newField['name']
          delete newField['attr']

          if (!values[field.attr][field.name]) {
            values[field.attr][field.name] = {}
          }

          values[field.attr][field.name] = newField
        }
      })

      return {
        id: id,
        x: 0,
        y: 0,
        selected: false,
        type: MMNode.type,
        title: MMNode.title || MMNode.type,
        icon: MMNode.icon,
        inputs: inputs,
        outputs: outputs,
        values: values
      }
    },
    deselectAll (withoutID = null) {
      this.nodes.forEach((value) => {
        if (value.id !== withoutID && value.selected) {
          this.nodeDeselect(value)
        }
      })
    },
    // Events
    elementSelect (element) {
      element.selected = true
      this.selectedElement = element
      this.deselectAll(element.id)
      this.$emit('elementSelect', element)
    },
    linkSelect (linkId) {
      for (let index = 0; index < this.links.length; index++) {
        const element = this.links[index]
        if (element.id === linkId) {
          this.elementSelect(element)
          break
        }
      }
    },
    nodeDeselect (node) {
      node.selected = false

      if (node &&
        this.selectedElement &&
        this.selectedElement.id === node.id
      ) {
        this.selectedElement = null
      }

      this.$emit('nodeDeselect', node)
    },
    nodeDelete (node) {
      if (node.selected) {
        this.nodeDeselect(node)
      }
      this.links.forEach(l => {
        if (l.originID === node.id || l.targetID === node.id) {
          this.removeLink(l.id)
        }
      })
      this.nodes = this.nodes.filter(b => {
        return b.id !== node.id
      })
      this.updateDiagram()
    },
    //
    prepareNodes (nodes) {
      return nodes.map(node => {
        let MMNode = this.internalMMNodes.find(n => {
          return n.type === node.type
        })

        if (!MMNode) {
          return null
        }

        let newNode = this.createNode(MMNode, node.id)

        newNode = merge(newNode, node, {
          arrayMerge: (d, s) => {
            return s.length === 0 ? d : s
          }
        })

        return newNode
      }).filter(b => {
        return !!b
      })
    },
    prepareNodesLinking (nodes, links) {
      if (!nodes) {
        return []
      }

      let newNodes = []

      nodes.forEach(node => {
        let inputs = links.filter(link => {
          return link.targetID === node.id
        })

        let outputs = links.filter(link => {
          return link.originID === node.id
        })

        node.inputs.forEach((s, index) => {
          // is linked
          node.inputs[index].active = inputs.some(i => i.targetSlot === index)
        })

        node.outputs.forEach((s, index) => {
          // is linked
          node.outputs[index].active = outputs.some(i => i.originSlot === index)
        })

        newNodes.push(node)
      })

      return newNodes
    },
    importMetaModel () {
      if (this.MMNodes) {
        this.internalMMNodes = merge([], this.MMNodes)
      }
    },
    importDiagram () {
      if (this.diagram === null) {
        return
      }
      let diagram = merge(this.defaultDiagram, this.diagram)

      let nodes = this.prepareNodes(diagram.nodes)
      nodes = this.prepareNodesLinking(nodes, diagram.links)

      // set last selected after update nodes from props
      if (this.selectedElement) {
        let node = nodes.find(b => this.selectedElement.id === b.id)
        if (node) {
          node.selected = true
        }
      }

      this.nodes = nodes
      this.links = merge([], diagram.links)

      let container = diagram.container
      if (container.centerX) {
        this.centerX = container.centerX
      }
      if (container.centerY) {
        this.centerY = container.centerY
      }
      if (container.scale) {
        this.scale = container.scale
      }
    },
    exportDiagram () {
      let clonedNodes = merge([], this.nodes)
      let nodes = clonedNodes.map(value => {
        delete value['inputs']
        delete value['outputs']
        delete value['selected']

        return value
      })

      return {
        nodes: nodes,
        links: this.links,
        container: this.container
      }
    },
    updateDiagram () {
      this.$emit('update:diagram', this.exportDiagram())
    }
  },
  watch: {
    MMNodes () {
      this.importMetaModel()
    },
    diagram () {
      this.importDiagram()
    }
  }
}
</script>

<style lang="less" scoped>
.container {
  position: relative;
  overflow: hidden;
  box-sizing: border-box;
}
</style>
