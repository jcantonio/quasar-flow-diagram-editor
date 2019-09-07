<template>
    <div class="row full-width">
      <div class="col-10 q-card">
        <vue-nodes-container
          ref="container"
          :MMNodes="MMNodes"
          :diagram.sync="internalDiagram"
          @elementSelect="selectElement"
          @nodeDeselect="deselectElement"
          class="container"
        />
      </div>
      <div class="col-2 container-1 q-card">
        <q-list class="container-2" separator>
          <q-expansion-item
            expand-separator
            icon="category"
            label="Nodes"
            caption="Click to add"
            default-opened
          >
            <q-card>
              <q-card-section>
                <q-btn
                  :icon="node.icon || 'adjust'"
                  no-caps
                  v-bind:key="node.type"
                  v-for="node in MMNodes"
                  :label="node.title || node.type"
                  @click.stop="addNode(node.type)"
                >
                  <q-tooltip
                    v-if="node.description"
                    :delay="1000"
                    :offset="[0, 10]"
                    >{{ node.description }}</q-tooltip
                  >
                </q-btn>
              </q-card-section>
            </q-card>
          </q-expansion-item>

          <q-expansion-item
            expand-separator
            icon="settings"
            label="Properties"
            caption="Click on element to display its properties"
            default-opened
          >
            <q-card>
              <q-card-section>
                <component
                  :selected="selectedElement!=null"
                  :title="selectedElementTitle"
                  @save-title="saveTitle"
                  :properties="selectedElementProperties"
                  @save="saveProperties"
                  v-bind:is="componentPropertiesType"
                />
              </q-card-section>
            </q-card>
          </q-expansion-item>
        </q-list>
      </div>
    </div>
</template>

<script>
import merge from 'deepmerge'

import VueNodesContainer from 'components/VueNodesContainer'
import VueNodeProperties from 'components/VueNodeProperties'
import VueLinkProperties from 'components/VueLinkProperties'

export default {
  name: 'vue-flow-designer',
  components: {
    VueNodesContainer,
    VueNodeProperties,
    VueLinkProperties
  },
  data: function () {
    return {
      internalDiagram: null,
      selectedElement: null
    }
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
    }
  },
  mounted () {
    this.internalDiagram = this.diagram
  },
  computed: {
    selectedElementProperties () {
      if (!this.selectedElement || !this.selectedElement.values || !this.selectedElement.values.properties) {
        return null
      }
      return this.selectedElement.values.properties
    },
    selectedElementTitle () {
      if (!this.selectedElement || !this.selectedElement.title) {
        return null
      }
      return this.selectedElement.title
    },
    componentPropertiesType () {
      if (this.selectedElement) {
        if (this.selectedElement.type) {
          return 'vue-node-properties'
        } else {
          return 'vue-link-properties'
        }
      } else {
        return 'vue-link-properties'
      }
    }
  },
  methods: {
    selectElement (node) {
      console.log('select', node)
      this.selectedElement = node
    },
    deselectElement (node) {
      console.log('deselect', node)
      this.selectedElement = null
    },
    addNode (type) {
      this.$refs.container.addNewNode(type, 100, 100)
      this.emit()
    },
    saveProperties (val) {
      console.log(val)
      let diagram = this.internalDiagram
      let node = diagram.nodes.find(b => {
        return b.id === this.selectedElement.id
      })
      node.values.properties = val
      this.internalDiagram = merge({}, diagram)
    },
    saveTitle (val) {
      console.log(val)
      let diagram = this.internalDiagram
      let node = diagram.nodes.find(b => {
        return b.id === this.selectedElement.id
      })
      node.title = val
      this.internalDiagram = merge({}, diagram)
    }
  },
  watch: {
    nodes (newValue) {
      // console.log('nodes', JSON.stringify(newValue))
    },
    diagram (newValue) {
      this.$emit('update:diagram', this.internalDiagram)
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
</style>
