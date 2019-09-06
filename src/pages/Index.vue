<template>
  <q-page class="flex justify-center full-width">
    <div class="row full-width">
      <div class="col-10 q-card">
        <VueNodesContainer
          @contextmenu.native="showContextMenu"
          @click.native="closeContextMenu"
          ref="container"
          :MMNodes="MMNodes"
          :diagram.sync="diagram"
          @nodeSelect="selectNode"
          @nodeDeselect="deselectNode"
          class="container"
        />

        <ul
          id="contextMenu"
          ref="contextMenu"
          tabindex="-1"
          v-show="contextMenu.isShow"
          @blur="closeContextMenu"
          :style="{
            top: contextMenu.top + 'px',
            left: contextMenu.left + 'px'
          }"
        >
          <template v-for="type in selectMMNodes">
            <li v-bind:key="type.index" class="label">{{ type }}</li>
            <li
              v-for="node in filteredMMNodes(type)"
              v-bind:key="node.index"
              @click="addNodeContextMenu(node.name)"
            >
              {{ node.title || node.name }}
            </li>
          </template>
        </ul>
      </div>
      <div class="col-2 container-1 q-card">
        <q-list class="container-2" separator>
          <q-expansion-item
            expand-separator
            icon="adjust"
            label="Nodes"
            caption="Click to add"
            default-opened
          >
            <q-card>
              <q-card-section>
                <q-btn no-caps v-bind:key="node.name" v-for="node in selectableMMNodes" :label="node.label || node.value" @click.stop="addNode(node.value)"></q-btn>
              </q-card-section>
            </q-card>
          </q-expansion-item>

          <q-expansion-item
            expand-separator
            icon="settings"
            label="Property"
            caption="Click on element to display its properties"
            default-opened
          >
            <q-card>
              <q-card-section>
                <VueNodeProperty
                  :property="selectedNodeProperty"
                  @save="saveProperty"
                />
              </q-card-section>
            </q-card>
          </q-expansion-item>

          <q-expansion-item
            expand-separator
            icon="text_fields"
            label="Generated object"
          >
            <q-card>
              <q-card-section>
                <pre>
              {{ this.contextMenu.mouseX + ':' + this.contextMenu.mouseY }}
            </pre
                >
                <pre>
              {{ diagram }}
            </pre
                >
              </q-card-section>
            </q-card>
          </q-expansion-item>
        </q-list>
      </div>
    </div>
  </q-page>
</template>

<script>
import merge from 'deepmerge'

import VueNodesContainer from 'components/VueNodesContainer'
import VueNodeProperty from 'components/VueNodeProperty'
import domHelper from '../helpers/dom'
import MMNodes from './mm.json'
import diagram from './diagram.json'

export default {
  name: 'App',
  components: {
    VueNodesContainer,
    VueNodeProperty
  },
  data: function () {
    return {
      MMNodes: MMNodes,
      diagram: diagram,
      selectedNode: null,
      selectedType: 'delay',
      useContextMenu: true,
      contextMenu: {
        isShow: false,
        mouseX: 0,
        mouseY: 0,
        top: 0,
        left: 0
      }
    }
  },
  computed: {
    selectedNodeProperty () {
      if (!this.selectedNode || !this.selectedNode.values || !this.selectedNode.values.property) {
        return null
      }
      return this.selectedNode.values.property
    },
    selectableMMNodes () {
      return this.MMNodes.map(function (item) { return { value: item.name, label: item.title ? item.title : item.name } })
    },
    selectMMNodes () {
      return this.MMNodes.map(b => {
        return b.family
      }).filter((value, index, array) => {
        return array.indexOf(value) === index
      })
    }
  },
  methods: {
    selectNode (node) {
      console.log('select', node)
      this.selectedNode = node
    },
    deselectNode (node) {
      console.log('deselect', node)
      this.selectedNode = null
    },
    filteredMMNodes (type) {
      return this.MMNodes.filter(value => {
        return value.family === type
      })
    },
    addNode (type) {
      this.$refs.container.addNewNode(type, 100, 100)
    },
    saveProperty (val) {
      console.log(val)

      let diagram = this.diagram
      let node = diagram.nodes.find(b => {
        return b.id === this.selectedNode.id
      })
      node.values.property = val

      this.diagram = merge({}, diagram)
    },
    showContextMenu (e) {
      if (!this.useContextMenu) return
      if (e.preventDefault) e.preventDefault()

      this.contextMenu.isShow = true
      this.contextMenu.mouseX = e.x
      this.contextMenu.mouseY = e.y

      this.$nextTick(function () {
        this.setMenu(e.y, e.x)
        this.$refs.contextMenu.focus()
      })
    },
    setMenu (top, left) {
      let border = 5
      let contextMenuEl = this.$refs.contextMenu
      let containerElRect = this.$refs.container.$el.getBoundingClientRect()
      let largestWidth = containerElRect.right - contextMenuEl.offsetWidth - border
      let largestHeight = containerElRect.bottom - contextMenuEl.offsetHeight - border

      console.log(this.$refs.container)
      console.log(containerElRect)

      if (left > largestWidth) left = largestWidth
      if (top > largestHeight) top = largestHeight

      this.contextMenu.top = top
      this.contextMenu.left = left
    },
    addNodeContextMenu (name) {
      let offset = domHelper.getOffsetRect(this.$refs.container.$el)

      let x = this.contextMenu.mouseX - offset.left
      let y = this.contextMenu.mouseY - offset.top

      this.$refs.container.addNewNode(name, x, y)
      this.closeContextMenu()
    },
    closeContextMenu () {
      this.contextMenu.isShow = false
    }
  },
  watch: {
    nodes (newValue) {
      // console.log('nodes', JSON.stringify(newValue))
    },
    diagram (newValue) {
      // console.log('diagram', JSON.stringify(newValue))
    }
  }
}
</script>

<style lang="less">
html,
body {
  margin: 0;
  padding: 0;
}

html {
  width: 100vw;
  height: 100vh;
}

body {
  width: 100%;
  height: 100%;
}

#app {
  width: ~'calc(100% - 40px)';
  height: ~'calc(100% - 40px)';
  padding: 20px 0 0 20px;
}

.container {
  width: 100%;
  height: ~'calc(100% - 50px)';
  border: 1px solid rgb(226, 226, 226);
}

#contextMenu {
  position: absolute;
  z-index: 1000;
  background: white;
  border: 1px solid black;
  padding: 5px;
  margin: 0;

  li {
    &.label {
      color: gray;
      font-size: 90%;
    }
    list-style: none;
  }

  &:focus {
    outline: none;
  }
}
</style>
