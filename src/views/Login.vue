<template>
  <div class="login-container">
    <div class="auth-container">
      <p class="title">
        The Art of Good
        <span class="diamond">&#9670;</span>
        فن الصالحه
      </p>
      <div id="firebaseui-auth-container" />
    </div>
  </div>
</template>
<script>
import { mapState } from "vuex";
import firebase from "firebase";
import firebaseui from "firebaseui";
import { ui } from "../initFirebase";
// import router from '../router'

const uiConfig = {
  callbacks: {
    // Called when the user has been successfully signed in
    signInSuccess(user, credential, redirectUrl) {
      // router.push('/')
      // Do not redirect.

      var setUser = firebase.database().ref(`users/${user.uid}/profile/`);
      setUser.set({
        'name': user.displayName,
        'image': user.photoURL
      });

      return false;
    }
  },
  credentialHelper: firebaseui.auth.CredentialHelper.NONE,
  // Opens IDP Providers sign-in flow in a popup
  signInFlow: "popup",
  signInOptions: [
    {
      provider: firebase.auth.GoogleAuthProvider.PROVIDER_ID,
      scopes: ["https://www.googleapis.com/auth/plus.login"]
    }
  ]
  // Terms of service url.
  // tosUrl: 'https://www.google.com'
};

export default {
  computed: {
    ...mapState(["user"])
  },
  watch: {
    user(val) {
      if (val) {
        if (this.$router.currentRoute.query.redirect) {
          this.$router.replace(this.$router.currentRoute.query.redirect);
        } else {
          this.$router.replace("/");
        }
      }
    }
  },
  mounted() {
    ui.start("#firebaseui-auth-container", uiConfig);
  }
};
</script>
<style lang="sass">
.login-container
  width: 100%
  height: 100%
  display: flex
  justify-content: center;
  align-items: center
  text-align: center
  background: url('../assets/backgrounds/1.jpg')
  background-size: cover;

.auth-container
    width: calc(100% - 40px);
    padding: 36px;
    background-color: #FFF;
    margin-left: auto;
    margin-right: auto;
    min-width: 320px;
    border: 1px solid rgba(0,0,0,.4);
    box-shadow: 0 2px 3px rgba(0,0,0,.55);
    margin-bottom: 28px;
    max-width: 412px;

.title
  font-size: 20px

.diamond
  font-size: 8px
  margin: 0 10px
</style>
