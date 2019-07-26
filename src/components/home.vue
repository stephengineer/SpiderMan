<template>
  <div>
    <!-- Todo: this container should be removed because I add this instruction page to intruduce project -->
    <v-container v-show="homePage">
      <v-layout align-center justify-center column>
        <span style="color: dodgerblue; font-size: 70px;"> Welcome to Spider Man </span>
        <v-img src="../assets/spiderManPeace.png" class="ma-10"></v-img>
        <v-btn x-large dark rounded color="primary" minWidth=300 minHeight=60 elevation=24 class="mb-10"
          @click="homePage=!homePage, enterButtonLabel='Continue'">{{enterButtonLabel}}
        </v-btn>
        <a href="https://github.com/zhongqi1112/spider"><img alt="GitHub stars" src="https://img.shields.io/github/stars/zhongqi1112/spider?style=social"></a>
        <v-flex class="mt-1">
          <label><a href="https://github.com/zhongqi1112">Stephen Wang</a></label>
        </v-flex>
      </v-layout>
    </v-container>
    <v-container fluid grid-list-md v-show="!homePage" class="pa-0">
      <v-layout row>
        <v-flex sm3 v-show="sidePanel">
          <v-card dark height="100%">
            <v-list>
              <v-layout row>
                <v-tooltip bottom>
                  <template v-slot:activator="{ on }">
                    <v-flex xs2>
                      <v-btn icon v-on="on" @click="homePage=!homePage">
                        <v-icon color="error">home</v-icon>
                      </v-btn>
                    </v-flex>
                  </template>
                  <span>Home Page</span>
                </v-tooltip>
                <label style="padding-top:22px">Lesson {{ lessonTitle }}</label>
              </v-layout>
              <v-list-group prepend-icon="menu" value="true" color="light-blue">
                <template v-slot:activator>
                  <v-list-item-title>Concepts</v-list-item-title>
                </template>
                <v-list-item v-for="(lesson, i) in lessons" :key="i" link>
                  <v-list-item-content @click="clickLesson(lesson)">
                    <v-list-item-title v-text="i + 1 + '. ' + lesson.title"></v-list-item-title>
                  </v-list-item-content>
                  <v-list-item-action>
                    <v-tooltip left>
                      <template v-slot:activator="{ on }">
                        <v-layout justify-space-around>
                          <v-btn icon small v-on="on" @click="lesson.bookmark = !lesson.bookmark">
                            <v-icon :color="lesson.bookmark ? 'orange' : 'grey'">bookmark</v-icon>
                          </v-btn>
                        </v-layout>
                      </template>
                      <span v-if="!lesson.bookmark">Not Bookmarked</span>
                      <span v-if="lesson.bookmark">Bookmarked</span>
                    </v-tooltip>
                  </v-list-item-action>
                </v-list-item>
              </v-list-group>
            </v-list>
          </v-card>
        </v-flex>
        <v-flex class="ml-2">
          <v-toolbar dark>
            <v-tooltip right>
              <template v-slot:activator="{ on }">
                <v-flex xs2>
                  <v-btn icon color="light-blue" v-on="on" @click="sidePanel=!sidePanel">
                    <v-icon>menu</v-icon>
                  </v-btn>
                </v-flex>
              </template>
              <span v-if="sidePanel">Hide Lessons</span>
              <span v-if="!sidePanel">Show Lessons</span>
            </v-tooltip>
            <v-spacer></v-spacer>
            <v-tooltip left>
              <template v-slot:activator="{ on }">
                <v-flex xs2>
                  <v-btn text target="_blank" v-on="on" href="https://github.com/zhongqi1112">
                    <span class="mr-2" style="color: dodgerblue;">Send Feedback</span>
                  </v-btn>
                </v-flex>
              </template>
              <span>Talk to Author</span>
            </v-tooltip>
          </v-toolbar>
          <v-layout column>
            <div id="lessonPlayer"></div>
            <v-layout align-center justify-start row fill-height>
              <span>Coding</span>
              <v-switch
                v-model="coding"
                color="primary"
                :label="coding ? 'Yes' : 'No'"
                class="ml-2"
              ></v-switch>
              <v-flex xs1 class="ml-4">
                <v-select v-model="language" :disabled="!coding" :items="languages" label="Lanuage"></v-select>
              </v-flex>
              <v-spacer></v-spacer>
              <v-tooltip left>
                <template v-slot:activator="{ on }">
                  <div>
                    <v-btn icon v-on="on" @click="createCustomTime">
                      <v-icon color="orange">book</v-icon>
                    </v-btn>
                  </div>
                </template>
                <span>Mark on Timeline</span>
              </v-tooltip>
              <v-flex xs7 v-for="(lesson, i) in lessons" :key="i" v-show="lesson.timeline" class="mr-4">
                <div :id="lesson.id"></div>
              </v-flex>
            </v-layout>
            <v-flex>
              <v-textarea v-show="!coding" filled label="Note"></v-textarea>
              <v-textarea v-show="coding" filled label="Code"></v-textarea>
            </v-flex>
            <v-layout justify-end row class="mb-4 mr-4">
              <v-btn color="success" :disabled="!coding" class="mr-4">Run</v-btn>
              <v-btn color="primary" :disabled="!coding" class="mr-4">Submit</v-btn>
              <v-btn color="primary">Download</v-btn>
            </v-layout>
          </v-layout>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import vis from 'vis'
