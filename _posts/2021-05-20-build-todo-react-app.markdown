---
layout: post
title: "Todo List: React + Typescript + Material UI"
date: 2021-05-20 20:35:46 -0800
categories: node express mysql
---

A Todo list is a simple great way to get started with any new language. I will go ahead and share my files and give brief instructions. 

Create our app:


# npx create-react-app AppName --template typescript


Useful website for material UI:


# https://material-ui.com/components/material-icons/


Dependencies for Material-UI: 

{% highlight tsx %}
npm install @material-ui/icons
npm install @material-ui/core
npm install @material-ui/lab
{% endhighlight %}

Going forward one of the things you will notice is that our CSS isn't on one giant CSS or SASS file. This is for ease of use, 
and to better keep up with styling rules. In general this is a better practice. As your projects get bigger you will appreciate this.

For our folder structure, we are going to have a component's folder. Every time you use the React framework you should do this with every project.
We will also have a model folder as well.

Components-
# Header.tsx
# TodoContext.tsx
# TodoList.tsx
# TodoTask.tsx

Model-
# Task.ts

Our validateTask function is there to ensure that people actually input a task. Being a Todo App this sort of thing is important to always include.
Our header Component allows us to change the task name, and to put a deadline(in terms of days) into place.

# Header.tsx

```css


import React, {FC, useState, ChangeEvent, useContext} from 'react';
import {Task} from '../model/Task'
import {TodoContext} from "./TodoContext";
import styled from "styled-components";
import Button from '@material-ui/core/Button';
import AddIcon from '@material-ui/icons/Add';
import {Box, TextField} from "@material-ui/core";


const StyledHeader = styled.div`
&.header {
    display: flex;
    width: 100%;
    justify-content: center;
    align-items: center;
    background-color: tomato;

    .input-container {
      display: flex;
      flex-flow: row nowrap;

      input {
        background-color: white;
        font-size: 17px;
        border: 1px solid grey;
      }

      .task-name-input {
        .MuiInputBase-root {
          height: 58px;
          background-color: white;
        }

        textarea {
        }
      }

      .deadline-input {
        input {
          max-width: 80px;
        }
      }
    }

    button {
      width: 70px;
      height: 59px;
      border-top-right-radius: 8px;
      border-bottom-right-radius: 8px;
      border: 1px solid grey;
    }

    .MuiSvgIcon-root {
      display: flex;
      justify-items: center;
      padding-left: 16px;
    }

}
`

```

```tsx
const validateTask = (task: Task): boolean => {
if (!task.taskName || task.taskName.trim().length === 0) {
alert("Must provide task name.");
return false;
}

    // deadline not required, but validate if present.
    if (task.deadline != null && task.deadline < 0) {
        alert("Deadline must be greater than zero.");
        return false;
    }

    return true;
}

export const Header: FC = () => {
const {todoList, setTodoList} = useContext(TodoContext);

    const [taskName, setTaskName] = useState<string>("");
    const [deadline, setDeadline] = useState<number | string>(""); // allow number | string because setting state for number input to undefined, does not clear the input.

    const onChangeTaskName = (event: ChangeEvent<HTMLInputElement>): void => {
        setTaskName(event.target.value);
    };

    const onChangeDeadline = (event: ChangeEvent<HTMLInputElement>): void => {
        const deadLine = Number(event.target.value);
        if (deadLine >= 0) {
            setDeadline(deadLine);
        } else {
            setDeadline("");
        }
    };

    const addTask = (): void => {
        const task: Task = {
            taskName: taskName,
            deadline: Number(deadline)
        }

        if (!validateTask(task)) {
            return;
        }

        // prepend task to todo list
        setTodoList([task].concat(todoList));

        setTaskName("");
        setDeadline("");
    };

    return (
        <StyledHeader className="header">
            <div className="input-container">
                <Box p="10px">
                    <label htmlFor="task">
                        <TextField
                            id="outlined-basic"
                            className="task-name-input"
                            multiline
                            name="task"
                            onChange={onChangeTaskName}
                            type="text"
                            placeholder="task"
                            value={taskName}
                            variant="outlined"
                        />
                    </label>
                </Box>

                <Box p="10px">
                    <label htmlFor="deadline">
                        <TextField
                            id="outlined-basic"
                            className="deadline-input"
                            name="deadline"
                            onChange={onChangeDeadline}
                            placeholder="days"
                            type="number"
                            value={deadline}
                            variant="outlined"
                        />
                    </label>
                </Box>

                <Box p="10px">
                    <Button
                        variant="contained"
                        color="primary"
                        onClick={addTask}
                        startIcon={<AddIcon style={{fontSize: '60px'}}/>}>
                    </Button>
                </Box>
            </div>
        </StyledHeader>
    );
}

```

We want to save our todo list to local storage. We will be building a context provider in our TodoContext.tsx file.

{% highlight tsx %}
import React, {FC, useEffect, useState} from 'react';
import {Task} from "../model/Task";
import {loadTodoList, saveTodoList} from "../localstorage";

interface TodoContextState {
todoList: Task[];
setTodoList: (todoList: Task[]) => void;
}

export const TodoContext = React.createContext({} as TodoContextState);

export const TodoContextProvider: FC = (props) => {
const [todoList, setTodoList] = useState<Task[]>([]);

    // when component loads, read todo list from local storage.
    useEffect(() => {
        setTodoList(loadTodoList());

    }, []);

    // a single place to handle saving todo list to local storage.
    // this is easier than remembering to do it in both Header.addTask() and TodoTask.completeTask()
    useEffect(() => {
        saveTodoList(todoList);

    }, [todoList]);

    return (
        <TodoContext.Provider
            value={{
                todoList, setTodoList,
            }}
        >
            {props.children}
        </TodoContext.Provider>
    );
};
{% endhighlight %}

