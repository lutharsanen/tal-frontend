<template>
  <div>
    <v-autocomplete
      :items="objects"
      v-model="selectedObject"
      label="Objects"
    />
    <v-select
      v-model="numberOfObjects"
      :items="numbers"
    />
    <v-btn @click="query">Query</v-btn>
  </div>
</template>

<script>
export default {
  name: "ObjectNumber",
  props: {
    objects: Array
  },
  data: () => ({
    selectedObject: '',
    numberOfObjects: '',
    numbers: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  }),
  methods: {
    async query() {
      var data = {
        object: this.selectedObject,
        number: this.numberOfObjects
      }
      console.log('****API SEARCH BY OBJECT NUMBER****', data)
      this.loading = true
      try {
        let response = await this.$axios.$post('/api/searchByNumberObject', data)
        console.log('SearchByObjectNumber: ', response)
        if (response.results.length > 0) {
          console.log('emit query: ', response.results.length)
          this.$emit('query', response);
        } else {
          console.log('emit snackbar: ', response.results.length)
          this.$emit('snackbar', 'No results')
        }
      } catch (e) {
        console.log(e);
        this.$emit('snackbar', e.message, 'red')
      }
      this.loading = false
    }
  }
}
</script>

<style scoped>

</style>
