<template>
  <v-row justify="center">
    <v-dialog
      v-model="showDialog"
      persistent
      max-width="600px"
    >
      <v-card>
        <v-card-title>
          <span class="headline">Login Details</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  label="Session ID*"
                  v-model="currentId"
                  :default="sessionId"
                  required
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="blue darken-1"
            text
            @click="submitId"
          >
            Submit
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
export default {
  name: "LoginDialog",
  props: {
    sessionId: String
  },
  data: () => ({
    currentId: ''
  }),
  methods: {
    submitId() {
      if (this.currentId !== '') {
        localStorage.sessionId = this.currentId;
        this.$emit('submit', this.currentId)
      }
      else {
        this.$emit('snackbar', 'Please submit session ID')
      }
    }
  },
  computed: {
    showDialog: function () {
      return this.sessionId.length < 1
    }
  }
}
</script>

<style scoped>

</style>
