<script>
export default {
  data() {
    return {
      categories: ["Cat 1", "Cat 2", "Cat 3", "Cat 4"],
      categoryIds: [],
      values: [100, 200, 300, 400, 500],
      players: [
        { id: 1, score: 0 },
        { id: 2, score: 0 },
        { id: 3, score: 0 }
      ],
      board: {},
      currentQuestion: null,
      currentPlayer: 1,
      loading: true,
      sessionToken: null,
      badCategories: [13,21,27,30,32]
    }
  },
  components: {
  },
  methods: {
    async selectQuestion(cat, value) {
      if (this.currentQuestion) return;
      if (!this.board[cat]) {
        this.board[cat] = [];
      }

      let existing = this.board[cat].find(q => q.value === value);
      if (existing) {
        if (!existing.usedBy) {
          this.currentQuestion = existing;
        }
        return;
      }

      let catIndex = this.categories.indexOf(cat);
      let catId = this.categoryIds[catIndex];

      let difficulty = "easy";
      if (value >= 300 && value < 500) difficulty = "medium";
      if (value === 500) difficulty = "hard";

      let url = `https://opentdb.com/api.php?amount=1&category=${catId}&difficulty=${difficulty}&type=boolean&encode=base64&token=${this.sessionToken}`;
      try {
        let res = await fetch(url);
        let data = await res.json();

        if (data.response_code === 0 && data.results.length > 0) {
          let qData = data.results[0];
          let questionObj = {
            value,
            question: atob(qData.question),
            correct: atob(qData.correct_answer),
            usedBy: null
          };

          this.board[cat].push(questionObj);
          this.currentQuestion = questionObj;
        } else {
          let questionObj = {
            value,
            question: "No question available for this slot.",
            correct: "True",
            usedBy: null
          };
          this.board[cat].push(questionObj);
          this.currentQuestion = questionObj;
        }

      } catch (err) {
        console.error("Error fetching question:", err);
      }

    },
    
    answer(choice) {
      if (!this.currentQuestion) return;

      let correctAnswer = this.currentQuestion.correct;
      let player = this.players[this.currentPlayer - 1];
      let value = this.currentQuestion.value;

      let wasCorrect = (choice ? "True" : "False") === correctAnswer;
      if(wasCorrect) {
        player.score += value;
        alert("Correct!");
      } else {
        player.score -= value;
        alert("Incorrect!");
      }

      this.currentQuestion.usedBy = `P${this.currentPlayer}`;

      if(!wasCorrect) {
        this.currentPlayer = (this.currentPlayer % this.players.length) + 1;
      }
      this.currentQuestion = null;
    },

    getQuestion(cat, value) {
      return this.board[cat]?.find(q => q.value === value);
    }
  },
  mounted() {
    fetch("https://opentdb.com/api_token.php?command=request")
      .then(res => res.json())
      .then(tokenData => {
        this.sessionToken = tokenData.token;
        return fetch("https://opentdb.com/api_category.php");
      })
      .then(res => res.json())
      .then(data => {
        let allCategories = data.trivia_categories;
        let chosen = [];
        for (let i = 0; i < 4; i++) {
          let randIndex = Math.floor(Math.random() * allCategories.length);
          let picked = allCategories[randIndex];
          while (this.badCategories.includes(picked.id)) {
            randIndex = Math.floor(Math.random() * allCategories.length);
            picked = allCategories[randIndex];
          }
          
          allCategories.splice(randIndex, 1);
          chosen.push(picked);
        }

        this.categories = chosen.map(c => c.name);
        this.categoryIds = chosen.map(c => c.id);
        this.loading = false;

      }).catch((function(err) {
        console.error("Error fetching categories:", err);
      }));
  }
}
</script>

<template>
  <div id="app">
  <table class="jeopardy">
    <thead>
      <tr>
        <h1>Jeopardy!</h1>
      </tr>
    </thead>
    <tbody>
      <tr v-for="player in players" :key="player.id">Player {{ player.id }}: ${{ player.score }}</tr>
    </tbody>
  </table>
  <br>
  <table class="board" v-if="!loading">
    <thead>
      <tr>
        <th v-for="(cat, index) in categories" :key="index">
          {{ cat }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="value in values" :key="value">
        <td v-for="(cat, index) in categories" :key="index" @click="!getQuestion(cat, value)?.usedBy && selectQuestion(cat, value)">
        <span v-if="!getQuestion(cat, value)">
          ${{ value }}
        </span>
        <span v-else-if="!getQuestion(cat, value).usedBy">
          ${{ value }}
        </span>
        <span v-else>
          {{ getQuestion(cat, value).usedBy }}
        </span>
        </td>
      </tr>
    </tbody>
  </table>
  <br>

  <div v-if="currentQuestion">
    <h3>{{ currentQuestion.category }} - ${{ currentQuestion.value }}</h3>
    <p>{{ currentQuestion.question }}</p>
    <button @click="answer(true)">True</button>
    <button @click="answer(false)">False</button>
  </div>
  <div v-if="loading" class="loading">Loading game...</div>
  </div>
</template>

<style>
.scoreboard {
  margin-bottom: 20px;
  font-weight: bold;
}

.board {
  margin: 0 auto;
  border-collapse: collapse
}

.board th,
.board td {
  border: 2px solid black;
  padding: 15px;
  min-width: 100px;
  text-align: center;
  cursor: pointer;
}

.loading {
  margin: 20px;
  font-weight: bold;
  color: #ffcc00;
}

</style>
