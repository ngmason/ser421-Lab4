<script>
import { BAD_CATEGORIES, QUESTION_VALUES, NUM_CATEGORIES, DIFFICULTY_MAP, API_BASE, API_CATEGORY_URL, API_TOKEN_URL } from './config.js';
export default {
  data() {
    return {
      categories: [],
      categoryIds: [],
      values: QUESTION_VALUES,
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
      badCategories: BAD_CATEGORIES,
      gameOver: false,
      winner: null,
      feedback: null,
      feedbackCorrect: null,
      fetching: false,
      doubleJeopardy: false,
      wager: null,
      pendingWager: null
    }
  },
  components: {
  },
  methods: {
    async selectQuestion(cat, value) {
      if (this.currentQuestion || this.fetching) return;
      this.feedback = null;
      this.fetching = true;

      if (!this.board[cat]) {
        this.board[cat] = [];
      }

      let existing = this.board[cat].find(q => q.value === value);
      if (existing) {
        if (!existing.usedBy) {
          this.currentQuestion = existing;
        }
        this.fetching = false;
        return;
      }

      let catIndex = this.categories.indexOf(cat);
      let catId = this.categoryIds[catIndex];
      const difficulty = DIFFICULTY_MAP[value];

      const url = `${API_BASE}?amount=1&category=${catId}&difficulty=${difficulty}&type=boolean&encode=base64&token=${this.sessionToken}`;
      
      try {
        let res = await fetch(url);
        let data = await res.json();

        if (data.response_code === 0 && data.results.length > 0) {
          let qData = data.results[0];
          let questionObj = {
            category: cat,
            value,
            question: atob(qData.question),
            correct: atob(qData.correct_answer),
            usedBy: null
          };

          this.board[cat].push(questionObj);
          this.currentQuestion = questionObj;

          if (Math.random() < 0.1) {
            this.doubleJeopardy = true;
            this.feedback = `Double Jeopardy! Player ${this.currentPlayer}, enter your wager.`

            let player = this.players[this.currentPlayer - 1];
            if (player.score < value) {
              this.wager = value;
              this.feedback = `Double Jeopardy! Player ${this.currentPlayer}, your balance is too low. Wager set to $${value}.`;
            } else {
              this.wager = null;
              this.pendingWager = null;
            }
          } else {
            this.doubleJeopardy = false;
          }
        } else {
          this.feedback = "Too many requests! Please wait a moment and try again."
        }

      } catch (err) {

        console.error("Error fetching question:", err);
        this.feedback = "Error loading questions. please try again."

      } finally {

        this.fetching = false;
      }

    },
    
    answer(choice) {
      if (!this.currentQuestion) return;

      let correctAnswer = this.currentQuestion.correct;
      let player = this.players[this.currentPlayer - 1];
      let value = this.doubleJeopardy ? this.wager : this.currentQuestion.value;

      let wasCorrect = (choice ? "True" : "False") === correctAnswer;
      let nextPlayer = this.currentPlayer;

      if(wasCorrect) {
        player.score += value;
        this.feedback = `Correct! Player ${this.currentPlayer}, pick the next question.`;
        this.feedbackCorrect = true;
      } else {
        player.score -= value;
        nextPlayer = (this.currentPlayer % this.players.length + 1);
        this.feedback = `Incorrect! Player ${nextPlayer}, it's your turn.`;
        this.feedbackCorrect = false;
      }

      this.currentQuestion.usedBy = `P${this.currentPlayer}`;
      this.currentQuestion.wasCorrect = wasCorrect;

      this.currentPlayer = nextPlayer;
      this.currentQuestion = null;
      this.doubleJeopardy = false;
      this.wager = null;

      let allQuestions = this.values.length * this.categories.length;
      let usedQuestions = Object.values(this.board).flat().filter(q => q.usedBy).length;
      if(usedQuestions === allQuestions) {
        this.gameOver = true;
        this.feedback = null;
        let maxScore = Math.max.apply(null, this.players.map(p => p.score));
        let winners = this.players.filter(p => p.score === maxScore);
        if(winners.length > 1) {
          this.winner = "It's a tie! Players " + winners.map(p => p.id).join(", ") + " with $" + maxScore;  
        } else {
          this.winner = `Player ${winners[0].id} wins with $${maxScore}!`;
        }
      }

    },

    getQuestion(cat, value) {
      return this.board[cat]?.find(q => q.value === value);
    },

    confirmWager() {
      let player = this.players[this.currentPlayer - 1];
      let maxWager = Math.max(player.score, this.currentQuestion.value);

      if(!this.pendingWager || this.pendingWager < 1 || this.pendingWager > maxWager) {
        this.feedback = `Invalid wager! You can wager up to $${maxWager}`;
        return;
      }

      this.wager = this.pendingWager;
      this.feedback = `Wager set to $${this.wager}. Now answer the question!`;
    }

  },
  mounted() {
    fetch(API_TOKEN_URL)
      .then(res => res.json())
      .then(tokenData => {
        this.sessionToken = tokenData.token;
        return fetch(API_CATEGORY_URL);
      })
      .then(res => res.json())
      .then(data => {
        let allCategories = data.trivia_categories;
        let chosen = [];
        for (let i = 0; i < NUM_CATEGORIES; i++) {
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
  <div class="game-container">
    <div class="scoreboard">
      <h1>Jeopardy!</h1>
      <div v-for="player in players" :key="player.id" class="player-score">
        Player {{ player.id }}: ${{ player.score }}
      </div>
    </div>

    <table class="board" v-if="!loading">
      <thead>
        <tr>
          <th v-for="(cat, index) in categories" :key="index">{{ cat }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="value in values" :key="value">
          <td
            v-for="(cat, index) in categories"
            :key="index"
            @click="!currentQuestion && !getQuestion(cat, value)?.usedBy && selectQuestion(cat, value)"
          >
            <span v-if="!getQuestion(cat, value)">${{ value }}</span>
            <span v-else-if="!getQuestion(cat, value).usedBy">${{ value }}</span>
            <span
              v-else
              :class="getQuestion(cat,value).wasCorrect ? 'correct' : 'incorrect'"
            >
              {{ getQuestion(cat, value).usedBy }}
            </span>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="question-area" v-if="currentQuestion">
      <h3>
        Player {{ currentPlayer }}: {{ currentQuestion.category }} -
        ${{ currentQuestion.value }}
      </h3>

      <div v-if="doubleJeopardy && wager === null" class="wager-input">
      <p>Double Jeopardy! Player {{ currentPlayer }}, enter your wager.</p>
        <label>
          Wager:
          <input type="number" v-model.number="pendingWager" min="1" :max="Math.max(players[currentPlayer - 1].score, currentQuestion.value)" />
        </label>
        <button @click="confirmWager">Confirm Wager</button>
      </div>
      <div v-else>
        <p>{{ currentQuestion.question }}</p>
        <div class="buttons">
          <button @click="answer(true)">True</button>
          <button @click="answer(false)">False</button>
        </div>
      </div>
    </div>

    <p
      v-if="feedback && !doubleJeopardy"
      :class="feedback.includes('Correct') ? 'correct feedback' : 'incorrect feedback'"
    >
      {{ feedback }}
    </p>
    <p v-if="doubleJeopardy && (wager === null || feedback.includes('Wager set'))" class="double-jeopardy">
      {{ feedback }}
    </p>

    <div v-if="gameOver" class="winner">
      {{ winner }}
    </div>

    <p v-if="fetching" class="loading">Loading question... please wait!</p>
  </div>
  </div>
</template>
<style>
#app {
  max-width: 900px;
  min-width: 600px;
  margin: 0 auto;
  text-align: center;
  color: #fff;
  font-family: Arial, sans-serif;
}

.game-container {
  width: 800px;
  margin: 0 auto;
}

.scoreboard {
  margin-bottom: 20px;
}

.player-score {
  margin: 5px 0;
}

.board {
  margin: 20px auto;
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

.question-area {
  margin-top: 30px;
}

.buttons {
  margin-top: 10px;
}

button {
  margin: 5px;
  padding: 8px 15px;
  font-size: 1em;
  cursor: pointer
}

.feedback {
  margin-top: 15px;
  font-size: 1.2em;
}

.loading {
  margin: 20px;
  font-weight: bold;
  color: #ffcc00;
}

.winner {
  margin-top: 30px;
  font-size: 1.5em;
  font-weight: bold;
  color: #4CAf50
}

.correct {
  color: #4CAF50;
  font-weight: bold;
}

.incorrect {
  color: #F44336;
  font-weight: bold;
}

.wager-input {
  margin: 10px 0;
  font-size: 1.1em;
}

.wager-input input {
  width: 80px;
  margin-left: 8px;
  padding: 4px;
}

.double-jeopardy {
  color: #ffcc00;
  font-weight: bold;
  margin-top: 15px;
  font-size: 1.2em;
}
</style>
