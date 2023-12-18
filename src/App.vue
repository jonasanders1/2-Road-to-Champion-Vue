<template>
  <div class="app">
    <div class="main-container">
      <div class="main-content">
        <h3 class="title">Drawn Conclusions</h3>
        <div class="image-container">
          <div v-if="isLoading" style="display: flex; justify-content: center; flex-direction: column; align-items: center;">
            <div class="spinner"></div>
            <p>{{ loadingStateMessage }}</p>
          </div>
          <img v-if="imageUrl" :src="imageUrl" class="image" />
        </div>
        <div class="input-container">
          <input v-model="userGuess" placeholder="Enter a prompt" class="input" v-on:keydown.enter="handleUserSubmit">
          <button class="button" @click="handleUserSubmit">Submit</button>
        </div>
      </div>
    </div>
    <div class="log-container">
      <div class="log" v-if="userGuesses.length > 0">
        <h3 class="log-title">Results log:</h3>
        <div class="results">
          <div class="guess" v-for="(entry, index) in userGuesses" :key="index">
            <p class="user-guess">{{ entry.guess }}</p>
            <p
              :class="{ 'guess-state': true, 'correct': entry.state === 'correct', 'wrong': entry.state === 'wrong', 'almost': entry.state === 'almost' }">
              {{ entry.state }}
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import axios from 'axios';

