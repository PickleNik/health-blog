<template>
  <v-app dark class="head">
    <v-container fluid class="app">
      <v-flex md3 lg2 class="zoom hidden-sm-and-down pa-4" style="position:fixed; z-index: 100; left:0; top:0;">
        <v-layout column>
          <a v-for="nav in navs" :key="nav.name" href="#" v-scroll-to="{el: nav.href, offset: -50}">
            <v-btn small round flat class="ma-0 pr-2 link link--kukuri">
              <v-icon left class="pink--text">{{ nav.icon }}</v-icon>
              {{ nav.name }}
            </v-btn>
          </a>
        </v-layout>
      </v-flex>
      <v-flex md3 lg2 class="zoom pa-4" style="position:fixed; z-index: 100; right:0; top:0;">
        <v-tooltip bottom color="pink" v-if="!user"><span class="text--text">Log in with Google</span><v-btn class="ma-2" large slot="activator" color="pink" fab flat id="b1" @click="GoogleSignIn" :loading="loading"><v-icon large>fab fa-google</v-icon></v-btn></v-tooltip>
        <v-tooltip bottom color="pink" v-if="user"><span class="text--text text-xs-center">Logout</span>
          <v-avatar slot="activator" class="ma-2" @click="Logout" size="72">
             <img :src="user.photo" alt="alt">
          </v-avatar>
        </v-tooltip>
      </v-flex>
      <v-layout row wrap>
        <v-flex xs12 sm12 class="text-xs-center mb-5">
          <v-layout column>
            <h1 id="title" class="white--text mt-5">Health Fair Blog</h1>
            <a class="mt-5" href="https://www.linkedin.com/in/picklenik/"><img id="face" src="./assets/me.jpg"></a>
            <span id="info"><i><h2> by Nikita Krupin</h2></i>
            Junior @ WJHS</span>
          </v-layout>
        </v-flex>
        <v-flex xs12 sm12 md6 offset-md3 lg8 offset-lg2 class="post">
          <v-layout column>
            <v-flex v-for="p in posts" :key="p.id" :id="p.id" class="my-4 pa-4 round black">
              <v-layout row class="pb-4">
                <span class="pt-3 pink--text" style="font-size: 2em; font-family: 'Indie Flower', sans-serif;">{{ p.title }}</span>
                <v-spacer></v-spacer>
                <social-sharing inline-template
                  :url="'https://health-fair.firebaseapp.com#' + p.id"
                  :title="p.title"
                  hashtags="healthfair, mdma, ecstasy, molly">
                <v-speed-dial v-model="p.fab" direction="left" :open-on-hover="true" transition="slide-y-transition" >
                  <v-btn slot="activator" v-model="p.fab" style="position: sticky;" dark fab depressed >
                    <v-icon>share</v-icon>
                    <v-icon>close</v-icon>
                  </v-btn>
                  <v-btn fab dark small color="grey" class="hidden-sm-and-down" v-clipboard:copy="'https://health-fair.firebaseapp.com#' + p.id">
                    <v-icon>faa fa-copy</v-icon>
                  </v-btn>
                  <v-btn fab dark small color="light-blue">
                    <network network="twitter">
                      <v-icon>fab fa-twitter</v-icon>
                    </network>
                  </v-btn>
                  <v-btn fab dark small color="blue">
                    <network network="facebook">
                      <v-icon>fab fa-facebook-f</v-icon>
                    </network>
                  </v-btn>
                  <v-btn fab dark small color="blue darken-1" class="hidden-sm-and-down">
                    <network network="linkedin">
                      <v-icon>fab fa-linkedin-in</v-icon>
                    </network>
                  </v-btn>
                  <v-btn fab dark small color="green" class="hidden-md-and-up">
                    <network network="whatsapp">
                      <v-icon>fab fa-whatsapp</v-icon>
                    </network>
                  </v-btn>
                  <v-btn fab dark small color="pink" class="hidden-sm-and-down">
                    <network network="pinterest">
                      <v-icon>fab fa-pinterest-p</v-icon>
                    </network>
                  </v-btn>
                  <v-btn fab dark small color="red" class="hidden-sm-and-down">
                    <network network="googleplus">
                      <v-icon>fab fa-google-plus-g</v-icon>
                    </network>
                  </v-btn>
                </v-speed-dial>
                </social-sharing>
              </v-layout>
              <v-divider></v-divider>

              <v-flex v-if="p.video" class="my-4 video"><iframe :src="p.video" frameborder="0" style="border-radius: 3em !important;" allow="autoplay; encrypted-media" allowfullscreen></iframe></v-flex>

              <p v-if="p.typed" class="pt-2 text-xs-justify" style="font-size: 1.5em; min-height: 4.5em;" id="greeting"></p>

              <p v-if="p.text" class="pt-2 text-xs-justify" style="font-size: 1.5em;">{{ p.text }}</p>

              <v-img v-if="p.image" :src="p.image" :lazy-src="p.image" class="mx-auto round grey"></v-img>

              <a v-if="p.links" v-for="link in p.links" :key="link.title" :href="link.href" target="_blank"><v-btn small round flat class="link link--kukuri"><v-icon left class="pink--text">link</v-icon>{{ link.title }}</v-btn></a>

              <v-list three-line v-if="user" class="round mt-2 pt-3">
                <template v-for="comment in comments" v-if="comment.post === p.id">
                  <v-list-tile avatar>
                    <v-list-tile-avatar>
                      <img :src="comment.photo">
                    </v-list-tile-avatar>
                    <v-list-tile-content>
                      <v-list-tile-title v-html="comment.name"></v-list-tile-title>
                      <v-list-tile-sub-title v-html="comment.text"></v-list-tile-sub-title>
                    </v-list-tile-content>
                    <v-list-tile-action v-if="comment.creator == user.id || user.id == 'N9KttaK3lcfjPAtbGG9c5wIS4Bj1'" class="pr-4">
                      <v-btn icon flat small color="pink" @click="deleteComment(comment.text)" :loading="loading"><v-icon>delete</v-icon></v-btn>
                    </v-list-tile-action>
                  </v-list-tile>
                  <v-divider inset></v-divider>
                </template>
                <v-text-field  v-on:keypress.enter="addComment(p.comment, p.id), p.comment = ''" :loading="loading" counter="200" v-if="user" class="mx-3" v-model="p.comment" color="pink" placeholder="Comment" background-color="grey darken-3 pl-0 py-3 elevation-0" solo>
                  <v-avatar slot="prepend-inner" class="mr-2" size="50">
                    <img :src="user.photo" alt="alt">
                  </v-avatar>
                  <v-btn icon flat color="pink" class="mr-0" slot="append" @click="addComment(p.comment, p.id), p.comment = ''" :loading="loading" :disabled="p.comment == ''"><v-icon>send</v-icon></v-btn>
                </v-text-field>
              </v-list>
            </v-flex>
          </v-layout>
        </v-flex>
      </v-layout>
    </v-container>
  </v-app>
