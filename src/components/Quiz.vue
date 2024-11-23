<template>
    <div class="quiz-container">
      <h1 class="quiz-title">{{ strings.quizTitle }}</h1>
      <div v-if="!isJoined" class="join-container">
        <input v-model="userName" :placeholder="strings.enterNamePlaceholder" class="input-field" />
        <button @click="joinQuiz" class="join-button">{{ strings.joinButton }}</button>
      </div>
      <div v-if="isAdmin" class="admin-controls">
        <button @click="previousQuestion" class="control-button">{{ strings.previousQuestionButton }}</button>
        <button @click="nextQuestion" class="control-button">{{ strings.nextQuestionButton }}</button>
        <button @click="toggleStatistics" class="control-button">{{ strings.showStatisticsButton }}</button>
        <button @click="clearStatistics" class="control-button">{{ strings.clearStatisticsButton }}</button>
      </div>
      <div v-if="question" class="question-container">
        <h2 class="question-text">{{ question.text }}</h2>
        <ul class="options-list">
          <li v-for="(option, index) in question.options" :key="index" class="option-item">
            <input type="radio" :value="index" v-model="selectedAnswer" @change="submitAnswer" class="radio-input" />
            <label class="option-label">{{ option }}</label>
          </li>
        </ul>
      </div>
      <div v-if="showStatistics" class="statistics-container">
        <h2 class="statistics-title">{{ strings.statisticsTitle }}</h2>
        <ul class="statistics-list">
          <li v-for="(count, user) in sortedStatistics" :key="user" class="statistics-item">
            {{ user }}: {{ count }} {{ strings.correctAnswers }}
          </li>
        </ul>
      </div>
    </div>
  </template>
  
  <script>
  import * as signalR from '@microsoft/signalr';
  import strings from '@/strings';
  
  export default {
    data() {
      return {
        userName: localStorage.getItem('userName') || '',
        isAdmin: false,
        isJoined: false,
        question: null,
        selectedAnswer: null,
        statistics: null,
        showStatistics: false,
        connection: null,
        strings
      };
    },
    watch: {
      userName(newName) {
        localStorage.setItem('userName', newName);
      }
    },
    computed: {
      sortedStatistics() {
        if (!this.statistics) return {};
        return Object.fromEntries(
          Object.entries(this.statistics).sort(([,a],[,b]) => b - a)
        );
      }
    },
    methods: {
      joinQuiz() {
        this.isAdmin = this.userName === 'Денис Зайцев';
        this.isJoined = true;
        this.connection = new signalR.HubConnectionBuilder()
          .withUrl('http://localhost:5122/quizHub')
          .build();
  
        this.connection.on('ReceiveQuestion', (question) => {
          this.question = question;
          this.selectedAnswer = null;
        });
  
        this.connection.on('ReceiveStatistics', (statistics) => {
          this.statistics = statistics;
        });
  
        this.connection.start().then(() => {
          if (this.isAdmin) {
            this.connection.invoke('SendQuestion');
          } else {
            this.connection.invoke('GetCurrentQuestion');
          }
        }).catch(err => console.error(err.toString()));
      },
      submitAnswer() {
        this.connection.invoke('SubmitAnswer', this.userName, this.selectedAnswer);
      },
      nextQuestion() {
        this.connection.invoke('NextQuestion');
      },
      previousQuestion() {
        this.connection.invoke('PreviousQuestion');
      },
      toggleStatistics() {
        this.showStatistics = !this.showStatistics;
        if (this.showStatistics) {
          this.connection.invoke('GetStatistics');
        }
      },
      clearStatistics() {
        this.connection.invoke('ClearStatistics');
      }
    }
  };
  </script>
  
  <style scoped>
  body {
    font-family: 'Arial', sans-serif;
    background-color: #f5f5f5;
    margin: 0;
    padding: 0;
  }
  
  .quiz-container {
    text-align: center;
    padding: 20px;
    background-color: #ffffff;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    max-width: 600px;
    margin: 50px auto;
    animation: fadeIn 1s ease-in-out;
  }
  
  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(-20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  .quiz-title {
    color: #333333;
    font-size: 2.5em;
    margin-bottom: 20px;
    animation: slideIn 0.5s ease-in-out;
  }
  
  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateX(-20px);
    }
    to {
      opacity: 1;
      transform: translateX(0);
    }
  }
  
  .join-container {
    margin-bottom: 20px;
    animation: fadeIn 0.5s ease-in-out;
  }
  
  .input-field {
    padding: 10px;
    margin-right: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    font-size: 16px;
    width: 200px;
    transition: border-color 0.3s;
  }
  
  .input-field:focus {
    border-color: #e74c3c;
    outline: none;
  }
  
  .join-button, .control-button {
    padding: 10px 20px;
    background-color: #e74c3c;
    color: #ffffff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    margin: 5px;
    transition: background-color 0.3s, transform 0.3s;
  }
  
  .join-button:hover, .control-button:hover {
    background-color: #c0392b;
    transform: scale(1.05);
  }
  
  .admin-controls {
    margin-bottom: 20px;
    animation: fadeIn 0.5s ease-in-out;
  }
  
  .question-container {
    margin-bottom: 20px;
    animation: fadeIn 0.5s ease-in-out;
  }
  
  .question-text {
    font-size: 20px;
    color: #333333;
    margin-bottom: 10px;
    animation: slideIn 0.5s ease-in-out;
  }
  
  .options-list {
    list-style-type: none;
    padding: 0;
  }
  
  .option-item {
    margin-bottom: 10px;
    animation: fadeIn 0.5s ease-in-out;
  }
  
  .radio-input {
    margin-right: 10px;
  }
  
  .option-label {
    font-size: 18px;
    color: #333333;
  }
  
  .statistics-container {
    margin-top: 20px;
    animation: fadeIn 0.5s ease-in-out;
  }
  
  .statistics-title {
    font-size: 20px;
    color: #333333;
    margin-bottom: 10px;
    animation: slideIn 0.5s ease-in-out;
  }
  
  .statistics-list {
    list-style-type: none;
    padding: 0;
  }
  
  .statistics-item {
    font-size: 16px;
    color: #333333;
    margin-bottom: 5px;
    animation: fadeIn 0.5s ease-in-out;
  }
  </style>