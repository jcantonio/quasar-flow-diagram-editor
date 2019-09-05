<template>
  <div >
    <div v-bind:key="p.name" class="property" v-for="p in properties">
      <q-input :label="p.label || p.name" type="text" v-model="p.value" />
    </div>
    <q-btn v-if="properties != null" label="Save" @click.prevent="save"/>
  </div>
</template>

<script>
export default {
  name: 'VueBlockProperty',
  props: ['property'],
  data () {
    return {
      properties: null
    }
  },
  methods: {
    loadProperties () {
      this.properties = this.property
    },
    save () {
      this.$emit('save', this.properties)
    }
  },
  watch: {
    property () {
      this.loadProperties()
    }
  }
}
</script>