export default {
  data() {
    return {
      userGuess: '',
      userGuesses: [],
      punPrompt: 'Spiderman eating cake',
      imageUrl: null,
      isLoading: false,
      loadingStateMessage: ''
    };
  },
  methods: {
    async getRandomPun() {
      this.isLoading = true
      this.loadingStateMessage = "Getting a famous saying"

      try {
        const response = await axios.post('https://api.openai.com/v1/chat/completions', {
          model: "gpt-3.5-turbo",
          messages: [
            {
              "role": "system",
              "content": "You are an assistant skilled at creating short, well-known sayings that are limited to three words. These sayings should be easily visualizable for image creation. The sayings should not include any punctuation like periods, exclamation marks, or commas. Respond with only the saying itself, with no additional text, explanation, or punctuation."
            },
            { "role": "user", "content": "Give me a three-word visual saying without punctuation." },
            { "role": "user", "content": "Please provide another three-word saying, no punctuation." },
            { "role": "user", "content": "I need one more, remember no punctuation." }
          ],
          temperature: 0.7
        }, {
          headers: {
            'Authorization': `Bearer ${import.meta.env.VITE_API_KEY}`,
            'Content-Type': 'application/json'
          }
        });

        const pun = response.data.choices[0].message.content.trim();
        this.punPrompt = pun
      } catch (error) {
        if (error.response) {
          console.error(error.response.data);
          console.error(error.response.status);
          console.error(error.response.headers);
        } else if (error.request) {
          console.error(error.request);
        } else {
          console.error('Error', error.message);
        }
      } finally {
        this.isLoading = false
        this.loadingStateMessage = ""
      }
    },
    async generateImageWithPun() {
      await this.getRandomPun()

      if (this.punPrompt) {
        this.isLoading = true
        this.loadingStateMessage = "Generating image based on the saying..."
      }
      try {
        if (this.punPrompt) {
          const response = await axios.post('https://api.openai.com/v1/images/generations', {
            model: "dall-e-3",
            prompt: `Create a visual representation of the saying '${this.punPrompt}' that captures its essence without using any textual elements. The image should symbolically or metaphorically represent the saying in a clear and recognizable way.`,
            n: 1,
            size: "1024x1024"
          }, {
            headers: {
              'Authorization': `Bearer ${import.meta.env.VITE_API_KEY}`,
              'Content-Type': 'application/json'
            }
          });

          this.imageUrl = response.data.data[0].url;
          console.log("Answer: ", this.punPrompt)
        }
      } catch (error) {
        if (error.response) {
          console.error(error.response.data);
          console.error(error.response.status);
          console.error(error.response.headers);
        } else if (error.request) {
          console.error(error.request);
        } else {
          console.error('Error', error.message);
        }
      } finally {
        this.isLoading = false
        this.loadingStateMessage = ""
      }
    },
    handleUserSubmit() {
      if (this.userGuess !== "") {
        const punWords = this.punPrompt.split(/\s+/)
        const guess = this.userGuess.toLowerCase()
        let result = { guess: this.userGuess, state: "wrong" }

        if (punWords.length > 1) {
          if (punWords.some(word => guess.includes(word.toLowerCase()))) {
            result.state = "almost"
          }
        } else {
          const matchCount = [...this.punPrompt.toLowerCase()].reduce((count, char) => {
            return count + (guess.includes(char) ? 1 : 0)
          }, 0)

          if (matchCount >= 2) {
            result.state = "almost"
          }
        }
        if (guess === this.punPrompt.toLowerCase()) {
          result.state = "correct"
        }

        this.userGuesses.push(result);
        this.userGuess = '';
      }
    },
  },
  mounted() {
    this.generateImageWithPun()
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300;400;500;600&display=swap');

* {
  font-family: "Montserrat";
  font-weight: 400;
}

.app {
  padding: 26px;
  display: flex;
  width: 100%;
  height: 100vh;
  gap: 20px;
  background-color: rgb(41, 41, 41);
}

.main-container {
  display: flex;
  flex: 2;
  justify-content: end;
  height: 620px;
}

.log-container {
  display: flex;
  flex: 1;
  justify-content: center;
  margin-top: 68px;
}

.log {
  box-shadow: 3px 6px 8px rgba(0, 0, 0, 0.2);
  background-color: rgba(95, 95, 95, 0.5);
  /* Semi-transparent background */
  backdrop-filter: blur(10px);
  /* Apply blur effect */
  border-radius: 8px;
  height: fit-content;
  max-height: 300px;
  width: 80%;
  padding: 10px;
  color: white;
  display: flex;
  flex-direction: column;
  overflow: scroll;
  position: relative;
}

.log-title {
  font-size: 1.5rem;
  font-weight: 600;
  border-bottom: 2px solid rgb(152, 152, 152);
}

.results {
  flex: 1;
  padding: 10px;
}

.guess {
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
  padding-bottom: 5px;
  border-bottom: 1px solid rgb(74, 74, 74);
  font-weight: 500;
}

.user-guess {
  color: white;
  text-transform: capitalize;
}

.guess-state {
  font-weight: 600;
}

.wrong {
  color: rgb(255, 110, 110);
}

.correct {
  color: rgb(141, 255, 110);
}

.almost {
  color: rgb(250, 245, 110);
}

.main-content {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  width: 500px;
}

.title {
  font-size: 35px;
  font-weight: 600;
  color: rgba(81, 202, 243);
}

.image-container {
  box-shadow: inset 0px 0px 5px rgba(95, 95, 95);
  background-color: rgba(95, 95, 95);
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 500px;
  height: 500px;
  /* box-shadow: 3px 6px 8px rgba(0, 0, 0, 0.2); */
}

.image {
  aspect-ratio: 1;
  object-fit: cover;
  border-radius: 8px;
}

.input-container {
  width: 100%;
  display: flex;
  gap: 10px;
}

.input,
.button {
  border-radius: 8px;
  padding: 10px;
}

.input {
  flex: 3;
  border: none;
  box-shadow: inset 0px 0px 5px rgba(95, 95, 95);
  border-radius: 8px;
  padding: 10px;
  outline: none;
  transition: all 100ms ease-in-out;
}

.button {
  flex: 1;
  background-color: rgb(23, 23, 23);
  color: white;
  font-weight: 600;
  border: none;
  outline: none;
  opacity: .6;
  transition: all 100ms ease-in-out;
}

.button:focus,
.button:hover {
  background-color: rgb(81, 203, 243);
  opacity: 1;
}

.input:focus,
.input:hover {
  box-shadow: inset 0px 0px 5px rgba(81, 203, 243);
}

.spinner {
  border: 4px solid rgba(95, 95, 95, 0.7);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border-left-color: rgba(81, 203, 243);
  animation: spin 1s ease infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>
