<h1 align="center">Ramadan Challange</h1>

<p align="center">
Progressive Web App (PWA) Ramadan Challange App
</p>



## Features

An easy to use app to enhance yours and others Ramadan

- Google Based Auth
- Points System
- Scoreboard


## Build Commands

``` bash
# install dependencies
yarn # or `npm install`

# serve with hot reload at localhost:8080
yarn dev # or `npm run dev`

# build for production with minification and prerendering
yarn build # or `npm run build`

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
yarn run unit # or `npm run unit`

# run e2e tests
yarn run e2e # or `npm run e2e`

# run all tests
yarn test # or `npm test`
```

## Setup

### Firebase Config

Create a Firebase Application. Register the App as a Web App. Firebase will provide you with a script to add to your application. Instead go back to the app you just created and click on Config radio button instead. Firebase will provide a JSON Config Object.

Copy the object and replace `config` inside `initFirebase.js`.

### Firebase Hosting

Download the Firebase CLI by doing `npm install -g firebase-tools`.
Login with the CLI with `firebase login`
Initialiaze with `firebase init`
Deploy with `firebase deploy`

### Update Database in Firebase

The Ramadan Challange application useses the older Realtime Database instead of CloudFirestore. This repo provides an initial setup of `Deeds` or `Tasks` for your users. Go to Firebase Realtime Database and use the import function and import the file `art-of-good-export.json`

The bassic structure of a deed is the following...

## Deed Structure

### Deed Description
```javascript
{
  "Category": "Prayers, Habits, Deeds, Morning etc..",
  "Description": "Subtext of the Deed, funny line or something"
  "Limit": "How many times a person can check in a day"
  "Points": "(int) Points awarded"
  "Text": "Main Descriptive Text of the Deed"
}
```
### Example

```javascript
{
  "Category": "Prayers",
  "Description": "If you miss one that's sad"
  "Limit": "5"
  "Points": "20"
  "Text": "5 Daily Prayers"
}
```
