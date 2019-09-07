<template>
  <q-page class="flex justify-center full-width">
    <vue-flow-designer :MMNodes="MMNodes" :diagram.sync="diagram" />
    <q-page-sticky position="bottom-left" :offset="[18, 18]">
      <q-btn
        icon="info"
        color="secondary"
        label="show diagram object"
        @click="showDialog"
      ></q-btn>
    </q-page-sticky>
  </q-page>
</template>

<script>

import VueFlowDesigner from 'components/VueFlowDesigner'
import MMNodes from './mm.json'
import diagram from './diagram.json'

export default {
  name: 'App',
  components: {
    VueFlowDesigner
  },
  data: function () {
    return {
      MMNodes: MMNodes,
      diagram: diagram
    }
  },
  methods: {
    showDialog () {
      const self = this
      const json = JSON.stringify(self.diagram, undefined, 2)
      this.$q.dialog({
        title: 'Alert<em>!</em>',
        message: '<pre>' + json + '</pre>',
        html: true
      }).onOk(() => {
        console.log('OK')
      }).onCancel(() => {
        console.log('Cancel')
      }).onDismiss(() => {
        console.log('I am triggered on both OK and Cancel')
      })
    }
  }
}
</script>