import 'vis/dist/vis.css'
const _ = require('lodash')
const YTPlayer = require('yt-player')
var player = null
var timelineTemplate = _.template('<%= date %> <%= hour %>:<%= minute %>:<%= second %>')
var lessonsTimeline = [
  { id: 'fYMQ7toJwQ0', timeline: null },
  { id: 'iVe6Yx31V-w', timeline: null },
  { id: 'sknsduLepYc', timeline: null }
]

export default {
  mounted() {
    this.lessonTitle = this.lessons[0].title
    this.lessonId = this.lessons[0].id
    this.previousLessonId = this.lessons[0].id
    this.lessons[0].timeline = true
    player = new YTPlayer("#lessonPlayer")
    this.loadVideo(this.lessons[0].id)
    this.lessons.forEach(lesson =>{
      this.createTimeline(lesson)
    })
  },
  data: () => ({
    enterButtonLabel: 'Start',
    homePage: true,
    sidePanel: true,
    lessonTitle: '',
    lessonId: '',
    previousLessonId: '',
    lessons: [
      { title: 'Humans', bookmark: false, id: 'fYMQ7toJwQ0', timeline: false },
      { title: 'Computers', bookmark: false, id: 'iVe6Yx31V-w', timeline: false },
      { title: 'Interaction', bookmark: false, id: 'sknsduLepYc', timeline: false }
    ],
    coding: false,
    language: 'Python',
    languages: [
      'C++',
      'Java',
      'JS',
      'Python'
    ],
    todayDate: ''
  }),
  methods: {
    /**
      * @description select lesson on the lessons list
      * @param {object} lesson object of lesson
      */
    clickLesson: function (lesson) {
      this.lessonTitle = lesson.title
      this.previousLessonId = this.lessonId
      this.lessonId = lesson.id
      player.destroy()
      this.loadVideo()
      this.loadTimeline()
    },
    /**
      * @description load the YouTube video to player element
      */
    loadVideo: function () {
      player = new YTPlayer('#lessonPlayer', {
        height: '600',
        width: '100%',
        related: false
      })
      player.load(this.lessonId)
    },
    /**
      * @description create custom time on timeline
      */
    createCustomTime: function () {
      // set timeline window
      var videoTotalSeconds = Math.ceil(player.getDuration()) // in seconds
      var lessonTimelineObj = lessonsTimeline.find(lessonTimeline => lessonTimeline.id === this.lessonId)
      var videoMinute = parseInt(videoTotalSeconds / 60, 10)
      var videoSecond = parseInt(videoTotalSeconds % 60, 10) + 5
      videoMinute = videoMinute < 10 ? "0" + videoMinute : videoMinute
      videoSecond = videoSecond < 10 ? "0" + videoSecond : videoSecond
      lessonTimelineObj.timeline.setWindow(
        new Date(timelineTemplate({"date": this.todayDate, "hour": "00", "minute": "00", "second": "00"})),
        new Date(timelineTemplate({"date": this.todayDate, "hour": "00", "minute": videoMinute, "second": videoSecond})),
        { animation: true })
      // add custom time on timeline
      var currentSeconds = Math.ceil(player.getCurrentTime()) // in seconds
      var currentMinute = parseInt(currentSeconds / 60, 10)
      var currentSecond = parseInt(currentSeconds % 60, 10)
      lessonTimelineObj.timeline.addCustomTime(
        new Date(timelineTemplate({"date": this.todayDate, "hour": "00", "minute": currentMinute, "second": currentSecond})),
        currentSeconds, { editable: false })
    },
    /**
      * @description create timeline for each lesson
      * @param {object} lesson object of lesson
      */
    createTimeline: function (lesson) {
      // DOM element where the Timeline will be attached
      var container = document.getElementById(lesson.id);
      // Create a DataSet (allows two way data-binding)
      var items = new vis.DataSet([]);
      // Configuration for the Timeline
      var today = new Date()
      var yyyy = today.getFullYear()
      var mm = today.getMonth() + 1 // As January is 0
      var dd = today.getDate()
      dd = dd < 10 ? "0" + dd : dd
      mm = mm < 10 ? "0" + mm : mm
      this.todayDate = yyyy + "-" + mm + "-" + dd
      // Configuration for the Timeline
      var options = {
        showCurrentTime: false,
        moveable: false,
        min: timelineTemplate({"date": this.todayDate, "hour": "00", "minute": "00", "second": "00"}),
        max: timelineTemplate({"date": this.todayDate, "hour": "01", "minute": "00", "second": "00"})
      };
      // Create a Timeline
      var lessonTimelineObj = lessonsTimeline.find(lessonTimeline => lessonTimeline.id === lesson.id)
      lessonTimelineObj.timeline = new vis.Timeline(container, items, options)
      // timeline = new vis.Timeline(container, items, options);
    },
    /**
      * @description load timeline for current lesson
      */
    loadTimeline () {
      var preLessonTimelineObj = this.lessons.find(lesson => lesson.id === this.previousLessonId)
      var lessonTimelineObj = this.lessons.find(lesson => lesson.id === this.lessonId)
      preLessonTimelineObj.timeline = false
      lessonTimelineObj.timeline = true
    }
  }
}
</script>
