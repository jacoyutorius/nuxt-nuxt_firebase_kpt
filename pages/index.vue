<template>
  <section class="container">
    <section id="new_board" class="section">
      <div class="field">
        <div class="control has-icons-left has-icons-right">
          <input class="input is-large" v-bind:class="errorActive ? 'is-danger' : ''" type="text" v-model="boardTitle" placeholder="input new Board name.">
          <p v-if="errorActive" class="help is-danger">{{ errorMessage }}</p>
          <span class="icon is-small is-left">
            <i class="fa fa-table"></i>
          </span>
        </div>
      </div>
      <div class="links">
        <a target="_blank"
          class="button--green"
          @click="createBoard">Create New Board</a>
      </div>
    </section>
    <section id="boards" class="section">
      <h2 class="subtitle is-5">Recent Boards</h2>

      <table class="table">
        <tbody>
          <tr v-for="(key, index) in Object.keys(boards)" v-bind:key="index">
            <th><a @click="goBoardPage(key)"><i class="fa fa-table fa-1"></i>  {{ boards[key].title }}</a></th>
            <th><small>{{ parseTimestamp(boards[key].timestamp) }}</small></th>
            <!-- <th><span class="icon is-small" @click="deleteBoard(key)"><i class="fa fa-trash"></i></span></th> -->
          </tr>
        </tbody>
      </table>
    </section>
  </section>
</template>

<script>
import firebase from "~/plugins/firebase"

export default {
  data: function(){
    return {
      boardTitle: "",
      boards: {},
      errorMessage: ""
    }
  },
  computed: {
    errorActive: function(){
      return this.errorMessage.length > 0
    }
  },
  mounted: function(){
    this.boards = [];
    var boardsRef = firebase.database().ref("boards");
    boardsRef.on("value", function(snapshot){
      // 更新日の降順にするためにreverseする
      this.boards = Object.keys(snapshot.val())
        .reverse()
        .reduce((obj, key)=>{ 
          obj[key] = snapshot.val()[key];
          return obj;
          }, {});
    }.bind(this));
  },
  methods: {
    parseTimestamp(timestamp) {
      const d = new Date(timestamp);
      const week = "日月火水木金土";

      const hours = String(d.getHours()).padStart(2, "0");
      const minutes = String(d.getMinutes()).padStart(2, "0");
      const seconds = String(d.getSeconds()).padStart(2, "0");

      return `${d.getFullYear()}/${d.getMonth()+1}/${d.getDate()}(${week[d.getDay()]})  ${hours}:${minutes}:${seconds}`;
    },
    createBoard() {
      if (this.boardTitle.length <= 0){
        this.errorMessage = "タイトルを入力してください！"
        return;
      }

      var key = firebase.database().ref().child('boards').push().key;
      firebase.database().ref("boards/" + key).set({
        title: this.boardTitle,
        timestamp: Date.now()
      })
      this.boardTitle = "";
      this.goBoardPage(key);
    },
    boardPageUrl(board_key) {
      return "/boards/" + board_key;
    },
    deleteBoard(board_key) {
      if (window.confirm(this.boards[board_key].title + "を削除しますか？")){
        firebase.database().ref("boards/" + board_key).remove();
      }
    },
    goBoardPage(key){
      // boardレイアウト側でタイトルを表示したいので、vue-routerのqueryパラムにタイトルを入れて渡す
      this.$router.push({path: "/board/", query: {id: key, title: this.boards[key].title} });
    }
  }
}
</script>

<style>
#boards > h2.subtitle {
  margin-bottom: 0.5em;
}
</style>
