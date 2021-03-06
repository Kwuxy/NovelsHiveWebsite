<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
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

      <v-select
        v-model="kind"
        :items="kinds"
        label="Kind"
        :rules="kindRules"
        required
        menu-props="auto"
        @change="getKindIdByName"
        attach></v-select>

      <tag-combobox
      @tags-changed="updateTags"></tag-combobox>

      <v-textarea
        auto-grow
        v-model="synopsis"
        label="Synopsis"
        required
        :rules="synopsisRules"></v-textarea>

      <beta-reader
      @beta-readers-changed="updateBetaReader"
      style="margin-bottom: 3%;"></beta-reader>

      <div id="pictureInputContainer">
        <vue-picture-input
          ref="pictureInput"
          accept="image/jpg, image/jpeg, image/png"
          removable
          :crop="false"
          remove-button-class="v-btn error"
          button-class="v-btn"
          label="Panel (Optional)"
          :z-index="0"
        ></vue-picture-input>
      </div>

      <v-btn
        color="success"
        :disabled="!formIsValid"
        @click="createStory"
        >Create</v-btn>
    </v-form>
  </div>
</template>

<script>
import TagCombobox from '../utils/tagCombobox'
import VuePictureInput from 'vue-picture-input'
import FormDataUpload from '../../scripts/formDataUpload'
import BetaReader from '../betaReaders/BetaReader'

export default {
  components: {TagCombobox, VuePictureInput, BetaReader},
  data () {
    return {
      formIsValid: false,
      title: '',
      titleRules: [
        v => !!v || 'Every great story has a name !'
      ],
      kindRules: [
        v => !!v || 'What is the kind of your story ?'
      ],
      synopsis: '',
      synopsisRules: [
        v => !!v || 'Let your reader know the purpose of your story'
      ],
      serverDoesNotRespondErrorMessage: process.env.SERVER_DOES_NOT_RESPOND_ERROR_MESSAGE,
      creationError: false,
      creationErrorMessage: '',
      kind: '',
      kinds: [],
      kindsObject: [],
      kindId: 0,
      tags: [],
      tagsProposal: [],
      tagValue: '',
      betaReaders: []
    }
  },
  created () {
    this.$http.get(process.env.API_LOCATION + '/kinds')
      .then((response) => {
        this.kindsObject = response.data

        response.data.forEach((kind) => {
          this.kinds.push(kind.name)
        })
      })
      .catch((error) => console.log(error))
  },
  methods: {
    createStory () {
      if ((this.title === undefined || this.title === '') || (this.synopsis === undefined || this.synopsis === '') ||
          (this.kindId === undefined || this.kindId === '')) {
        this.formIsValid = false
      }

      if (this.$refs.pictureInput.file) {
        FormDataUpload(process.env.API_LOCATION + '/containers/storyImage/upload', this.$refs.pictureInput.file, 'file')
          .then(response => {
            this.storyCreation(response.data.result.files.file[0].name)
          })
      } else {
        this.storyCreation()
      }
    },
    storyCreation (filename) {
      let now = Date.now()

      let requestBody = {
        'title': this.title,
        'synopsis': this.synopsis,
        'publication_date': now,
        'update_date': now,
        'userId': localStorage.userId,
        'storyKindId': this.kindId
      }

      if (filename !== undefined) {
        requestBody.panel = filename
      }

      if (this.tags !== null && this.tags !== undefined) {
        requestBody.storyTags = this.tags
      }

      if (this.betaReaders !== null && this.betaReaders !== undefined) {
        requestBody.betaReaders = this.betaReaders
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
    },
    getKindIdByName () {
      this.kindsObject.forEach((kind) => {
        if (kind.name === this.kind) {
          this.kindId = kind.id
        }
      })
    },
    removeTag (item) {
      this.tags.splice(this.tags.indexOf(item), 1)
      this.tags = [...this.tags]
    },
    updateTags (tags) {
      this.tags = tags
    },
    updateBetaReader (betaReaders) {
      this.betaReaders = betaReaders
    }
  },
  watch: {
    tagValue () {
      let tag = this.tagValue

      if (tag === null || tag === undefined) return

      tag = tag.trim()

      if (tag.length < 3) {
        this.tagsProposal = []
        return
      }

      this.$http.get(process.env.API_LOCATION + '/tags', {
        params: {
          'filter': {
            'where': {
              'name': {
                'like': '%' + tag + '%'
              }
            }
          }
        }
      }).then(response => {
        let tags = response.data.map(object => object.name)

        this.tagsProposal = tags
      })
    }
  }
}
</script>

<style scoped>
  #pictureInputContainer {
    max-width: 30%;
    max-height: 30%;
    margin: auto;
  }

  .containerOfContainer {
    max-width: 30%;
    margin: auto;
  }

  .container {
    background-color: red;
    position: relative;
    padding-top: 75%; /* 4:3 Aspect Ratio */
  }

  .text {
    position:  absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    text-align: center;
    font-size: 20px;
    color: white;
  }
</style>
