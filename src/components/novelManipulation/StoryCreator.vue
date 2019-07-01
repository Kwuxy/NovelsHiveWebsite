<template>
  <div id="storyCreator">
    <h1>Let's begin a nice story</h1>

    <v-form
      ref="form"
      v-model="formIsValid"
      lazy-validation
    >

      <v-alert
        :value="creationError"
        type="error"
        dismissible
        style="width: 80%"
      >
        {{creationErrorMessage}}
      </v-alert>

      <v-text-field
        v-model="title"
        label="Title"
        :rules="titleRules"
        required></v-text-field>

      <v-textarea
        auto-grow
        v-model="synopsis"
        label="Synopsis"
        required
        :rules="synopsisRules"></v-textarea>

      <v-btn
        color="success"
        :disabled="!formIsValid"
        @click="createStory"
        >Create</v-btn>

    </v-form>
    <!--<editor />-->
  </div>
</template>

<script>
export default {
  data () {
    return {
      formIsValid: false,
      title: '',
      titleRules: [
        v => !!v || 'Every great story has a name !'
      ],
      synopsis: '',
      synopsisRules: [
        v => !!v || 'Let your reader know the purpose of your story'
      ],
      serverDoesNotRespondErrorMessage: process.env.SERVER_DOES_NOT_RESPOND_ERROR_MESSAGE,
      creationError: false,
      creationErrorMessage: ''
    }
  },
  methods: {
    createStory () {
      if ((this.title === null || this.title === '') || (this.synopsis === null || this.synopsis === '')) {
        this.formIsValid = false
        return
      }

      let now = Date.now()

      let requestBody = {
        'title': this.title,
        'synopsis': this.synopsis,
        'publication_date': now,
        'update_date': now,
        'userId': localStorage.userId
      }

      this.$http.post(process.env.API_LOCATION + '/stories', requestBody)
        .then(response => this.createSuccessful(response))
        .catch(error => this.createFailed(error))
    },
    createFailed (error) {
      this.creationError = true

      if (!error.response) {
        this.creationErrorMessage = this.serverDoesNotRespondErrorMessage
        return
      }

      if (error.response.status === 409) {
        this.creationErrorMessage = 'You already created a story with this name'
      } else {
        this.creationErrorMessage = this.serverDoesNotRespondErrorMessage
      }
    },
    createSuccessful (response) {
      this.$router.push('/') // Where to redirect ?
    }
  }
}
</script>

<style scoped>

</style>