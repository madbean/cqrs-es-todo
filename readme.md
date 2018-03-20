# Todo-App

## Write side

### interface graphql
- todo-graphql
    - post /todos
    - put /todos/:id
    - put /todos/:id/title
    - put /todos/:id/description
    - put /todos/:id/state
    - delete /todos/:id

- global Types :
    - ActionResultEvent: {
        todoId: number,
        success: bool,
        errorMessage: string,
        status: number,
    }

- func createTodo(TodoInput payload)
    - TodoInput: {
        ownerId: number
        todoId: number,
        title: string,
        description: string,
        state: string,
    }
    - resolver: {
        const createTodoCommand = {
            userId: number,
            todo: {
                ...payload,
                timestamp
            },
        };
        publish('/createTodo', createTodoCommand);
    }
- func updateTodo(TodoUpdateInput payload)
    - TodoUpdateInput: {
        todoId: number,
        title: string,
        description: string,
        state: string,
    }
    - resolver: {
        const updateTodoCommand = {
            userId: number,
            todo: {
                ...payload,
                timestamp
            },
        };
        publish('/updateTodo', updateTodoCommand);
    }
- func updateTodoTitle(TodoTitleInput payload)
    - TodoTitleInput: {
        todoId: number,
        title: string,
    }
    - resolver: {
        const updateTodoTitleCommand = {
            userId: number,
            todo: {
                ...payload,
                timestamp
            },
        };
        publish('/updateTodoTitle', updateTodoTitleCommand);
    }
- func updateTodoDescription(TodoDescriptionInput payload)
    - TodoDescriptionInput: {
        todoId: number,
        description: string,
    }
    - resolver: {
        const updateTodoDescriptionCommand = {
            userId: number,
            todo: {
                ...payload,
                timestamp
            },
        };
        publish('/updateTodoDescription', updateTodoDescriptionCommand);
    }
- func updateTodoState(TodoStateInput payload)
    - TodoStateInput: {
        todoId: number,
        state: string,
    }
    - resolver: {
        const updateTodoStateCommand = {
            userId: number,
            todo: {
                ...payload,
                timestamp
            },
        };
        publish('/updateTodoState', updateTodoStateCommand);
    }
- func deleteTodo(number todoId)
    - resolver: {
        const deleteTodoCommand = {
            userId: number,
            todo: {
                todoId,
                timestamp
            },
        };
        publish('/deleteTodo', deleteTodoCommand);
    }


## Read Side

### interface graphql
- todo-graphql
    - get /todos/:id
    - get /todos

- func getTodo(number todoId) : Todo
    - Todo: {
        ownerId: number
        todoId: number,
        title: string,
        description: string,
        state: string,
        creationDate: date,
        lastUpdateDate: date,
    }
    - resolver: {
        todoService.getTodo(number todoId) : Todo
    }
- func getTodos() : [TodoLine]
    - TodoLine: {
        ownerId: number
        todoId: number,
        title: string,
        state: string,
    }
    - resolver: {
        todoService.getTodos() : [Todo]
    }

## Architecture

- domain
    - entities
        - todo
            - createTodo
            - updateTodo
            - updateTodoTitle
            - updateTodoDescription
            - updateTodoState
            - deleteTodo
    - todoReducer
        - toTodo(number todoId)
        - toTodos(string ownerId)
        - toTodoVersion(number todoId, number version)
        - toTodos(string ownerId, number version)
    - todoRepository
        - getById: ownerId =>
            db.loadEvents(ownerId).then(history => ({
                todoState: reduceToTodo(history),
                save: events => db.append(ownerId, history.length, events)
            })),
        - create: ownerId => ({
            save: events => db.append(ownerId, 0, events)
        })
    - types
        - command //=> io-ts check graphql-input
        - model //=> graphql-type
        - events
            - createTodo: [todoCreated, titleUpdated, descriptionUpdated]
                - todoCreated: {
                    type: 'todoCreated',
                    payload: {
                        ownerId: number,
                        todoId: number,
                        dateCreation: timestamp,
                    },
                }
                - titleUpdated: {
                    type: 'titleUpdated',
                    payload: {
                        todoId: number,
                        title: string,
                    },
                }
                - descriptionUpdated: {
                    type: 'descriptionUpdated',
                    payload: {
                        todoId: number,
                        description: string,
                    },
                }
            - updateTodo : [titleUpdated, descriptionUpdated, stateUpdated]
                - stateUpdated: {
                    type: 'stateUpdated',
                    payload: {
                        todoId: number,
                        state: string,
                    },
                }
            - updateTodoTitle : [titleUpdated]
            - updateTodoDescription : [descriptionUpdated]
            - updateTodoState : [stateUpdated]
            - deleteTodo : [todoDeleted]
                - todoDeleted: {
                    type: 'todoDeleted',
                    payload: {
                        todoId: number,
                        state: string,
                    },
                }



## Func command

