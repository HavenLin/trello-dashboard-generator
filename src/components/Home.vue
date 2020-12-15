<template>
<div id="layout" class="pure-g">
    <div class="sidebar pure-u-1 pure-u-md-1-4">
        <div class="header">
            <h4 class="brand-title">A Simple Trello Dashboard</h4>
            <h5 class="brand-tagline">Creating a dashboard using Trello</h5>
            <nav class="nav">
                <ul class="nav-list">
                    <li class="nav-item" >
                        <a class="pure-button" @click="clickTask">Task Panel</a>
                    </li>
                    <li class="nav-item">
                        <a class="pure-button" @click="clickLabel">Label Panel</a>
                    </li>
                </ul>
            </nav>
        </div>
    </div>

    <div class="content pure-u-1 pure-u-md-3-4">
          <!-- select boards-->
          <div class="posts">
            <section class="post">
              <h1 class="content-subhead">Choose Your Board</h1>
                  <b-row class="justify-content-left">
                    <select id='boards-select' v-model="selectedBoard" @change="getSelectedObj">
                      <option id='boardName'
                        v-for="board in boards"
                        :key="board.id"
                        :value="board"
                      >
                        {{ board.name }}
                      </option>
                    </select>
                  </b-row>
            </section>
          </div>

          <div v-if='showTaskPanel'>
             <div class="posts">
                <h1 class="content-subhead">Choose List</h1>
                <section class="post">
                    <b-row class="justify-content-left">
                      <b-col>
                        <b-button v-b-toggle="'collapse-task-cards'" class="m-1">Avaliable Task Cards</b-button>
                        <b-collapse id="collapse-task-cards" >
                          <b-card>
                            <b-form-group id="checkboxes-task-cards">
                              <b-form-checkbox
                                v-for="list in lists"
                                v-model="selectedList"
                                :key="list.id"
                                :value="list.id"
                                name="task-cards"
                                @change="getTrelloCards"
                              >
                                {{ list.name }}
                              </b-form-checkbox>
                            </b-form-group>
                          </b-card>
                        </b-collapse>
                      </b-col>
                    </b-row>
                </section>
             </div>

             <div class="posts">
                <h1 class="content-subhead">Choose Date Range</h1>
                <section class="post">
                  <b-row>
                    <b-col md=12>
                      <chart :trelloCards="trelloCards"></chart>
                    </b-col>
                  </b-row>
                </section>
              </div>
          </div>

          <div v-if='showLabelPanel'>
            <div class="posts">
              <h1 class="content-subhead">Choose Date Range</h1>
              <section class="post">
                <b-row>
                  <b-col md=12>
                    <chart :trelloCards="trelloCardsByLabels"></chart>
                  </b-col>
                </b-row>
              </section>
            </div>
          </div>

          <div class="footer">
            <div class="pure-menu pure-menu-horizontal">
              <ul>
                <li class="pure-menu-item"><a href="TBD" class="pure-menu-link">About</a></li>
                <li class="pure-menu-item"><a href="https://github.com/HavenLin/TrelloReportGenerator" class="pure-menu-link">GitHub</a></li>
              </ul>
            </div>
          </div>

    </div>
</div>
</template>

<script>
import Chart from '@/components/Chart'

