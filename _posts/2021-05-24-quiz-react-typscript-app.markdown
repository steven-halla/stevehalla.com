---
layout: post
title: "Quiz App: React + Typescript + Styled Components"
date: 2021-05-24 20:35:46 -0800
categories: node express mysql
---

The following quiz app uses an external API to fetch questions and answers for quiz like questions.
There are catogorys and diffulculty levels that you can adjust. 

The website where we will get our API:

# https://opentdb.com/


For our dependencies with material ui:

```css
"@material-ui/core": "^4.11.4",
"@material-ui/icons": "^4.11.2",
"@material-ui/lab": "^4.0.0-alpha.58",
"@material/button": "^11.0.0",

```
We will have a components folder that is inside of the src folder. Inside our src folder we will have an API.ts,App.tsx,
Difficullty.ts, index.scss,index.tsx, and a utils.ts files.

#### QuestionCard.tsx

The QuestionCard file contains the props for the questions. dangerouslySetInnerHTML is  used due to the API that we will be using. If this is not set then some of the questions will return # and other characters that do not belong in our questions.


```css
import React from 'react';
import {AnswerObject} from "../App";
import styled from "styled-components";
import Button from '@material-ui/core/Button';


const StyledQuestionCard = styled.div`
&.question-card {
  .question-number {
  font-size: 2rem;
}

.question-text {
  font-size: 2rem;
}

.answer-button {
  margin: 4px;
  min-width: 200px;
  background-color: pink;
  color: #fff;

  &:hover {
    filter: brightness(90%);
    background-color: pink;
  }
 }
}`
```

```tsx


type Props = {
  question: string;
  answers: string[];
  callback: (event: React.MouseEvent<HTMLButtonElement>) => void;
  userAnswer: AnswerObject | undefined;
  questionNumber: number;
  totalQuestions: number;
}

export const QuestionCard: React.FC<Props> = (props) => {
    const {question, answers, callback, userAnswer, questionNumber, totalQuestions} = props;

  return (
    <StyledQuestionCard className="question-card">
      <p className="question-number">
        Question: {questionNumber} / {totalQuestions}
    </p>

    <p className="question-text" dangerouslySetInnerHTML={{__html: question}} />
      <div>
    {answers.map(answer => (
    <div key={answer}>
        <Button
                className="answer-button"
                variant="contained"
                color="default"
                disabled={userAnswer ? true : false}
                onClick={callback}
                value={answer}
        >
            <span dangerouslySetInnerHTML={{__html: answer}}/>
        </Button>
    </div>
    ))}
    </div>
  </StyledQuestionCard>
  );
};
```


# API.ts

For our interface we set everything to a string. We want this since our questions that are returned will be in string format.
Our API handles get request to the site that we are using. The function in this file handles this which returns a list of questions 
as well as the difficulty of the questions asked. You can set how hard the questions are on the website. Set yours to easy for testing purposes. 

```tsx
import {shuffleArray} from "./utils";
import {Difficulty} from "./Difficulty";

export interface Question {
  category: string;
  correct_answer: string;
  difficulty: string;
  incorrect_answers: string[];
  question: string;
  type: string;
}

export interface QuestionState extends Question {
  answers: string[];
}

export const fetchQuizQuestions = async (amount: number, difficulty: Difficulty): Promise<QuestionState[]> => {
  const endpoint = `https://opentdb.com/api.php?amount=${amount}&difficulty=${difficulty}&type=multiple`;
  const data = await (await fetch(endpoint)).json();
  return data.results.map((question: Question): QuestionState => ({
    ...question,
    answers: shuffleArray([
      ...question.incorrect_answers,
      question.correct_answer,
    ]),
  }));
};
```


# App.tsx

This file will handle how many questions that are returned to us. You will want to set your questions to 10 for testing purposes. 
Current state of what question we are on, score, the number of questions among other things will be handled here.
You will notice that Difficulty is set to easy, this is done for testing purposes. 
There is a nextQuestion function in this file. Pop Quiz: Can you guess where we will use this function. Stop reading and think for a minute. If your guess is an onClick button event then you are correct!

```css
import React, {FC, useState} from 'react';

import {fetchQuizQuestions, QuestionState} from "./API";
import {QuestionCard} from "./components/QuestionCard";
import {Difficulty} from "./Difficulty";
import styled from "styled-components";
import Button from '@material-ui/core/Button';

