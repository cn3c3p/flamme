<template>
  <div tabindex="-1" id="swiper" @keyup.left="pass" @keyup.right="like" @keyup.up="superLike" @keyup.down="nextImage">
    <ul class="card-stack">
      <card v-for="(card, index) in cards" :key="index" :match="card.user" :index="index" :main="card.user.photos[0].url" v-if="card.user"></card>
    </ul>
    <div class="head" @click="getCards" v-if="!cards.length">
      <img :src="profilepic">
      <p v-if="recs">There's no one new around you.</p>
    </div>
    <img src="../images/keyboard.png" alt="" class="keyboard">
  </div>
</template>

<script>
  import Api from '../services/Api'
  import Sync from '../services/Sync'
  import Card from './Profile/Card'
  let swipeInProgress = false
  let cardsInProgress = false
  let focusInterval
  export default {
    data () {
      return {
        cards: [],
        profilepic: '',
        recs: false
      }
    },
    components: {
      Card,
      Api
    },
    methods: {
      pass () {
        if (swipeInProgress || !this.$children[this.cards.length - 1]) return
        if (this.cards.length === 2 || this.cards.length === 1) {
          this.getCards()
        }
        App.modalController.modals = []
        swipeInProgress = true
        const child = this.$children[this.cards.length - 1]
        child.$el.style.transform = 'translateX(-470px)'
        child.$el.style.opacity = 0
        Api.pass(child.match._id, function (error, message) {
          if (error) console.log(error)
          console.log(message)
        })
        setTimeout(() => {
          this.cards.splice(-1)
          swipeInProgress = false
        }, 200)
      },
      like () {
        if (swipeInProgress || !this.$children[this.cards.length - 1]) return
        if (this.cards.length === 2 || this.cards.length === 1) {
          this.getCards()
        }
        App.modalController.modals = []
        swipeInProgress = true
        const child = this.$children[this.cards.length - 1]
        child.$el.style.transform = 'translateX(470px)'
        child.$el.style.opacity = 0
        Api.like(child.match._id, (error, message) => {
          if (error) console.log(error)
          console.log(message)
          if (message.likes_remaining === 0) {
            alert('No likes remaining.')
            return
          }
          if (message.match) {
            message.match.person = child.match
            Sync.addNewMatch(message.match)
            alert('It\'s a match!!')
            var event = new Event('update')
            window.dispatchEvent(event)
          }
          if (message.likes_remaining) {
            App.likes_remaining = message.likes_remaining
          }
        })
        setTimeout(() => {
          this.cards.splice(-1)
          swipeInProgress = false
        }, 300)
      },
      superLike () {
        if (App.super_likes_remaining < 1) {
          alert('You are out of superlikes.')
          return
        }
        if (swipeInProgress || !this.$children[this.cards.length - 1]) return
        if (this.cards.length === 2 || this.cards.length === 1) {
          this.getCards()
        }
        App.modalController.modals = []
        swipeInProgress = true
        const child = this.$children[this.cards.length - 1]
        child.$el.style.transform = 'translateY(-470px)'
        child.$el.style.opacity = 0
        Api.superLike(child.match._id, (error, message) => {
          if (error) console.log(error)
          if (message.likes_remaining === 0) {
            alert('No likes remaining.')
            return
          }
          if (message.match) {
            message.match.person = child.match
            Sync.addNewMatch(message.match)
            alert('It\'s a match!!')
            var event = new Event('update')
            window.dispatchEvent(event)
          }
          if (message.likes_remaining) {
            App.likes_remaining = message.likes_remaining
          }
        })
        setTimeout(() => {
          this.cards.splice(-1)
          swipeInProgress = false
        }, 200)
      },
      nextImage () {
        if (swipeInProgress || !this.$children[this.cards.length - 1]) return
        const child = this.$children[this.cards.length - 1]
        if (child.imageIndex === child.match.photos.length - 1) {
          child.imageIndex = 0
        } else {
          child.imageIndex++
        }
        child.imageUrl = child.match.photos[child.imageIndex].url
      },
      getCards () {
        if (cardsInProgress) { return }
        cardsInProgress = true
        this.recs = false
        console.log('fetching cards..')
        Api.getCards((error, response) => {
          console.log(response)
          cardsInProgress = false
          if (error) {
            console.log(error)
            return
          }

          if (response.message && response.message.indexOf('recs') > -1) {
            this.recs = true
            return
          }
          this.recs = false

          if (response.results) {
            this.cards = this.cards.concat(response.results)
          }
        })
      }
    },
    mounted () {
      this.getCards()
      document.getElementById('swiper').focus()
      this.profilepic = App.profile.user.photos[0].url
      focusInterval = setInterval(() => {
        document.getElementById('swiper').focus()
      }, 1000)
    },
    beforeDestroy () {
      clearInterval(focusInterval)
    },
    name: 'swipe'
  }
</script>

<style scoped>
.swiper {
  overflow: hidden;
}
.keyboard {
  position: absolute;
  bottom: 15px;
  width: 130px;
  height: auto;
  left: 50%;
  margin-left: -65px;
  right: 0;
}
.card-stack {
  list-style-type: none;
  background: #999
}
.head {
  text-align: center;
  outline: 0;
  position: fixed;
  z-index: 1;
  width: 200px;
  top: 50%;
  left: 50%;
  margin-top: -100px;
  margin-left: -100px;
}
.head img {
  width: 100px;
  height: auto;
  border-radius: 50%;
  border: 3px solid #ff4081;
  margin-bottom: 10px;
}
</style>
