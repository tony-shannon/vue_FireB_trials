<template>
      <div id="app">
          <Notebook @change-page="changePage" @new-page="newPage" :pages="pages" :activePage="index" />
          <Page @save-page="savePage" @delete-page="deletePage" :page="pages[index]" />
      </div>
    </template>

    <script>
    import Notebook from './components/Notebook'
    import Page from './components/Page'
    import Firebase from 'firebase'

    var database = Firebase.initializeApp({
      apiKey: "AIzaSyC2DR-dhFDL6z_oKZlXGxW9jhQYnxfWsqU",
      authDomain: "notebook-1a.firebaseapp.com",
      databaseURL: "https://notebook-1a.firebaseio.com",
      projectId: "notebook-1a",
      storageBucket: "notebook-1a.appspot.com",
      messagingSenderId: "292619954408",
      appId: "1:292619954408:web:6fa6b2abd180dd60ed015d"
    }).database().ref();

    export default {
      name: 'app',
      components: {
        Notebook,
        Page
      },
      data: () => ({
        pages: [],
        index: 0
      }),
      mounted() {
      database.once('value', (pages) => {
        pages.forEach((page) => {
          this.pages.push({
            ref: page.ref,
            title: page.child('title').val(),
            content: page.child('content').val()
          })
        })
      })
      },
      methods: {
        newPage () {
          this.pages.push({
            title: '',
            content: ''
          })
          this.index = this.pages.length - 1
        },
        changePage (index) {
          this.index = index
        },
        savePage () {
           // nothing as of yet
          var page = this.pages[this.index]      // added for firebase
          if (page.ref) {
                this.updateExistingPage(page)
              } else {
                this.insertNewPage(page)        // added for firebase
              }
        },
        updateExistingPage (page) {           //added for firebase
              page.ref.update({
                title: page.title,
                content: page.content
              })
            },
        insertNewPage (page) {                //added for firebase
            page.ref = database.push(page)
          },
        deletePage () {
          var ref = this.pages[this.index].ref  // for firebase
          ref && ref.remove()                   // for firebase
          this.pages.splice(this.index, 1)
          this.index = Math.max(this.index - 1, 0)
        }
      }
    }
    </script>

    <style>
    html, body, #app {
        height: 100%;
    }

    body {
        margin: 0;
    }

    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        display: flex;
        flex-direction: row;
    }
    </style>