const StyledApp = styled.div`
  &.app {    
    width: 100%;
    height: 100vh;
    padding: 15px;
    display: flex;
    flex: 1 1 auto;
    flex-flow: column nowrap;
    justify-content: flex-start;
    align-items: center;
    text-align: center;
    background-image: url('beach-quotes.jpeg');
    background-position: center;
    background-repeat: no-repeat;
    background-size: cover;
    color: black;
  }

  .score {
    font-size: 2rem;
    margin: 0px;
  }

  .quiz-title {
    font-size: 2rem;
    font-family: Verdana;
  }

  .start-quiz-button {
    min-width: 200px;
    background-color: darkred;
    color: #ffffff;

    &:hover {
      filter: brightness(90%);
      background-color: darkred;
    }
  }

  .next-question-button {
    margin: 4px;
    min-width: 200px;
    background-color: aqua;
    color: #ffffff;

    &:hover {
      filter: brightness(90%);
      background-color: aqua;
  }
}`
```

```tsx


export interface AnswerObject {
  question: string;
  answer: string;
  correct: boolean;
  correctAnswer: string;
}

const TOTAL_QUESTIONS = 10;
export const App: FC = () => {

  const [loading, setLoading] = useState(false);
  const [questions, setQuestions] = useState<QuestionState[]>([]);
  const [questionNumber, setQuestionNumber] = useState(0);
  const [userAnswers, setUserAnswers] = useState<AnswerObject[]>([]);
  const [score, setScore] = useState<number>(0);
  const [gameOver, setGameOVer] = useState(true);

  console.log(questions);

  const startTrivia = async () => {
    setLoading(true);
    setGameOVer(false);

    const newQuestions: QuestionState[] = await fetchQuizQuestions(
      TOTAL_QUESTIONS,
      Difficulty.EASY
    );

    setQuestions(newQuestions);
    setScore(0);
    setUserAnswers([]);
    setQuestionNumber(0);
    setLoading(false);
  };

  const checkAnswer = (event: React.MouseEvent<HTMLButtonElement>) => {
    if (!gameOver) {
      //Users Answer
      const answer = event.currentTarget.value;
      //check answer against correct answer
      const correct = questions[questionNumber].correct_answer === answer;
      console.log(`answer is correct? ${correct}`);
      console.log(`${questions[questionNumber].correct_answer}`);
      console.log(questions[questionNumber]);
      //add  score if answer is correct
      if (correct) setScore(prev => prev + 1);
      //save answer in the array for user answers
      const answerObject = {
        question: questions[questionNumber].question,
        answer,
        correct,
        correctAnswer: questions[questionNumber].correct_answer,
      };
      setUserAnswers(prev => [...prev, answerObject]);
    }
  };

  const nextQuestion = () => {
    // move on to the next question if not the last question
    const nextQuestion = questionNumber + 1;

    if (nextQuestion === TOTAL_QUESTIONS) {
      setGameOVer(true);
    } else {
      setQuestionNumber(nextQuestion);
    }
  };

  return (
    <StyledApp className="app">
      <p className="quiz-title">React Quiz</p>
      {gameOver || userAnswers.length === TOTAL_QUESTIONS ? (

        <Button
          className="start-quiz-button"
          variant="contained"
          color="default"
          onClick={startTrivia}
        >
          Start
        </Button>
      ) : null}

      {!gameOver ? <p className="score"> Score: {score}</p> : null}
      {loading && <p>Loading Questions....</p>}

      {!loading && !gameOver && (
        <QuestionCard questionNumber={questionNumber + 1}
                      totalQuestions={TOTAL_QUESTIONS}
                      question={questions[questionNumber].question}
                      answers={questions[questionNumber].answers}
                      userAnswer={userAnswers ? userAnswers[questionNumber] : undefined}
                      callback={checkAnswer}
        />
      )}

      {!gameOver && !loading && userAnswers.length === questionNumber + 1 && questionNumber !== TOTAL_QUESTIONS - 1 ? (
        <Button
          className="next-question-button"
          variant="contained"
          color="default"
          onClick={nextQuestion}
        >
          Next Question
        </Button>
      ) : null}

    </StyledApp>
  );
};

export default App;
```


# Difficulty.ts

```tsx
export enum Difficulty {
  EASY = "easy",
  MEDIUM = "medium",
  HARD = "hard",
}
```


# index.scss

```css
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    padding: 0;
}

```


# index.tsx


```tsx
import React from 'react';
import ReactDOM from 'react-dom';
import './Index.scss';
import App from './App';

ReactDOM.render(
  <React.StrictMode>
    <App/>
  </React.StrictMode>,
  document.getElementById('root')
);


```

# utils.tsx

```tsx
export const shuffleArray = (array: any[]) =>
  [...array].sort(() => Math.random() - 0.5);
```
I want to give a shout out to freecodeCamp.org . Note that I do make changes from their original code base.
```
https://www.youtube.com/watch?v=F2JCjVSZlG0
```