</template>

<script>
import Typed from 'typed.js'
import firebase from 'firebase/app'
import 'firebase/auth'

export default {
  data () {
    return {
      typedOptions: ['Hi there', '3,4-Methyl​enedioxy​methamphetamine(or MDMA), commonly known as ecstasy, is a psychoactive drug primarily used as a recreational drug. The desired effects include altered sensations and increased energy, empathy, and pleasure. When taken by mouth, effects begin after 30–45 minutes and last 3–6 hours.'],
      direction: 'bottom',
      posts: [
        {
          title: 'What is Molly',
          typed: true,
          fab: false,
          id: 'post1',
          links: [
            { title: 'Outline', href: 'https://docs.google.com/document/d/1iZa7Gf0W3FIoVVw-vEPF_gbyCfb3pTEAJkVRuuvI--A/edit?usp=sharing' }
          ]
        },
        {
          title: 'Quiz',
          comment: '',
          fab: false,
          id: 'post2',
          text: 'Tip of the day: Ecstasy, in fact, is a term for Molly which has been chemically altered with other additives, such as amphetamine, caffeine etc.',
          links: [
            { title: 'Quiz', href: 'https://www.justthinktwice.gov/quiz/quiz-ecstasy-and-mdma' },
            { title: 'Source 1', href: 'https://www.drugabuse.gov/publications/drugfacts/mdma-ecstasymolly' },
            { title: 'Source 2', href: 'https://teens.drugabuse.gov/drug-facts/mdma-ecstasy-or-molly' },
            { title: 'Source 3', href: 'https://abovetheinfluence.com/drugs/mdma/' }
          ]
        },
        {
          title: 'Video',
          comment: '',
          fab: false,
          id: 'post3',
          video: 'https://www.youtube.com/embed/jEAr7ThsYew'
        },
        {
          title: 'Article',
          text: 'Developed in 1914 as an appetite suppressant, MDMA gained popularity in the 1980s with young adults at large music festivals and all-night dance parties or raves. \n The user experiences feelings of euphoria, increased energy, intimacy and emotional warmth, sensitivity to touch, and a distortion of time and of the senses. \n Ecstasy is frequently taken with other illegal drugs, and pills sold as MDMA on the street often contain additives. These factors can contribute to serious, and sometimes fatal, health effects. \n MDMA can also be addictive, and research suggests that long-term cognitive problems may develop in some users.',
          comment: '',
          fab: false,
          id: 'post4',
          links: [
            { title: 'Full Article', href: 'https://www.medicalnewstoday.com/articles/297064.php' }
          ]
        },
        {
          title: 'ADVOCACY',
          text: 'Local Resources or where to get addiction treatment:',
          comment: '',
          fab: false,
          id: 'post5',
          links: [
            { title: 'The Freedom Center', href: 'https://www.thefreedomcenter.com' },
            { title: 'Banyan Treatment Center', href: 'https://help.banyantreatmentcenter.com' },
            { title: 'Transformations Treatment Center', href: 'https://www.transformationstreatment.center' },
            { title: 'Map & More', href: 'https://www.google.com/search?q=ecstasy+addiction+treatment+MD&npsic=0&rflfq=1&rlha=0&rllag=39011768,-77118557,13142&tbm=lcl&ved=2ahUKEwiNrdX7yf_fAhWNt1kKHa6eAfoQtgN6BAgAEAQ&tbs=lrf:!2m1!1e2!2m1!1e3!2m1!1e16!3sIAE,lf:1,lf_ui:2&rldoc=1#rlfi=hd:;si:;mv:!1m2!1d39.5497361!2d-75.8523759!2m2!1d38.672478!2d-77.25952389999999;tbs:lrf:!2m1!1e2!2m1!1e3!2m1!1e16!3sIAE,lf:1,lf_ui:2' }
          ]
        },
        {
          title: 'Video',
          comment: '',
          image: 'https://i.pinimg.com/originals/8f/f4/47/8ff44710fdfa2222fcf6a3e29f1419ef.gif',
          fab: false,
          id: 'post6',
          video: 'https://www.youtube.com/embed/_sXhbieSzZs'
        }
      ],
      navs: [
        { name: 'Creator', href: '#title', icon: 'assignment_ind' },
        { name: 'About', href: '#post1', icon: 'assignment' },
        { name: 'Quiz', href: '#post2', icon: 'assignment_turned_in' },
        { name: 'Video', href: '#post3', icon: 'video_library' },
        { name: 'Article', href: '#post4', icon: 'library_books' },
        { name: 'ADVOCACY', href: '#post4', icon: 'location_on' },
        { name: 'Video', href: '#post4', icon: 'video_library' }
      ]
    }
  },
  mounted () {
    new Typed('#greeting', {// eslint-disable-line no-new
      strings: this.typedOptions,
      loop: false,
      typeSpeed: 50,
      backSpeed: 10,
      startDelay: 3000,
      backDelay: 2000,
      smartBackspace: true,
      showCursor: false
    })
  },
  computed: {
    user () {
      return this.$store.getters.user
    },
    error () {
      return this.$store.getters.error
    },
    loading () {
      return this.$store.getters.loading
    },
    comments () {
      return this.$store.getters.comments
    }
  },
  created () {
    this.$store.dispatch('getComments')
  },
  methods: {
    GoogleSignIn () {
      this.$store.dispatch('userProvider', new firebase.auth.GoogleAuthProvider())
    },
    Logout () {
      this.$store.dispatch('userLogout')
    },
    addComment (text, id) {
      if (text !== '') {
        this.$store.dispatch('addComment', {text: text, post: id, creator: this.user.id, name: this.user.name, photo: this.user.photo})
        .then(this.$store.dispatch('getComments'))
      }
    },
    deleteComment (text) {
      this.$store.dispatch('deleteComment', text)
      .then(this.$store.dispatch('getComments'))
    }
  }
}
</script>

