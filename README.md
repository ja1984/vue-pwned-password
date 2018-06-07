# vue-pwned-password
[![npm version](https://badge.fury.io/js/%40codejunkies%2Fvue-pwned-password.svg)](https://badge.fury.io/js/%40codejunkies%2Fvue-pwned-password)

## Installation
```
npm install --save vue-pwned-password
-or-
yarn add vue-pwned-password
```

## Usage
```html
<template>
  <div id="app">
    <pwned-password v-model="password"></pwned-password>
  </div>
</template>

<script>
import PwnedPassword from 'vue-pwned-password';

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
};
</script>
```
