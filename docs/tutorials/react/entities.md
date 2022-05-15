# Entities 

Let's start coding the app by defining the `Task` entity class.

The `Task` entity class will be used:
* As a model class for client-side code
* As a model class for server-side code
* By `remult` to generate API endpoints, API queries, and database commands

The `Task` entity class we're creating will have an auto-generated UUID `id` field a `title` field and a `completed` field. The entity's API route ("tasks") will include endpoints for all `CRUD` operations.

## Define the Model

1. Create a `shared` folder under the `src` folder. This folder will contain code shared between frontend and backend.

2. Create a file `Task.ts` in the `src/shared/` folder, with the following code:

  *src/shared/Task.ts*
  ```ts
  import { Entity, Fields } from "remult";

  @Entity("tasks", {
      allowApiCrud: true
  })
  export class Task {
      @Fields.uuid()
      id!: string;

      @Fields.string()
      title = '';

      @Fields.boolean()
      completed = false;
  }
  ```

The [@Entity](../../docs/ref_entity.md) decorator tells Remult this class is an entity class. The decorator accepts a `key` argument (used to name the API route and as a default database collection/table name), and an argument which implements the `EntityOptions` interface. We use an object literal to instantiate it, setting the [allowApiCrud](../../docs/ref_entity.md#allowapicrud) property to `true`.

The `@Fields.uuid` decorator tells remult to automatically generate an id using `uuid`. We mark that property as optional since the id will be auto-generated. 

The [@Fields.string](../../docs/ref_field.md) decorator tells Remult the `title` property is an entity data field of type `String`. This decorator is also used to define field-related properties and operations, discussed in the next sections of this tutorial and the same goes for `@Fields.boolean` and the `completed` property.

## Seed Test Data

Now that the `Task` entity is defined, we can use it to seed the database with some test data.

Add the highlighted code lines to `src/server/api.ts`.

*src/server/api.ts*
```ts{2,5-17}
import { remultExpress } from 'remult/remult-express';
import { Task } from '../shared/Task';

export const api = remultExpress({
    entities: [Task],
    initApi: async remult => {
        const taskRepo = remult.repo(Task);
        if (await taskRepo.count() === 0) {
            await taskRepo.insert([
                { title: "Task a" },
                { title: "Task b", completed: true },
                { title: "Task c" },
                { title: "Task d" },
                { title: "Task e", completed: true }
            ]);
        }
    }
});
```

The `initApi` callback is called only once, after a database connection is established and the server is ready to perform initialization operations.

`tasksRepo` is a Remult [Repository](../../docs/ref_repository.md) object used to fetch and create `Task` entity objects.

The code in `initApi` simply adds five new Tasks to the database if the current `count` is zero.

Saving the changes will cause the server to restart and seed the database with the test data. Navigate to the `tasks` API route at <http://localhost:3002/api/tasks> to see the data.

::: warning Wait, where is the backend database?
While remult supports [many relational and non-relational databases](https://remult.dev/docs/databases.html), in this tutorial we start by storing entity data in a backend **JSON file**. Notice that a `db` folder has been created under the root folder, with a `tasks.json` file containing the created tasks.
:::


## Display the Task List
Let's start developing the web app by displaying the list of existing tasks in a React component.

Replace the contents of `src/App.tsx` with the following code:

*src/App.tsx*
```tsx
import { useEffect, useState } from "react";
import { remult } from "./common";
import { Task } from "./shared/Task";

const taskRepo = remult.repo(Task);

async function fetchTasks() {
  return taskRepo.find();
}

function App() {
  const [tasks, setTasks] = useState<Task[]>([]);

  useEffect(() => {
    fetchTasks().then(setTasks);
  }, []);

  return (
    <div>
      {tasks.map(task => (
        <div key={task.id}>
          <input type="checkbox" checked={task.completed} />
          {task.title}
        </div>
      ))}
    </div>
  );
}

export default App;
```

Here's a quick overview of the different parts of the code snippet:

* `tasksRepo` is a Remult [Repository](../../docs/ref_repository.md) object used to fetch and create Task entity objects.
* The `fetchTasks` function uses the Remult repository's [find](../../docs/ref_repository.md#find) method to fetch tasks from the server.
* `tasks` is a Task array React state to hold the list of tasks.
* React's useEffect hook is used to call `fetchTasks` once when the React component is loaded.

After the browser refreshes, the list of tasks appears.