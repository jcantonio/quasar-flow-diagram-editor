<template>
  <div v-if="selected" >
    <q-input label="Title" type="text" v-model="internalTitle" v-on:input="saveTitle" />
    <div v-bind:key="p.name" class="property" v-for="p in properties">
      <q-input :label="p.label || p.name" type="text" v-model="p.value" v-on:input="save" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'VueNodeProperty',
  props: ['title', 'property', 'selected'],
  data () {
    return {
      internalTitle: null,
      properties: null
    }
  },
  methods: {
    loadTitle () {
      this.internalTitle = this.title
    },
    loadProperties () {
      this.properties = this.property
    },
    saveTitle () {
      this.$emit('save-title', this.internalTitle)
    },
    save () {
      this.$emit('save', this.properties)
    }
  },
  watch: {
    property () {
      this.loadProperties()
    },
    title () {
      this.loadTitle()
    }
  }
}
</script>
