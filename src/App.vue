<template>
  <div id="app" class="col-md-8 offset-md-2">
    <a class="appSettings" @click="showZone('adminZone')"><i class="fa fa-cog pull-right" v-b-tooltip.hover title="Change sentence or sight words" ></i></a>
    <a class="appSettings" @click="showZone('infoZone')"><i class="fa fa-question-circle pull-right"></i></a>
    <h1 class="header" @click="showZone('questionZone')">Sight Word Practice</h1>
    <section class="questionZone" v-bind:class="contentClass('questionZone')">
      <h2>{{currentTitle}}</h2>
      <hr />
      <div class="questionContainer">
        <b-button variant="primary" @click="sayIt()" v-bind:style="{'background-color': randColor()}">
          <span class="questionTxt">{{currentMsg()}}</span>
          <b-badge variant="light" v-on:click.stop="nextSight()">
            &#x2192;
          </b-badge>
        </b-button>
      </div>
    </section>
    <section class="adminZone text-center" v-bind:class="contentClass('adminZone')">
      <hr />
      <div class="input-group mb-3">
        <label for="sightSentence" class="col-md-2 offset-md-2">Practice sentence:</label>
        <a @click="restorePrev('sentence')" v-bind:class="prevSentence.length ? 'd-inline-block' : 'd-none'">
          <span class="fa-stack">
            <i class="fa fa-circle fa-stack-2x"></i>
            <i class="fa fa-undo fa-stack-1x fa-inverse"></i>
          </span>
          Undo?
        </a>
        <input name="sightSentence" v-model="sentence" v-on:focus="updateQuiz('sentence', sentence)" type="text" class="col-md-6 form-control" placeholder="Enter reading practice sentence" aria-label="Sentence" />
      </div>
      <h4>Current sight words:</h4>
      <ul>
        <li v-for="(sightWord, index) in sightWords">
          <span class="wordItem">
            {{ sightWord }}
            <a @click="removeWord(index)"><i class="fa fa-times-circle pull-right"></i></a>
          </span>
        </li>
      </ul>
      <label for="addSightword" class="col-md-6">Paste new sight words (comma or space seperated, max. 50):</label>
      <a @click="restorePrev('words')" v-bind:class="prevSightWords.length ? 'd-inline-block' : 'd-none'">
        <span class="fa-stack">
          <i class="fa fa-circle fa-stack-2x"></i>
          <i class="fa fa-undo fa-stack-1x fa-inverse"></i>
        </span>
        Undo?
      </a>
      <div>
        <textarea :value="newSightWords" @focus="updateQuiz('words')" @input="updateQuiz('words', $event.target.value)" rows="5" cols="250" class="col-md-6"></textarea>
      </div>
      <button type="button" class="btn btn-success" @click="showZone('questionZone')">Save Changes</button>
    </section>
    <section class="text-center addendum" v-bind:class="contentClass('infoZone')">
    <hr />
    <h4>What is this?</h4>
    <br />
    <p>Made this for my younger kids who are learning to read.</p>
    <p>If they get it right, they can eat that color skittle!</p>
    </section>
  </div>
