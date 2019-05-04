<template>
  <div class="flex-column">
    <div class="profile flex-row flex-wrap">
      <img class="user"
           :src="user.photoURL" />
      <div class="flex-row flex-wrap"
           :class="[rs('xs') ? 'center-start' : 'start-start']">
        <div class="item flex-row start-start">
          <img class="coin"
               src="../assets/icons/medal.png" />
          <div class="data">
            <p class="available">{{ totalAvailablePoints }}</p>
            <p class="available-text">Available Points</p>
            <p class="lifetime-text">
              <span class="lifetime">{{ totalAvailablePoints }}</span>Lifetime Points</p>
          </div>
        </div>
        <div class="item flex-row start-start">
          <img class="coin"
               src="../assets/icons/streak.png" />
          <div class="data">
            <p class="available">0 Days</p>
            <p class="available-text">Streak count</p>
            <p class="lifetime-text">
              <span class="lifetime">0</span>(Not Implemented)</p>
          </div>
        </div>
        <div class="item flex-row start-start">
          <img class="coin"
               src="../assets/icons/coins.png" />
          <div class="data">
            <p class="available">{{ totalPointsEarnedToday }} / {{totalPointsToday}}</p>
            <p class="available-text">Points Earned Today</p>
          </div>
        </div>
      </div>
    </div>
    
    <p class="flex-row center-center todays-date">Deeds for {{today.split('_').join(' / ')}}</p>

    <div class="flex-row center-center fix-yesterday"
         @click="toggleYesterday">
      <p v-if="yesterday == 0">Update Yesterdays Deeds</p>
      <p v-else>Go Back to Days Deeds</p>
    </div> 

    <div class="habits-containers flex-row center-start"
        :class="{ 'flex-column start-center big': isSmall, 'large': isLarge }">
      <div v-for="(deedType, typeIndex) in allDeeds"
          class="habits-container"
          :key="typeIndex">
        <p class="habit-title">{{ deedType.title }}</p>
        <div v-for="(deed, index) in deedType.deeds"
            :key="index"
            class="habit-container flex-column start-start">
          <div class="point-container flex-row end-center self-end">
            <div v-show="showMinus(deed)"
                class="button flex-row center-center"
                @click="removeDeed(deed)" >
              <img class="minus-image"
                   src="../assets/icons/minus.svg"/>
            </div>
            <div v-show="!hasMaxDeeds(deed)"
                class="button"
                @click="addDeed(deed)" >
              <img class="coin point-image"
                   src="../assets/icons/plus.svg"/>
            </div>
            <div v-show="hasMaxDeeds(deed)"
                class="button">
              <img class="coin point-image"
                  src="../assets/icons/check.svg"/>
            </div>
            <span class="point-text">{{deed.Points}}</span>
          </div>
          <p class="habit-text">{{deed.Text}}</p>
          <div class="habit-subtext flex-row">
            <p class="habit-description">{{deed.Description}}</p>
            <p class="habit-points">{{getPoints(deed)}}/{{getPointsTotal(deed)}}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";

