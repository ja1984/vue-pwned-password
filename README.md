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
