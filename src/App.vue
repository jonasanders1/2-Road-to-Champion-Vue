<template>
  <div class="app">
    <div class="main-container">
      <div class="main-content">
        <h3 class="title">Drawn Conclusions</h3>
        <div class="image-container">
          <img src="./assets/pun.webp" class="image" />
        </div>
        <div class="input-container">
          <input v-model="prompt" placeholder="Enter a prompt" class="input">
          <button class="button">Submit</button>
        </div>
      </div>
    </div>
    <div class="log-container">
      <div class="log">
        <h3 class="log-title">Results log</h3>
        <div class="results">
          <div class="guess">
            <p class="user-guess">Ant with brain</p>
            <p class="guess-state wrong">Wrong</p>
          </div>
          <div class="guess">
            <p class="user-guess">Ant with brain</p>
            <p class="guess-state wrong">Wrong</p>
          </div>
          <div class="guess">
            <p class="user-guess">Ant with brain</p>
            <p class="guess-state correct">Correct</p>
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
      userGuesses: [],
      punPrompt: '',
      imageUrl: null,
    };
  },
  methods: {
    async getRandomPun() {
      try {
        const response = await axios.post('https://api.openai.com/v1/chat/completions', {
          model: "gpt-3.5-turbo",
          messages: [
            {
              "role": "system",
              "content": "You are an assistant who creates clever visual puns. These puns should be easily visualizable and consist of a short description followed by the pun answer. They are for a guessing game where players guess the pun from an image description."
            },
            { "role": "user", "content": "Create a visual pun for me." },
            {
              "role": "assistant",
              "content": "An image of a cat wearing a crown. Pun: A purr-fect king."
            },
            { "role": "user", "content": "I need another visual pun." }
          ],
          temperature: 0.7
        }, {
          headers: {
            'Authorization': `Bearer ${import.meta.env.VITE_API_KEY}`,
            'Content-Type': 'application/json'
          }
        });

        const pun = response.data.choices[0].message.content.trim();
        this.punPrompt = pun.split('. Pun: ')[1]; // Extracting just the pun
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
      }
    },
    async generateImageWithPun() {
      try {
        await this.getRandomPun()

        if (this.punPrompt) {
          const response = await axios.post('https://api.openai.com/v1/images/generations', {
            model: "dall-e-3",
            prompt: `Generate an image representing the pun: ${this.punPrompt}`,
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
      }
    }

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
  height: 85vh;
  gap: 20px;
}

.main-container {
  display: flex;
  flex: 2;
  justify-content: end;
}

.log-container {
  display: flex;
  flex: 1;
  justify-content: center;
  margin-top: 68px;
}

.log {
  background-color: rgb(95, 95, 95);
  border-radius: 8px;
  height: 300px;
  width: 80%;
  padding: 10px;
  color: white;
  display: flex;
  flex-direction: column;
}
.log-title {
  font-size: 1.5rem;
  font-weight: 600;
}
.results {
  flex: 1;
  padding: 10px;
  border-top: 2px solid rgb(152, 152, 152);
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
.main-content {
  display: flex;
  flex-direction: column;
  align-items: end;
  justify-content: space-between;
  width: 500px;
}

.title {
  font-size: 35px;
  font-weight: 600;
  text-align: right;
  color: rgb(95, 95, 95);
}

.image-container {
  box-shadow: inset 0px 0px 5px rgba(95, 95, 95);
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 500px;
  height: 500px;
}

.image {
  aspect-ratio: 1;
  object-fit: cover;
  border-radius: 8px;
  box-shadow: 3px 6px 8px rgba(0, 0, 0, 0.2);
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
}</style>
