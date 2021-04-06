<template>
  <v-container style="text-align: center">
    <v-card
    class="d-inline-flex flex-column align-center pa-5"
    outlined
    style="min-width: 600px"
    >
      <h1>/test</h1>
      <v-text-field
        v-model="testInput"
        type="number"
        label="Input number"
        style="width: 100px"
      />
      <v-btn @click="apiTestCall">Submit</v-btn>
      <h1>{{ testResult }}</h1>
    </v-card>
    <v-card
      class="d-inline-flex flex-column align-center pa-5"
      outlined
      style="width: 600px"
    >
      <h1>/api/specific-text</h1>
      <v-text-field
        v-model="apiSpecificTextInput"
        label="Input text"
        style="width: 300px"
      />
      <v-btn @click="apiSpecificTextCall">Submit</v-btn>
      <p>{{ apiSpecificTextResult }}</p>
    </v-card>
  </v-container>
</template>

<script>

export default {
  name: "apiTest",
  data: () => ({
    testInput: 0,
    testResult: 0,
    apiSpecificTextInput: '',
    apiSpecificTextResult: '',
  }),
  methods: {
    async apiTestCall() {
      let response = await this.$axios.$get('/test?limit=' + this.testInput)
      this.testResult = response.data;
      console.log(this.testResult);
    },
    async apiSpecificTextCall() {
      let response = await this.$axios.$get('/api/specific-text?text=' + this.apiSpecificTextInput)
      this.apiSpecificTextResult = response;
      console.log(this.apiSpecificTextResult);
    }
  }
}
</script>

<style scoped>

</style>
