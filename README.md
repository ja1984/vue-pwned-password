# vue-pwned-password
[![npm version](https://badge.fury.io/js/%40codejunkies%2Fvue-pwned-password.svg)](https://badge.fury.io/js/%40codejunkies%2Fvue-pwned-password)

## Installation
```
npm install --save @codejunkies/vue-pwned-password
-or-
yarn add @codejunkies/vue-pwned-password
```

## Demo
You can try a small demo of the component on [CodeSandBox](https://codesandbox.io/s/8l909v1v9)


## Usage
```html
<template>
  <div id="app">
    <pwned-password v-model="password" @checkcomplete="checkComplete"></pwned-password>
  </div>
</template>

<script>
import PwnedPassword from '@codejunkies/vue-pwned-password';

export default {
  name: 'app',
  data() {
    return {
      password: '',
    };
  },
  components: {
    PwnedPassword,
  },
  methods: {
    checkComplete(result) {
      if(result.compromised) {
        alert(`You should consider a diffrent password, this password has appeared ${result.appearances} times in password leaks.`) ;
      }
    }
  }
};
</script>
```

## Events

The pwned-password component emits 2 events, one for every update to the field and one after the check is completed.
The "checkcomplete" event returns two properits, one that indicates if the password is compromised or not, and a number that shows how many times the password has appeared in password leaks.

```html
<pwned-password v-model="password" @checkcomplete="checkComplete" @input="handleInput"></pwned-password>

<script>
  methods: {
    checkComplete(result) {
      if(result.compromised) {
        alert(`You should consider a diffrent password, this password has appeared ${result.appearances} times in password leaks.`) ;
      }
    },
    handleInput(password) {
      console.log(`Input entered: ${password}`)
    }
  }
```


## Props

| Name        | Type | Options | Default | Description  |
| ------------- | ------------- | --- | --- | ----- |
| trigger      | String | blur, change | blur | Choose when to trigger a check. |
| triggerTime | Number | - | 500 | Only if trigger option is *change*, how long to wait after last change to check password |
| showToggle | Boolean | true/false | true |      Show icon to toggle between password and text |
| showPassword | Boolean | true/false | false |      If you want to use your own toggle button (overrides "internal" toggle button) |


## Classes
The component will add diffrent classes to itself based on the status of the entered password.

| Name         | Description  |
| ------------ | ----- |
| password-compromised   | The password has been checked and are compromised |
| password-uncompromised | The password has been checked but are not compromised |
| password-unchecked     | The password has not yet been checked |
| password-checking      | The password is being checked agaisnt havibeenpwned database |

```css
.pwned-password.password-compromised .pwned-password-input {
  border-color: #900;
}

.pwned-password.password-uncompromised .pwned-password-input {
  border-color: #090;
}

.pwned-password.password-checking .pwned-password-input {
  border-color: #009;
}
```

Bump version
`npm version patch -a -m "message"` && `git push --tags` && `npm run release`
