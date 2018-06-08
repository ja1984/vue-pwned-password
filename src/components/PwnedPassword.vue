<template>
  <div class="pwned-password" :class="{'password-compromised': isCompromised, 'password-uncompromised': (!isCompromised && isChecked), 'password-unchecked': !isChecked, 'password-checking': isChecking}">
    <input class="pwned-password-input" :type="doShowPassword ? 'text' : 'password'" :placeholder="placeholder" :value="value" @input="handleInput($event.target.value)" @blur="handleBlur">
    <i class="pwned-password-icon" v-if="showToggle" @click="passwordAsText = !passwordAsText" :class="{'show-password': doShowPassword}"></i>
  </div>
</template>

<script>
export default {
  name: 'PwnedPassword',
  props: {
    value: {
      type: String,
      required: true,
    },
    placeholder: {
      type: String,
      default: '',
    },
    trigger: {
      type: String,
      default: 'blur',
    },
    triggerTime: {
      type: Number,
      default: 500,
    },
    showToggle: {
      type: Boolean,
      default: true,
    },
    showPassword: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      passwordAsText: false,
      isCompromised: false,
      isChecked: false,
      isChecking: false,
      needsCheck: false,
      timer: null,
    };
  },
  methods: {
    handleInput(value) {
      clearTimeout(this.timer);
      this.isChecked = false;
      this.needsCheck = true;
      this.$emit('input', value);

      if (this.trigger !== 'change') return;
      this.timer = setTimeout(() => {
        this.check();
      }, this.triggerTime);
    },
    check() {
      if (!this.needsCheck) return;
      if (this.value.length === 0) {
        this.needsCheck = false;
        this.isCompromised = false;
        this.isChecked = true;
        return;
      }

      this.isChecking = true;
      this.sha1(this.value).then((hash) => {
        const range = hash.substring(0, 5);
        const check = hash.substring(5).toUpperCase();
        fetch.get(`https://api.pwnedpasswords.com/range/${range}`).then((response) => {
          this.isChecked = true;
          this.isChecking = false;
          this.needsCheck = false;
          const compromisedPasswords = response.text().data.split('\n');

          let appearances = 0;
          let compromised = false;

          for (let i = 0; i < compromisedPasswords.length; i += 1) {
            const row = compromisedPasswords[i];
            if (row.toUpperCase().indexOf(check) > -1) {
              const result = row.split(':');
              compromised = true;
              appearances = result[1];
              break;
            }
          }
          this.$emit('checkcomplete', { compromised, appearances });
          this.isCompromised = compromised;
        });
      });
    },
    handleBlur() {
      if (this.trigger !== 'blur') return;
      this.check();
    },
    sha1(value) {
      const buffer = new TextEncoder('utf-8').encode(value);
      return crypto.subtle.digest('SHA-1', buffer).then(hash => Array.from(new Uint8Array(hash)).map(x => x.toString(16).padStart(2, '0')).join('')); //eslint-disable-line
    },
  },
  computed: {
    doShowPassword() {
      if (this.showPassword) return true;
      return this.passwordAsText;
    },
  },
};
</script>

<style>
.pwned-password {
  position: relative;
  display: inline-block;
}

