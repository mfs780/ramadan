<template>
  <div class="flex-column">
    <div class="habits-containers flex-column center-start"
        :class="{ 'flex-column start-center big': isSmall, 'large': isLarge }">
        <div v-for="(user, index) in leaders"
            :key="index"
            class="profile flex-row flex-wrap">
            <img v-if="user.image"
                class="user"
                :src="user.image" />
            <p class="profile-name">{{ user.name }}</p>
            <div class="flex-row flex-wrap"
                :class="[rs('xs') ? 'center-start' : 'start-start']">
                <div class="item flex-row start-start">
                    <img class="coin"
                        src="../assets/icons/medal.png" />
                    <div class="data">
                        <p class="available">{{ user.points }}</p>
                        <p class="available-text">Total Points</p>
                    </div>
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
      users: {}
    };
  },

  computed: {
    ...mapState(["user"]),
    isSmall () {
      return this.$mq == 'sm' || this.$mq == 'xs';
    },
    isLarge () {
      return this.$mq == 'lg';
    },
    today () {
      let ms = Date.now();
      let d = new Date(ms);

      return `${d.getMonth() + 1}_${d.getDate()}_${d.getFullYear()}`;
    },
    leaders () {
        if (!Object.keys(this.users).length) {
            return []
        }

        return Object.keys(this.users).map((uid) => {
            if (!this.users[uid].profile) {
              return {
                name: 'Re Login',
                points: this.totalPoints(this.users[uid].deeds)
              }
            }
            return {
                name: this.users[uid].profile.name,
                image: this.users[uid].profile.image,
                points: this.totalPoints(this.users[uid].deeds)
            }
        }).sort((a, b) => {
          return b.points - a.points;
        })
    }
  },

  created () {
    var setUser = firebase.database().ref(`users/${this.user.uid}/profile/`);
    setUser.set({
      'name': this.user.displayName,
      'image': this.user.photoURL
    });

    let db = firebase.database();

    db.ref('Deeds/').once('value').then((snapshot) => {
      let deeds = snapshot.val();
      this.deeds = Object.keys(deeds).map((key) => {
        let deed = deeds[key];
        deed.key = key;
        return deed;
      });
    });

    db.ref('users/').once('value').then((snapshot) => {
      this.users = snapshot.val();
    })
  },

  mounted() {
  },

  methods: {
      totalPoints (myDeeds) {
      let total = 0;

      if (myDeeds) {
        Object.keys(myDeeds).map((dates) => {
            Object.keys(myDeeds[dates]).map((key) => {
                total += this.deeds && this.deeds[key] ? this.deeds[key].Points * (myDeeds[dates][key].amount || 1) : 0;
            });
        });
      }

      return total;
    },
  }
};
</script>

<style lang="scss">
.habits-containers {
  margin-top: 20px;
}

.profile {
  justify-content: space-around;
  width: calc(100% - 40px);
  max-width: 900px;
  min-width: 320px;
  margin: 10px auto;
  padding: 16px;
  background-color: #fff;
}

.profile-name {
    font-size: 22px;
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

.coin {
  align-self: flex-start;
  width: 32px;
  height: 32px;
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

.habit-description {
  margin: 0
}

.habit-earn-chevron {
  font-size: 30px;
  font-weight: 100;
}

.point-text {
  font-weight: 900;
}
</style>