export default {
  name: 'Home',
  components: {
    Chart
  },
  props: {
    trelloCards: new Map(),
    trelloCardsByLabels: new Map()
  },
  data: function () {
    return {
      err: '',
      showTaskPanel: true,
      showLabelPanel: false,
      selectedBoard: {},
      selectedList: [],
      boards: [],
      labelsMap: new Map(),
      lists: [],
      members: new Map()
      // customFields: [],
    }
  },
  created: function () {
    // load Boards
    window.Trello.get(
      'members/me/boards',
      (response) => {
        this.boards = response
        console.log('loadBoards')
        console.log(this.boards)
      },
      authenticationFailure
    )
  },
  methods: {
    getSelectedObj: function () {
      this.trelloCards = new Map()
      var id = this.selectedBoard.id
      // load lists
      window.Trello.get(
        'boards/' + id + '/lists',
        (response) => {
          this.lists = response
        },
        authenticationFailure
      )
      // load Members
      window.Trello.get(
        'boards/' + id + '/members',
        (response) => {
          response.forEach(element => {
            this.members.set(element.id, element.fullName)
            console.log('load members')
          })
        },
        authenticationFailure
      )
      // load lables
      window.Trello.get(
        'boards/' + id + '/labels',
        (response) => {
          response.forEach(element => {
            this.labelsMap.set(element.id, element.name)
            console.log('load labels')
          })
        },
        authenticationFailure
      )
    },
    getTrelloCards: function () {
      var allCards = new Map()
      this.trelloCards = new Map()
      window.Trello.get(
        'boards/' + this.selectedBoard.id + '/cards/all?fields=dateLastActivity,idList,idLabels,name,idMembers,labels,closed',
        (response) => {
          // convert response json array to map
          response.forEach(element => {
            var cardLists = allCards.get(element.idList)
            if (cardLists == null) {
              cardLists = []
            }
            cardLists.push(element)
            this.sortByDate(cardLists)
            allCards.set(element.idList, cardLists)
          })

          // remove cards which unselected
          var selectedCards = []
          this.selectedList.forEach(element => {
            if (allCards.has(element)) {
              selectedCards.push(allCards.get(element))
            }
          })

          // generate chart series
          var userMap = new Map()
          selectedCards.forEach(cardLists => {
            cardLists.forEach(cardDetail => {
              cardDetail.idMembers.forEach(idMember => {
                var userData = {}
                userData.name = this.members.get(idMember)
                userData.dateLastActivity = this.yyyymmddFormate(cardDetail.dateLastActivity)
                var userMapByDate = userMap.get(userData.name)
                if (userMapByDate == null) {
                  userMapByDate = new Map()
                }
                var userCardList = userMapByDate.get(userData.dateLastActivity)
                if (userCardList == null) {
                  userCardList = []
                }
                userCardList.push(userData)
                userMapByDate.set(userData.dateLastActivity, userCardList)
                userMap.set(userData.name, userMapByDate)
              })
            })
          })
          this.trelloCards = userMap
        },
        authenticationFailure
      )
    },
    getTrelloCardsByLabels: function () {
      this.trelloCardsByLabels = new Map()
      window.Trello.get(
        'boards/' + this.selectedBoard.id + '/cards/all?fields=dateLastActivity,idList,idLabels,name,idMembers,labels,closed',
        (response) => {
          // convert response json array to map
          var userMap = new Map()
          response.forEach(element => {
            element.idLabels.forEach(idLabel => {
              var userData = {}
              userData.name = this.labelsMap.get(idLabel)
              userData.dateLastActivity = this.yyyymmddFormate(element.dateLastActivity)
              var userMapByDate = userMap.get(userData.name)
              if (userMapByDate == null) {
                userMapByDate = new Map()
              }
              var userCardList = userMapByDate.get(userData.dateLastActivity)
              if (userCardList == null) {
                userCardList = []
              }
              userCardList.push(userData)
              this.sortByDate(userCardList)
              userMapByDate.set(userData.dateLastActivity, userCardList)
              userMap.set(userData.name, userMapByDate)
            })
            this.trelloCardsByLabels = userMap
          })
        },
        authenticationFailure
      )
    },
    yyyymmddFormate: function (date) {
      var x = new Date(date)
      var y = x.getFullYear().toString()
      var m = (x.getMonth() + 1).toString()
      var d = x.getDate().toString();
      (d.length === 1) && (d = '0' + d);
      (m.length === 1) && (m = '0' + m)
      var yyyymmdd = y + '-' + m + '-' + d
      return yyyymmdd
    },
    sortByDate: function (dateList) {
      dateList.sort((previous, current) => {
        var a = new Date(previous.dateLastActivity)
        var b = new Date(current.dateLastActivity)
        return a > b ? 1 : a < b ? -1 : 0
      })
    },
    clickTask: function () {
      this.showTaskPanel = true
      this.showLabelPanel = false
    },
    clickLabel: function () {
      this.showTaskPanel = false
      this.showLabelPanel = true
      this.getTrelloCardsByLabels()
    }
  },
  watch: {
    trelloCards () {
      console.log(this.trelloCards)
    }
  }
}
var authenticationSuccess = function () {
  console.clear()
  console.log('Successful authentication')
}

var authenticationFailure = function () {
  console.log('Failed authentication')
}

window.Trello.authorize({
  type: 'popup',
  name: 'Getting Started Application',
  scope: {
    read: 'true',
    write: 'false' },
  expiration: 'never',
  success: authenticationSuccess,
  error: authenticationFailure
})
</script>

<style scoped>
.nav{
  float:right;
}
</style>