.pwned-password-icon {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  transition: all ease 0.3s;
  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAADOklEQVRYR+2VT2hcVRTGv3Pu1ccsWu1EqrVitd24EKSIIHUhlZZKEcSCRQQVVAhJ3rszwdCpInQIhhoaZvLeKwlR7EZd2KKIpfiHNnZT2uLCLsWNsRZLjSFtLIGZzDtHHqQwvs4koy5KYe7yvXPu+d3vnu9cwi1edIvrowvQVaCrwO2jQKVSyVlrd6jqbiJ6FMAGAPeJyJ0AFph5RlUvMPP04uLi8VKp9FcnM2ZVBeI4vh/A/iRJ+pn5tKp+A+AcEV02xlyZnZ2t5/P5uwBsUdXHATxLRLsBHEmSZGRwcHBmJZC2AOVymfP5/H4RGTbGVAFUgyD4vZNTjY+P32uMCVS1pKoj8/Pz75XL5Uar3JYAY2Nj93ie97mI1AC8WSgULjYnx3H8iKq+D+AZAAxgmoj2BUHwU3NcFEUPiMgHANZ6nrenr6/vjyzETQCVSmWjtfaUqh51zh0gIm1R/AKAIWvtp8YYqdVqr4jIIWPM1izEspIHALwMYLtz7lLzfv8AiKJoLYAzRHQkCIJU9ptWFEVfAjjpnDvc/DMMw4CIdjjnnm+T9xaA1z3P29bb23vtRkwW4DNVvVwoFIrt7jqKogVr7ab+/v75MAzPM7MGQfBktVq9m4h+KxaLa1bITaHXO+f2tgM4BuCSc25whU2ue563MT1FHMfnVDVxzj01MTGxrl6vX1wFYAJA3jn3UjuA9ArOAphyzkVtpPwKwHfZK4iiKFXtaefcC63y4jjep6qvAtjmnFtoCZB+TDsXwPdE9Inv+8OtmjBJkh/TrheRjz3Po0aj8ZqIjFhrt/q+/3MzQNqEPT09wyLyIjNvz1q5pQ0nJyfX12q1L4wx11S1N9u5yzYcFZHUhmDmk8xcyhYPw/BBIvpQVXNLS0t7hoaG/lzVhjcCpqam7qjX6++o6rsiknr+cLFYvNLJIKpWqxuY2aUqEVF5bm7u4L8aRBnfP6yqbwN4Q1W/BvAtEf0gIr8w89U01hizrtFobGbmJ5Ik2cXMO0XkI2vtQd/3f/1PozibNDo6uiaXyz23PP0eA/AQgPQNgIhcJaIZZk4H1DQznxgYGLjeiVqrPkadbPJ/YroAXQW6CnQV+BujgGYwn/HP2gAAAABJRU5ErkJggg==');

  height: 24px;
  width: 24px;
  background-size: contain;
}

.pwned-password-icon.show-password {
  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAADFElEQVRYR+1VTYgcZRB9VT07BxPXcYVEIo6YkxfxJiQiYT3kFoiBLJ7Ug2YOvV09gdVhDSG9QdhEhJnub2fDRvCiiAclXgQFMd4MePGoN90EM7Ium0Rx2J+ukg5Z2Ex6/tSwEPa79KGr6r3vvar6CNt8aJvxsUNgR4EHW4E4jqNsysIwvP3NO/dNgQyciM6Y2cz/QmBhYeGRtbW1I2Y2DuA5AE+r6sPMPGJmNwEsEtEJEbkyKHimSF8F6vX6fmZ+h4iOA/jBzL4CcIWIrnue9/vS0tLfo6OjpZGRkT1BEPw0DHhPAlEUFcfGxk4BmCSiDwHUgyD4rdfm7ASP43iamXcVi8WZSqWyPnAPNBqNvcx8ycz+AvBGGIaLW5Odc8+Y2TkALxHRu0EQvJd3c+fcPlW9SESPeZ73su/7rU4S91jQbDafTNP0spl9IiJniMhywL8DMNNutz+u1Wp/9pLdzMg5dxrAq57njfu+f3VrvbsI1Ov1EhF973nexSAI6nmSJUnyBYCvReRC9n9Qz5MkqWZNWiwWD1Qqlaxpb5+7CMRx/BmAa2EYVrt5nSTJrXa7/UTezZMkqYnI+R65DsBeEZnIJdBoND5n5kUROdmtiHPu7TzP5+fnH93Y2LgqIru75cZx3Mz6QUReySWwaQEzXxCRpEehe5aMc+6kmR0SkaNdrHsLwGsADorIrVwCdzwtM/NlVf1oZWXlbBRFurVgp+fZzdM0fd3MplX1xWq1+vPW+KwJ4zg+S0QTzDzeOcq5i6jZbD6epuklADcAvCki1/IaLvMcQNbh36hqrRP8zkR9YGYPra+vH5uamvqj7xhuBmSLqFQqnSaim2EYvj9ot2f52QVUVdI0PeF5Xry8vDwbRdHGwIuoM7AT3Dn37Orq6vVyubzSarUKqloqFAr7zex5AIdV9QUz+7RQKMxOTk7+2mt79n0LOsHn5uYOqOqcqj4FoMTMqareIKJfmPlHAN8y85e+72dbtO/pSWAY2fsidQnoSyDL6/We/1vgrmP4XwsOm9+3B4YtOGz8DoEdBbZdgX8Avje7MFg9IugAAAAASUVORK5CYII=');
}

.pwned-password-input {
  padding-right: 45px;
}
</style>
