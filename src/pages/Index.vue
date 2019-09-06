<template>
  <q-page class="flex justify-center full-width">
    <div class="row full-width">
      <div class="col-10 q-card">
        <VueNodesContainer
          ref="container"
          :MMNodes="MMNodes"
          :diagram.sync="diagram"
          @nodeSelect="selectNode"
          @nodeDeselect="deselectNode"
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
                <VueNodeProperty
                  :selected="selectedNode!=null"
                  :title="selectedNodeTitle"
                  @save-title="saveTitle"
                  :properties="selectedNodeProperties"
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
      selectedType: 'delay'
    }
  },
  computed: {
    selectedNodeProperties () {
      if (!this.selectedNode || !this.selectedNode.values || !this.selectedNode.values.properties) {
        return null
      }
      return this.selectedNode.values.properties
    },
    selectedNodeTitle () {
      if (!this.selectedNode || !this.selectedNode.title) {
        return null
      }
      return this.selectedNode.title
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
    addNode (type) {
      this.$refs.container.addNewNode(type, 100, 100)
    },
    saveProperty (val) {
      console.log(val)

      let diagram = this.diagram
      let node = diagram.nodes.find(b => {
        return b.id === this.selectedNode.id
      })
      node.values.properties = val

      this.diagram = merge({}, diagram)
    },
    saveTitle (val) {
      console.log(val)

      let diagram = this.diagram
      let node = diagram.nodes.find(b => {
        return b.id === this.selectedNode.id
      })
      node.title = val

      this.diagram = merge({}, diagram)
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
</style>
