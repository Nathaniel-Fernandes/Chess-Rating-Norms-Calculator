<template>
  <div class='container'>
    <h2 style="padding-bottom: 10px;">USCF Rating and Norms Calculator</h2>

    <div class="info-inputs">
      <!-- rounds -->
      <div class="input">
        <span class="row-left">Rounds: {{ rounds }}</span>
        <input
          style="width: 50px;"
          type='number'
          v-model.number="rounds"
          @change="setResultsFalse"
        />
      </div>

      <!-- score -->
      <div class="input">
        <span class="row-left">Total Score: {{score}} </span>
        <input
          style="width: 50px;"
          type='number'
          v-model.number="score"
          min='0' step='1'
          @change="setResultsFalse"
        />
      </div>

      <!-- old rating -->
      <div class="input">
        <span class="row-left">Current Rating: {{current_rating}} </span>
        <input
          style="width: 50px;"
          type='number'
          v-model.number="current_rating"
          min='0' step='1'
          @change="setResultsFalse"/>
      </div>

      <!-- prior games -->
      <div class="input">
        <span class="row-left"># Prior Games: {{ prior_games }}</span>
        <input
          style="width: 50px;"
          type='number'
          v-model.number="prior_games"
          min='0' step='1'
          @change="setResultsFalse"/>
      </div>
    </div>

    <div v-bind:class="{ _green: message==='RICKROLL'}" class="row message" v-if="message">
      <span v-html='messageOptions[message]'></span>
    </div>

    <div class="ratings" v-if="rounds">
      <input
        v-for="i in rounds"
        :key="i"
        type="number"
        v-model.number="ratings[i-1]"
        @change="setResultsFalse"
    />
    </div>

    <button
      class="calculate"
      v-if="!message && !ratings.includes(undefined) && ratings.length && ratings.length === rounds"
      @click="calculateRating(), setResultsTrue()">Calculate</button>

    <div class="results" v-if="showResults">
      <span>
        <span class="bold">Performance:</span> {{performanceRating}}
      </span>
      <br />
      <span>
        <span class="bold">Bonus Points:</span> {{bonus}}
      </span>
      <br />
      <span class="bold">Final Rating: {{finalRating}}
        <span :style="{color: ratingDeltaSign === '+' ? 'green' : 'red'}">
          ({{ratingDeltaSign}}{{finalRating - current_rating}})
        </span>
      </span>
      <br />
      <h4 class="norms-table-h4">Did you earn a norm?</h4>
      <table class="norms-table">
        <thead>
          <tr>
            <th v-for="norm in normCategories" :key="norm">
              {{ norm }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td
              class="bold"
              :style="{color: n === 1 ? 'green' : 'red'}"
              v-for="(n, i) in calculateNorm" :key="i">
              {{n === 1 ? 'Yes!' : 'No'}}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'App',
  data() {
    return {
      score: 0,
      rounds: 1,
      prior_games: 50,
      current_rating: 0,
      finalRating: 0,
      performanceRating: 0,
      bonus: 0,
      showResults: false,
      // ratingChange: 0,
      normCategories: [1200, 1400, 1600, 1800, 2000, 2200, 2400],
      ratings: [],
      messageOptions: Object.freeze({
        NEGATIVE: '<span>No negative numbers allowed!</span>',
        ROUNDS_DECIMAL: '<span>Rounds must be an integer number greater than 0</span>',
        SCORE_DECIMAL: '<span>Score cannot have a decimal number other than .5</span>',
        SCORE_BIGGER_ROUNDS: '<span>Score cannot be greater than the # of rounds! :)</span>',
        RICKROLL: "<a href='https://youtu.be/dQw4w9WgXcQ' rel='noopener nofollow' target='_blank'>Do not click me ;)</a>",
      }),
    };
  },
  computed: {
    // http://www.glicko.net/ratings/titles.pdf
    calculateNorm() {
      if (this.rounds < 4) {
        return new Array(this.normCategories.length).fill(0);
      }
      let deltaY : number;
      const NormWinningExpectancy = this.normCategories.map((normLvl) => {
        const tempArr = this.ratings
          .map((oppRating) => {
            deltaY = normLvl - oppRating;
            let exp;

            if (deltaY <= -400) {
              exp = 0;
            } else if (deltaY <= 0) {
              exp = 0.5 + deltaY / 800;
            } else if (deltaY <= 200) {
              exp = 0.5 + deltaY / 400;
            } else {
              exp = 1;
            }

            return exp;
          })
          .reduce((a, b) => a + b, 0);

        return tempArr;
      });

      const earnedNorms = NormWinningExpectancy.map((perf) => {
        if (this.score - perf > 1) {
          return 1;
        }

        return 0;
      });

      return earnedNorms;
    },

    ratingDeltaSign() {
      if (this.finalRating - this.current_rating > 0) {
        return '+';
      }

      return '';
    },

    message() {
      if (this.current_rating === 420 || this.current_rating === 69) {
        return 'RICKROLL';
      }
      if (this.score < 0 || this.rounds < 0 || this.prior_games < 0 || this.current_rating < 0) {
        return 'NEGATIVE';
      }
      if (this.score > this.rounds) {
        return 'SCORE_BIGGER_ROUNDS';
      }
      if (this.rounds % 1 !== 0) {
        return 'ROUNDS_DECIMAL';
      }
      if (!((this.score * 2) % 2 in [0, 1])) {
        return 'SCORE_DECIMAL';
      }

      return null;
    },
  },

  methods: {
    // check_num(event: Event) {
    //   const num : number = parseInt((event.target as HTMLInputElement).value, 10);
    //   (event.target as HTMLInputElement).value = Math.floor(num).toString();
    //   console.log(num);
    // },
    setResultsTrue() {
      this.showResults = true;
    },
    setResultsFalse() {
      this.showResults = false;
    },

    logRatings() {
      console.log(this.ratings);
      console.log(this.ratings.includes(undefined));
    },

    calculateRating() {
      // USCF official ratings calculation: http://www.glicko.net/ratings/approx.pdf
      // This is not pretty but it works XD

      const ratingsAvg = this.ratings.reduce((a, b) => a + b, 0) / this.ratings.length;
      if (this.prior_games <= 8) {
        this.finalRating = (this.prior_games * this.current_rating + this.rounds * ratingsAvg
                          + (this.rounds - this.score) * 400) / (this.prior_games + this.rounds);
      } else {
        const adjustedPriorGames = (this.prior_games > 50) ? 50 : this.prior_games;
        const nR = (this.current_rating < 2355)
          ? 50 / Math.sqrt(0.662 + 0.00000739 * (2569 - this.current_rating) ** 2) : 50;

        const effPriorGames = Math.min(nR, adjustedPriorGames);

        const K = 800 / (effPriorGames + this.rounds);

        const WinningExpectancyArr = this.ratings.map((oppRating) => {
          const i = 1 / (10 ** ((this.current_rating - oppRating) / -400) + 1);
          return i;
        });

        const E = WinningExpectancyArr.reduce((a, b) => a + b, 0);

        const tempBonus = (this.rounds > 2)
          ? K * (this.score - E) - 14 * Math.sqrt(Math.max(this.rounds, 4))
          : 0;
        this.bonus = tempBonus > 0 ? Number(tempBonus.toFixed(1)) : 0;

        this.finalRating = this.current_rating + K * (this.score - E) + this.bonus;

        this.performanceRating = ratingsAvg
          + (400 * (Math.floor(this.score) - Math.floor(this.rounds - this.score))) / this.rounds;
        this.performanceRating = Math.round(this.performanceRating);

        // console.log(adjustedPriorGames, nR, effPriorGames);
        // console.log(K);
        // console.log(WinningExpectancyArr, E);
        // console.log(K * (this.score - E), 14 * Math.sqrt(Math.max(this.rounds, 4)));
        // console.log(this.bonus);
      }

      this.finalRating = Math.round(this.finalRating);
    },
  },
});
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

