<script>
export default {
  data() {
    return {
      categories: ["Cat 1", "Cat 2", "Cat 3", "Cat 4"],
      values: [100, 200, 300, 400, 500],
      players: [
        { id: 1, score: 0 },
        { id: 2, score: 0 },
        { id: 3, score: 0 }
      ],
      board: {},
      currentQuestion: null,
      currentPlayer: 1
    }
  },
  components: {
  },
  methods: {
    selectQuestion(cat, value) {
      console.log(`Selected ${cat} for $${value}`);
    },
    fetchQuestionsForCategory(catName, catId) {
      console.log(`TODO: Fetch questions for ${catName} (id: ${catId})`);
    }
  },
  mounted() {
    fetch("https://opentdb.com/api_category.php")
      .then(res => res.json())
      .then(data => {
        let allCategories = data.trivia_categories;
        let chosen = [];
        for (let i = 0; i < 4; i++) {
          let randIndex = Math.floor(Math.random() * allCategories.length);
          chosen.push(allCategories[randIndex]);
        }

        this.categories = chosen.map(function(c) {
          return c.name;
        });

        chosen.forEach(function(cat) {
          this.fetchQuestionsForCategory(cat.name, cat.id);
        }.bind(this));

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
  <table class="board">
    <thead>
      <tr>
        <th v-for="(cat, index) in categories" :key="index">
          {{ cat }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="value in values" :key="value">
        <td v-for="(cat, index) in categories" :key="index" @click="selectQuestion(cat, value)">
          ${{ value }}
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

</style>