export default {
  data () {
    return {
      deeds: [],
      myDeeds: [],
      yesterday: 0
    };
  },

  computed: {
    ...mapState(["user"]),
    allDeeds () {
      return [
        { deeds: this.prayerDeeds, title: 'worship' },
        { deeds: this.habitDeeds, title: 'habits' },
        { deeds: this.charityDeeds, title: 'charity' }
      ];
    },
    prayerDeeds () {
      return this.deeds.filter((deed, index) => {
        return deed.Category === 'Prayers'
      })
    },
    habitDeeds () {
      return this.deeds.filter((deed) => {
        return deed.Category === 'Habits'
      })
    },
    charityDeeds () {
      return this.deeds.filter((deed) => {
        return deed.Category === 'Charity'
      })
    },
    isSmall () {
      return this.$mq == 'sm' || this.$mq == 'xs';
    },
    isLarge () {
      return this.$mq == 'lg';
    },
    totalPointsToday () {
      return this.deeds.reduce((acc, deed) => {
        return acc + (deed.Points * (deed.Limit || 1));
      }, 0)
    },
    totalAvailablePoints () {
      let total = 0;

      if (this.myDeeds) {
        Object.keys(this.myDeeds).map((dates) => {
          Object.keys(this.myDeeds[dates]).map((key) => {
            total += this.deeds && this.deeds[key] ? this.deeds[key].Points * (this.myDeeds[dates][key].amount || 1) : 0;
          });
        });
      }

      return total;
    },
    totalPointsEarnedToday () {
      let total = 0;
      if (this.myDeeds && this.myDeeds[this.today]) {
        Object.keys(this.myDeeds[this.today]).map((key) => {
          total += this.deeds && this.deeds[key] ? this.deeds[key].Points * (this.myDeeds[this.today][key].amount || 1) : 0;
        });
      }

      return total;
    },
    today () {
      let ms = Date.now();
      let d = new Date(ms);
      d.setDate(d.getDate()-this.yesterday);

      return `${d.getMonth() + 1}_${d.getDate()}_${d.getFullYear()}`;
    }
  },

  created () {
    let db = firebase.database();

    db.ref('Deeds/').once('value').then((snapshot) => {
      let deeds = snapshot.val();
      this.deeds = Object.keys(deeds).map((key) => {
        let deed = deeds[key];
        deed.key = key;
        return deed;
      });
    })

    db.ref(`users/${this.user.uid}/deeds/`)
    .once('value')
    .then((snapshot) => {
      this.myDeeds = snapshot.val();
    })

    db.ref(`users/${this.user.uid}/deeds/`).on("value", (snapshot) => {
      this.myDeeds = snapshot.val();
    }, function (errorObject) {
      console.log("The read failed: " + errorObject.code);
    });
  },

  mounted() {
  },

  methods: {
    addDeed (deed) {
      firebase.database().ref('/.info/serverTimeOffset')
        .once('value')
        .then((data) => {
          let key = this.today;

          var setDeed = firebase.database().ref(`users/${this.user.uid}/deeds/${key}/${deed.key}`);

          if (!this.hasDeed(deed)) {
            setDeed.set({
              'id': deed.key,
              'text': deed.Text,
              'date': firebase.database.ServerValue.TIMESTAMP,
              'amount': 1
            });
          } else if(!this.hasMaxDeeds(deed)) {
            setDeed.update({
              'amount': this.getDeedAmount(deed) + 1
            });
          }
        }, function (err) {
          return err;
        });
    },
    removeDeed (deed) {
      firebase.database().ref('/.info/serverTimeOffset')
        .once('value')
        .then((data) => {
          let key = this.today;

          var setDeed = firebase.database().ref(`users/${this.user.uid}/deeds/${key}/${deed.key}`);

          if (this.hasDeed(deed)) {
            if (this.hasOneDeed(deed)) {
              setDeed.remove();
            } else {
              setDeed.update({
              'amount': this.getDeedAmount(deed) - 1
            });
            }
          }
        }, function (err) {
          return err;
        });
    },
    hasDeed (deed) {
      return this.myDeeds && this.myDeeds[this.today] && this.myDeeds[this.today][deed.key];
    },
    hasAtLeastOneDeed (deed) {
      return this.getDeed(deed).amount > 0
    },
    hasOneDeed (deed) {
      return 1 === this.getDeed(deed).amount;
    },
    hasMaxDeeds (deed) {
      return this.hasAtLeastOneDeed(deed) && (deed.Limit <= this.getDeedAmount(deed) || deed.Limit === undefined);
    },
    getDeedAmount (deed) {
      return this.getDeed(deed).amount;
    },
    getDeed (deed) {
      return this.hasDeed(deed) ? this.myDeeds[this.today][deed.key] : {};
    },
    getPoints (deed) {
      return this.getDeed(deed).amount ? this.getDeed(deed).amount * deed.Points : '0';
    },
    getPointsTotal (deed) {
      return deed.Limit ? deed.Limit * deed.Points : deed.Points;
    },
    showMinus (deed) {
      return this.hasAtLeastOneDeed(deed);
    },
    toggleYesterday () {
      this.yesterday = (this.yesterday + 1) % 2
    }
  }
};
</script>

<style lang="scss">
.profile {
  justify-content: space-around;
  width: calc(100% - 40px);
  max-width: 900px;
  min-width: 320px;
  margin: 28px auto;
  padding: 36px;
  background-color: #fff;
}

.todays-date {
  font-size: 32px;
}

.user {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  margin-bottom: 10px;
}

.item {
  min-width: 160px;
  margin: 10px 0;
}

.button {
  width: 32px;
  height: 32px;
}

.minus-image {
  width: 100%;
  max-width: 20px;
  height: 7px;
  border-radius: 50%;
  margin-right: 10px;
}

.coin {
  align-self: flex-start;
  width: 100%;
  border-radius: 50%;
  margin-right: 10px;
}

.data {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-self: flex-start;
}

.data p {
  margin: 0;
}

.available {
  font-size: 16px;
  font-weight: 900;
}

.available-text {
  font-size: 12px;
}

.lifetime-text {
  font-size: 12px;
  color: gray;
}

.lifetime {
  font-size: 14px;
  margin-right: 5px;
}

.hatbits-containers{
  user-select: none;
}

.habit-container {
  min-width: 220px;
  max-width: 350px;
  min-height: 100px;
  background-color: #FFF;
  padding: 10px 20px 10px 20px;
  margin: 10px;
  cursor: pointer;
}

.big .habit-container {
  min-width: 300px;
}

.large .habit-container {
  min-width: 400px;
}

.habit-title {
  font-size: 30px;
  text-align: center;
  text-transform: uppercase;
}

.habit-text {
  font-size: 18px;
  font-weight: 900;
  margin: 0;
}

.habit-earn {
  font-size: 16px;
  color: #0067b8;
}

.habit-subtext {
  width: 100%;
}

.habit-description {
  margin: 0;
  width: 100%;
}

.habit-points {
  margin: 0;
  text-align: right;
}

.habit-earn-chevron {
  font-size: 30px;
  font-weight: 100;
}

.point-text {
  font-weight: 900;
}

.fix-yesterday {
  border: 1px solid black;
  width: 200px;
  margin: 0 auto;
  cursor: pointer;
  font-weight: 900;
}
</style>