</template>
<style lang="scss">
body {
  background: transparent url('./assets/657907.jpg') no-repeat center fixed;
  background-size: cover;
}
#app {
  background-color: rgba(255, 255, 255, .5);
  font-family: 'Lato', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 1em 2em 4em;
  margin: 4em auto 2em auto;
  min-height: 60vh;
}
a {cursor: pointer}
.adminZone {
  margin-top: 11vh;
}
h1, h2 {
  font-weight: normal;
  color: #333;
  text-shadow: 1px 1px 2px rgba(255,255,255,0.5);
}
h1.header {
  cursor: pointer;
}
h2 {
  margin-top: 10vh;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.questionZone {
  margin: 10vh auto auto auto;
  width: 50vw;
  position: relative;
  text-shadow: 2px 0 0 #fff, -2px 0 0 #fff, 0 2px 0 #fff, 0 -2px 0 #fff, 1px 1px #fff, -1px -1px 0 #fff, 1px -1px 0 #fff, -1px 1px 0 #fff;
}
.questionContainer  {
  display: flex;
  justify-content: center;
  padding-bottom: 2rem;
  .btn-primary {
    display: flex;
    justify-content: space-between;
    white-space: normal;
    font-size: 2.5rem;
    text-align: right;
    min-width: 25vw;
    border-width: 0;
    &:hover,:focus,:active {
      box-shadow: none;
      border-width: 0;
    }
    .questionTxt {
      justify-content: center;
    }
    .badge {
      align-self: stretch;
      margin-left: 2rem;
    }
  }
}
.appSettings {
  font-size: 2rem;
}
</style>
<script>
export default {
  name: 'app',
  data () {
    return {
      currentZone: 'questionZone',
      returnZone: '',
      currentTitle: '',
      sentence: 'I can jump over a mat.',
      prevSentence: '',
      newSentence: '',
      sightWords: ['jump', 'not', 'up', 'down', 'like', 'have', 'over', 'too', 'it', 'yes'],
      newSightWords: [],
      prevSightWords: [],
      currentSight: 0,
      currentMsg: function () {
        let returnTitle
        let returnMsg
        if (this.currentZone === 'questionZone') {
          if (this.currentSight < 1) {
            returnTitle = 'Read the sentence:'
            returnMsg = this.sentence
          } else if (this.currentSight > 0) {
            returnTitle = 'Sight word #' + this.currentSight
            returnMsg = this.sightWords[this.currentSight - 1]
          }
        }
        if (this.currentZone === 'adminZone') {
          returnTitle = 'Update sight words'
        }
        this.currentTitle = returnTitle
        if (returnMsg) {
          if (returnMsg.length) {
            return returnMsg
          } else {
            return this.currentMsg()
          }
        }
      },
      sightIdx: function () {
        return this.currentSight
      },
      nextSight: function (evt) {
        let totalLength = this.sentence.length ? this.sightWords.length - 1 : this.sightWords.length
        if (this.currentSight <= totalLength) {
          this.currentSight++
        } else {
          this.currentSight = 0
        }
      },
      showZone: function (fromZone) {
        if (fromZone === this.currentZone) {
          this.currentZone = this.returnZone
        } else {
          this.returnZone = this.currentZone
          this.currentZone = fromZone
        }
      },
      contentClass: function (zoneName) {
        console.log('fn contentClass', zoneName)
        let returnClass = (zoneName !== this.currentZone) ? 'd-none' : 'd-block'
        return returnClass
      },
      removeWord: function (wordIndex) {
        this.sightWords.splice(wordIndex, 1)
      },
      updateQuiz: function (questionType, textVal) {
        switch (questionType) {
          case 'words':
            const collectLimit = 50
            let collectWords = []
            this.prevSightWords = this.sightWords
            if (typeof textVal !== 'undefined') {
              textVal.split(/[\s,]+/).forEach(function (word) {
                if (word.length && collectWords.length < collectLimit) {
                  collectWords.push(word)
                }
              })
              this.newSightWords = collectWords
              this.sightWords = collectWords
            }
          case 'sentence':
            if (typeof textVal !== 'undefined') {
              this.prevSentence = this.sentence
            }
        }
      },
      restorePrev: function (restoreType) {
        switch (restoreType) {
          case 'words':
            this.sightWords = this.prevSightWords
            this.prevSightWords = []
            this.newSightWords = []
          case 'sentence':
            this.sentence = this.prevSentence
            this.prevSentence = ''
            this.newSentence = ''
        }
      },
      randColor: function (needsOpacity) {
        let colorList = [
          {color: '255, 0, 255'},
          {color: '0, 255, 0'},
          {color: '128, 0, 128'},
          {color: '0, 89, 255'}
        ]
        let colorIdx = Math.floor(Math.random()*(colorList.length))
        let returnColor = colorList[colorIdx]
        let opacityBg = needsOpacity ? ', .5' : ''
        colorIdx =+ this.currentSight < colorList.length ? this.currentSight : 0
        if (!needsOpacity) {
          colorIdx =- 1
        }
        if (this.currentZone === 'adminZone') {
          returnColor = ''
        } else {
          returnColor = 'rgba(' + returnColor.color + opacityBg + ')'
        }
        return returnColor
      },
      sayIt: function () {
        let synth = window.speechSynthesis;
        let sayDat = new SpeechSynthesisUtterance(this.currentMsg());
        let voice = synth.getVoices()[0]
        sayDat.voice = voice
        sayDat.pitch = 1.5; 
        sayDat.rate = 1.1;
        synth.speak(sayDat)
      }
    }
  }
}
</script>