Next up is our Todo list component. This will exist in our TodoList.tsx file.
 ```css


import React, {FC, useContext} from 'react'
import {Task} from "../model/Task";
import styled from "styled-components";
import {TodoTask} from "./TodoTask";
import {TodoContext} from "./TodoContext";

const StyledTodoList = styled.div`
&.todo-list {
flex: 80%;
width: 100%;
display: flex;
align-items: center;
padding-top: 15px;
flex-direction: column;
}
`;

```

```tsx


export const TodoList: FC = (props) => {
const { todoList, setTodoList } = useContext(TodoContext);

    const completeTask = (taskNameToDelete: string): void => {
        setTodoList(todoList.filter((task) => {
            return task.taskName !== taskNameToDelete
        }));
    };

    return (
        <StyledTodoList className="todo-list">
            {todoList.map((task: Task, key: number) => {
                return <TodoTask key={key} task={task} completeTask={completeTask}/>;
            })}
        </StyledTodoList>
    );
};

```

# TodoTask.tsx 

```css


import React, {FC, useState} from 'react'
import {Task} from "../model/Task";
import styled from "styled-components";
import Button from '@material-ui/core/Button';
import CheckIcon from '@material-ui/icons/Check';
import {Box} from "@material-ui/core";

const StyledTodoTask = styled.div`
&.task {
width: 500px;
min-height: 50px;
display: flex;
color: black;
margin: 15px;

    .content {
      flex: 80%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;

      &:hover {
        filter: brightness(90%);
      }

      .task-row-column {
        margin-right: 4px;
        display: grid;
        padding: 16px;
        place-items: center;
        width: 100%;
        height: 100%;
        font-size: 18px;
        background-color: #ddd;

        &.task-name {
          word-break: break-word;
          .toggle-text-button {
            color: #3f50b5;
            cursor: pointer;

            &:hover {
              text-decoration: underline;
            }
          }
        }

        &.task-days {
          text-wrap: none;
          max-width: 100px;
        }
      }
    }

    button {
      flex: 20%;
      height: 100%;
      border: none;
      border-top-right-radius: 8px;
      border-bottom-right-radius: 8px;
      color: white;
      cursor: pointer;

      &:hover {
        filter: brightness(120%);
      }

    }
}
`;

```
```tsx


const MAX_TASK_NAME_DISPLAY_LENGTH = 200;

const getTaskNamePreview = (taskName: string): string => taskName.substr(0, MAX_TASK_NAME_DISPLAY_LENGTH); // take the first 200 characters of the task name.

interface Props {
task: Task;
completeTask: (taskNameToDelete: string) => void;
}

export const TodoTask: FC<Props> = (props) => {
const {task, completeTask} = props;

    const [showMore, setShowMore] = useState<boolean>(false);

    const isTextShortened = task.taskName.length > MAX_TASK_NAME_DISPLAY_LENGTH;

    return (
        <StyledTodoTask className="task">
            <div className="content">
                <div className="task-row-column task-name">
                    {isTextShortened
                        ? (showMore ? task.taskName : getTaskNamePreview(task.taskName))
                        : task.taskName
                    }
                    {isTextShortened && (
                        <Box className="toggle-text-button"
                             onClick={() => setShowMore(!showMore)}
                        >
                            show {showMore ? 'less' : 'more'}
                        </Box>
                    )}
                </div>

                {/*
                   i.e. "if task.deadline != null then ...",
                */}
                {task.deadline != null && task.deadline > 0 && (
                    <div
                        className="task-row-column task-days">{task.deadline} {task.deadline !== 1 ? 'days' : 'day'}</div>
                )}
            </div>
            <Button
                onClick={() => completeTask(task.taskName)}
                variant="contained" color="primary"
                startIcon={<CheckIcon/>}
            >
                Done
            </Button>
        </StyledTodoTask>
    );
};
```

Next is our model for our todo app. We are now leaving our components folder and going into our models folder.

# Task.ts
```tsx


export interface Task {
    taskName: string;
    deadline?: number;
}
```
This is from our App.scss file


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
import {App} from './App';

ReactDOM.render(
<React.StrictMode>
<App />
</React.StrictMode>,
document.getElementById('root')
);

```

# App.tsx
```css

import React, {FC} from 'react';
import './App.scss';
import {Header} from "./components/Header";
import {TodoContextProvider} from "./components/TodoContext";
import {TodoList} from "./components/TodoList";
import styled, {ThemeProvider} from "styled-components";
import {createMuiTheme} from "@material-ui/core";

const StyledApp = styled.div`
&.app {
    display: flex;
    align-items: center;
    flex-direction: column;
    width: 100vw;
    height: 100vh;
    font-family: Arial, Helvetica, sans-serif;
}
`;
```

```tsx

export const App: FC = () => {
    const theme = createMuiTheme({
        palette: {
            primary: {
                light: '#757ce8',
                main: '#3f50b5',
                dark: '#002884',
                contrastText: '#fff',
            },
            secondary: {
                light: '#ff7961',
                main: '#f44336',
                dark: '#ba000d',
                contrastText: '#000',
            },
        },
    });

    return (
        <ThemeProvider theme={theme}>
            <TodoContextProvider>
                <StyledApp className="app">
                    <Header/>
                    <TodoList/>
                </StyledApp>
            </TodoContextProvider>
        </ThemeProvider>
    );
}

```

I want to give a shout out to PedroTech who hosted a video showing how to make an todo app. Changes were made to the code presented in the video.

# https://www.youtube.com/watch?v=bjnW2NLAofI