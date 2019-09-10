<template>
  <q-page class="flex justify-center full-width">
    <flow-designer :MMNodes="MMNodes" :diagram.sync="diagram" :componentsProperties="{
      start: 'node-propertiesX',
              end_ok: 'node-propertiesX',
              end_ko: 'node-propertiesX',
              branch_out: 'node-propertiesX',
              branch_in: 'node-propertiesX',
              rest: 'node-propertiesX',
              sms: 'node-propertiesX',
              email: 'node-propertiesX',
              delay: 'node-propertiesX',
              save: 'node-propertiesX',
              link: 'link-propertiesX'
    }" />
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

import FlowDesigner from 'components/flow-diagram/flow-designer'
import MMNodes from './mm.json'
import diagram from './diagram.json'

export default {
  name: 'App',
  components: {
    FlowDesigner
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
        title: 'Diagram Object',
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