<style media="screen">
@import url('https://fonts.googleapis.com/css?family=Quicksand');
@import url('https://fonts.googleapis.com/css?family=Permanent+Marker');
@import url('https://fonts.googleapis.com/css?family=Indie+Flower');
*{
  font-family: 'Quicksand', sans-serif;
}
html{
  background: black;
}
a {
  text-decoration: none !important;
}
.v-input__slot {
  border-radius: 2em !important;
}
.video {
  -webkit-mask-image: -webkit-radial-gradient(circle, white 100%, black 100%); /*ios 7 border-radius-bug */
  -webkit-transform: rotate(0.000001deg); /*mac os 10.6 safari 5 border-radius-bug */
  -webkit-border-radius: 3em;
  -moz-border-radius: 3em;
  border-radius: 3em;
  overflow: hidden;
  z-index: 9;
	position: relative;
	padding-bottom: 57%; /* 16:9 */
	padding-top: 25px;
	height: 0;
}
.video iframe {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
}
body{
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch;
}
.head{
  position: relative;
}
.head::before{
  content: "";
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  position: absolute;
  background-image: url('https://i.pinimg.com/originals/e3/a6/4f/e3a64fe6ec8533b097c5708a19a00d10.jpg');
  background-attachment: fixed;
  background-repeat: repeat;
  filter: brightness(30%);
}
.round{
  border-radius: 2em;
}
::-webkit-scrollbar{
  width: 0;
}
::selection{
  background:transparent;
  color: pink;
}
@keyframes rollIn {
  from {
    opacity: 0;
    transform: translate3d(-100%, 0, 0) rotate3d(0, 0, 1, -120deg);
  }

  to {
    opacity: 1;
    transform: none;
  }
}
@keyframes zoomIn {
  from {
    opacity: 0;
    transform: scale3d(.3, .3, .3);
  }

  50% {
    opacity: 1;
  }
  to {
    opacity: 1;
  }
}
@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translate3d(0, -100%, 0);
  }

  to {
    opacity: 1;
    transform: none;
  }
}
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translate3d(0, 100%, 0);
  }

  to {
    opacity: 1;
    transform: none;
  }
}
@keyframes lightSpeedIn {
  from {
    transform: translate3d(100%, 0, 0) skewX(-30deg);
    opacity: 0;
  }

  60% {
    transform: skewX(20deg);
    opacity: 1;
  }

  80% {
    transform: skewX(-5deg);
    opacity: 1;
  }

  to {
    transform: none;
    opacity: 1;
  }
}
#title {
  font-family: 'Indie Flower', sans-serif;
  font-size: 3em;
  opacity: 0;
  animation: fadeInDown 1s 1s ease 1 forwards;
}
#face{
  opacity: 0;
  height: 30vh;
  width: 30vh;
  border-radius: 100%;
  animation: rollIn 1s 1.5s ease 1 forwards;
}
#info{
  opacity: 0;
  animation: lightSpeedIn 1s 1.5s ease 1 forwards;
}
.post{
  opacity:0;
  animation: fadeInUp 1s 2s ease 1 forwards;
}
.zoom{
  opacity:0;
  animation: zoomIn 1s 2s ease 1 forwards;
}
.link {
  text-decoration: none;
  position: relative;
}
/* Kukuri */
.link--kukuri {
  text-transform: uppercase;
  font-weight: 900;
  overflow: hidden;
  line-height: 0.75;
  color: white;
}

.link--kukuri:hover {
  color: #ea4964;
}

.link--kukuri::after {
  content: '';
  position: absolute;
  height: 6px;
  width: 100%;
  top: 50%;
  margin-top: -4px;
  right: 0;
  background: #ea4964;
  border-radius:1em;
  -webkit-transform: translate3d(-111%,0,0);
  transform: translate3d(-111%,0,0);
  -webkit-transition: -webkit-transform 0.4s;
  transition: transform .3s;
  -webkit-transition-timing-function: cubic-bezier(0.7,0,0.3,1);
  transition-timing-function: cubic-bezier(0.7,0,0.3,1);
}

.link--kukuri:hover::after {
  -webkit-transform: translate3d(100%,0,0);
  transform: translate3d(100%,0,0);
}
.link--kukuri:hover::before {
  width: 100%;
}
</style>
