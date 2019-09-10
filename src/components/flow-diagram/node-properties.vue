<template>
  <div>
    <q-input label="Title" type="text" v-model="internalTitle" v-on:input="saveTitle" />
    <div v-bind:key="p.name" class="property" v-for="p in internalProperties">
      <q-input :label="p.label || p.name" type="text" v-model="p.value" v-on:input="save" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'node-properties',
  props: ['title', 'properties'],
  data () {
    return {
      internalTitle: null,
      internalProperties: null
    }
  },
  mounted () {
    this.loadTitle()
    this.loadProperties()
  },
  methods: {
    loadTitle () {
      this.internalTitle = this.title
    },
    loadProperties () {
      this.internalProperties = this.properties
    },
    saveTitle () {
      this.$emit('save-title', this.internalTitle)
    },
    save () {
      this.$emit('save', this.internalProperties)
    }
  },
  watch: {
    properties () {
      this.loadProperties()
    },
    title () {
      this.loadTitle()
    }
  }
}
</script>