h2 {
  font-size: 24px;
}

span, td, th, p {
  font-size: 16px;
}

.results {
  margin-top: 10px;
}
.norms-table-h4 {
  margin-top: 10px;
}
.norms-table {
  border-spacing: 0;
}

.norms-table td, .norms-table th {
  padding: 3px 6px;
  border: 1px solid #000;
  text-align: center;
}

.bold {
  font-weight: bold;
}
.info-inputs {
  display: grid;
  grid-template-columns: 50% 50%;
  column-gap: 10px;
  row-gap: 5px;
}

.input>input {
  float: right;
}

.row {
  display: flex;
}
.row-left {
  min-width: 100px;
  padding-right: 10px;
}

button.calculate {
  /* margin: 10px 0; */
  padding: 10px;
}

.message {
  height: 50px;
  margin: 10px 0;
  background-color: #ffc9c9;
  border-left: 8px solid #df5c5c;
}

._green {
  background-color: #c9ffca !important;
  border-left: 8px solid #67df5c !important;
}

.message>span {
  height: min-content;
  align-self: center;
  margin-left: 10px;
  /* font-weight: bold; */
}

.ratings {
  margin: 10px 0;
  display: grid;
  grid-template-columns: repeat(5, minmax(50px, 1fr));
}

.container {
  max-width: 500px;
  margin: 30px auto;
  margin-top: 0;
  overflow: auto;
  min-height: 300px;
  border: 1px solid steelblue;
  padding: 20px;
  padding-top: 0;
  border-radius: 5px;
}
.btn {
  display: inline-block;
  background: #000;
  color: #fff;
  border: none;
  padding: 10px 20px;
  margin: 5px;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
  font-size: 15px;
  font-family: inherit;
}
.btn:focus {
  outline: none;
}
.btn:active {
  transform: scale(0.98);
}
.btn-block {
  display: block;
  width: 100%;
}
</style>
