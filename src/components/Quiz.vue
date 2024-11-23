<template>
    <div class="quiz-container">
      <div class="logo-block">
        <img src="/logo.png" alt="Quiz Logo" class="quiz-logo" />
      </div>
      <h1 v-if="!isJoined" class="quiz-title">{{ strings.quizTitle }}</h1>
      <div v-if="!isJoined" class="join-container">
        <input v-model="userName" :placeholder="strings.enterNamePlaceholder" class="input-field" />
        <button @click="joinQuiz" class="join-button">{{ strings.joinButton }}</button>
      </div>
      <div v-if="isAdmin" class="admin-controls-block">
        <button @click="previousQuestion" class="control-button">{{ strings.previousQuestionButton }}</button>
        <button @click="nextQuestion" class="control-button">{{ strings.nextQuestionButton }}</button>
        <button @click="toggleStatistics" class="control-button">{{ strings.showStatisticsButton }}</button>
        <button @click="clearStatistics" class="control-button">{{ strings.clearStatisticsButton }}</button>
      </div>
      <div v-if="question" class="question-block">
        <h2 class="question-text">{{ question.text }}</h2>
        <ul class="options-list">
          <li v-for="(option, index) in question.options" :key="index" class="option-item">
            <input type="radio" :id="'option-' + index" :value="index" v-model="selectedAnswer" class="radio-input" />
            <label :for="'option-' + index" class="option-label" @click="selectAnswer(index)">{{ option }}</label>
          </li>
        </ul>
      </div>
      <div v-if="showStatistics" class="statistics-block">
        <h2 class="statistics-title">{{ strings.statisticsTitle }}</h2>
        <ul class="statistics-list">
          <li v-for="(count, user) in sortedStatistics" :key="user" class="statistics-item">
            {{ user }}: {{ count }} {{ strings.correctAnswers }}
          </li>
        </ul>
        <div v-if="Object.keys(sortedStatistics).length === 0" class="statistics-item">Победить может каждый</div>
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
      selectAnswer(answerIndex) {
        this.selectedAnswer = answerIndex;
        this.submitAnswer();
      },
      submitAnswer() {
        if (this.selectedAnswer !== null) {
          this.connection.invoke('SubmitAnswer', this.userName, this.selectedAnswer);
        }
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
  @import '../styles.css';
  </style>
  