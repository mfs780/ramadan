<template>
  <mu-appbar v-show="isLogin" id="my-header">


    <router-link to="/"
                 class="logo">
      The Art of Good <span class="diamond">&#9670;</span> فن الصالحه
    </router-link>

    <mu-raised-button v-show="!user"
                      label="Login"
                      secondary
                      slot="right"
                      to="/login" />

    <mu-flat-button v-show="user"
                    slot="right"
                    ref="userButton"
                    @click="toggle">
      <mu-avatar :src="user && user.photoURL" />
    </mu-flat-button>

    <mu-popover slot="right"
                :trigger="userMenuTrigger"
                :open="userMenuOpen"
                @close="handleClose"
                :anchorOrigin="{vertical: 'bottom', horizontal: 'right'}"
                :targetOrigin="{vertical: 'top', horizontal: 'right'}">
      <mu-menu>
        <mu-menu-item title="Profile"
                      to="/profile"
                      @click.native="handleClose" />

        <mu-menu-item title="Leaderboard"
                      to="/leaderboard"
                      @click.native="handleClose" />

        <mu-menu-item title="Logout"
                      to="/logout"
                      @click.native="handleClose" />
      </mu-menu>
    </mu-popover>

  </mu-appbar>
</template>


<script>
import { mapState } from "vuex";

export default {
  data: () => ({
    userMenuOpen: false,
    userMenuTrigger: null
  }),
  computed: {
    ...mapState({
      user: "user"
    }),
    isLogin () {
      return this.$route.path !== '/login';
    }
  },
  watch: {
    user(val) {
      if (val) {
        this.userMenuTrigger = this.$refs.userButton.$el;
      }
    }
  },
  methods: {
    toggle() {
      this.userMenuOpen = !this.userMenuOpen;
    },
    handleClose(e) {
      this.userMenuOpen = false;
    }
  }
};
</script>


<style lang="sass">
#my-header
  background-color: #0067b8;
  .logo
    display: flex;
    justfiy-content: flex-start
    align-items: center
    font-family: 'Reem Kufi', sans-serif
    color: white
    text-decoration: none
    padding: 10px

  .diamond
    font-size: 8px
    margin: 0 10px
</style>
