# MERN Stack — Complete Project Building Guide

A practical, reusable guide for planning, building, running, debugging, and extending full-stack applications with **MongoDB, Express.js, React, and Node.js**.

This is a general-purpose reference. Follow the core CRUD workflow for almost any application, then replace the example `Task` resource with your own domain, such as students, products, employees, notes, expenses, bookings, or orders.

---

## Table of contents

1. [What MERN means](#1-what-mern-means)
2. [How a MERN application works](#2-how-a-mern-application-works)
3. [Plan any application](#3-plan-any-application)
4. [Prerequisites and installation](#4-prerequisites-and-installation)
5. [Create the project](#5-create-the-project)
6. [Backend: Express and Node.js](#6-backend-express-and-nodejs)
7. [Database: MongoDB and Mongoose](#7-database-mongodb-and-mongoose)
8. [Build REST APIs](#8-build-rest-apis)
9. [Frontend: React](#9-frontend-react)
10. [Connect React to the backend](#10-connect-react-to-the-backend)
11. [Complete CRUD flow](#11-complete-crud-flow)
12. [Validation, errors, and loading states](#12-validation-errors-and-loading-states)
13. [Authentication and authorization](#13-authentication-and-authorization)
14. [Relationships and references](#14-relationships-and-references)
15. [Search, filtering, sorting, and pagination](#15-search-filtering-sorting-and-pagination)
16. [File and image uploads](#16-file-and-image-uploads)
17. [Useful React and JavaScript concepts](#17-useful-react-and-javascript-concepts)
18. [Environment variables and security](#18-environment-variables-and-security)
19. [Testing and API debugging](#19-testing-and-api-debugging)
20. [Deployment overview](#20-deployment-overview)
21. [How to adapt this guide to any project](#21-how-to-adapt-this-guide-to-any-project)
22. [Common errors and fixes](#22-common-errors-and-fixes)
23. [Final project checklist](#23-final-project-checklist)

---

# 1. What MERN means

MERN is a JavaScript-based full-stack technology stack.

| Technology | Responsibility | What it does |
|---|---|---|
| MongoDB | Database | Stores application data as documents |
| Express.js | Backend framework | Defines routes and handles HTTP requests |
| React | Frontend library | Builds the user interface |
| Node.js | Runtime | Runs JavaScript on the server |

**Typical responsibility split:**

- React displays data and collects user input.
- Express receives HTTP requests and selects the correct route.
- Node.js runs the backend application.
- Mongoose communicates with MongoDB and works with database models.
- MongoDB stores the data persistently.

MERN does not force a particular folder structure or architecture. The structure in this guide is one maintainable way to organize a small-to-medium application.

# 2. How a MERN application works

Example: a user creates a task.

1. The user fills a form in React.
2. React sends a `POST` request to the Express API.
3. Express receives the request and runs the matching route/controller.
4. The controller validates the input.
5. Mongoose creates a document in MongoDB.
6. MongoDB stores the document and returns the result.
7. Express sends a JSON response.
8. React updates the screen.

```text
Browser
  |
  | User action
  v
React UI
  |
  | HTTP request (fetch / Axios)
  v
Express route
  |
  v
Controller / business logic
  |
  | Mongoose
  v
MongoDB
  |
  | Result
  v
Express JSON response
  |
  v
React updates the UI
```

## Important terms

- **Client:** the frontend application running in the browser.
- **Server:** the backend application receiving requests.
- **API:** a defined way for two applications to communicate.
- **Endpoint:** a URL and HTTP method, such as `GET /api/tasks`.
- **Request:** information sent by the client.
- **Response:** information returned by the server.
- **JSON:** a common format for sending structured data.
- **CRUD:** Create, Read, Update, Delete.
- **REST API:** an API style that uses resources and HTTP methods.

# 3. Plan any application

Before coding, define the following.

## 3.1 Identify the main resource

A resource is the main type of data your application manages.

Examples:

| Application | Main resource | Example fields |
|---|---|---|
| Task manager | Task | title, description, status |
| Student manager | Student | name, email, course |
| Product catalogue | Product | name, price, category |
| Expense tracker | Expense | title, amount, date |
| Notes application | Note | title, content |
| Employee manager | Employee | name, role, department |

## 3.2 Define the operations

For most resources, begin with:

- Create one record.
- Read all records.
- Read one record.
- Update one record.
- Delete one record.

## 3.3 Sketch the screens

A simple first version might contain:

- List page
- Create form
- Edit form
- Details view
- Navigation/header
- Empty, loading, and error states

## 3.4 Design the data

For a task manager:

```js
{
  title: "Complete project",
  description: "Build the task management application",
  status: "pending",
  createdAt: "2026-01-01T10:00:00.000Z"
}
```

Decide which fields are required, their data types, and any allowed values before creating the database model.

# 4. Prerequisites and installation

Install these tools:

- **Node.js LTS:** https://nodejs.org/
- **VS Code:** https://code.visualstudio.com/
- **MongoDB Community Server** or a MongoDB Atlas cluster: https://www.mongodb.com/
- **Git:** https://git-scm.com/
- **Postman** or another API client: https://www.postman.com/

Check Node and npm:

```bash
node -v
npm -v
```

You should understand basic JavaScript before building a MERN application:

- Variables: `const`, `let`
- Functions and arrow functions
- Objects and arrays
- Destructuring
- Array methods: `map`, `filter`, `find`, `reduce`
- Modules: `import` and `export`
- Promises and `async` / `await`
- `try` / `catch`
- Basic HTML and CSS

# 5. Create the project

Create a root folder with separate frontend and backend applications.

```bash
mkdir mern-project
cd mern-project
mkdir backend
```

The final structure will look like:

```text
mern-project/
├── backend/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   └── taskController.js
│   ├── middleware/
│   │   └── errorMiddleware.js
│   ├── models/
│   │   └── Task.js
│   ├── routes/
│   │   └── taskRoutes.js
│   ├── .env
│   ├── .gitignore
│   ├── package.json
│   └── server.js
│
└── frontend/
    ├── src/
    │   ├── components/
    │   │   └── TaskForm.jsx
    │   ├── services/
    │   │   └── taskService.js
    │   ├── App.jsx
    │   ├── main.jsx
    │   └── index.css
    ├── .env
    ├── package.json
    └── index.html
```

This guide uses **ES modules** (`import` / `export`) in the backend.

# 6. Backend: Express and Node.js

## 6.1 Initialize the backend

```bash
cd backend
npm init -y
npm install express mongoose dotenv cors
npm install --save-dev nodemon
```

Open `backend/package.json` and configure it:

```json
{
  "name": "mern-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

Keep any dependencies npm added to your `package.json`; the example above shows the important fields and scripts.

## 6.2 Create the Express server

Create `backend/server.js`:

```js
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import connectDB from "./config/db.js";
import taskRoutes from "./routes/taskRoutes.js";
import { notFound, errorHandler } from "./middleware/errorMiddleware.js";

dotenv.config();

const app = express();

// Middleware
app.use(cors({
  origin: process.env.CLIENT_URL || "http://localhost:5173"
}));
app.use(express.json());

// Health-check route
app.get("/", (req, res) => {
  res.json({ message: "API is running" });
});

// API routes
app.use("/api/tasks", taskRoutes);

// Error handlers (after routes)
app.use(notFound);
app.use(errorHandler);

const PORT = process.env.PORT || 5000;

const startServer = async () => {
  try {
    await connectDB();

    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
  } catch (error) {
    console.error("Unable to start server:", error.message);
    process.exit(1);
  }
};

startServer();
```

### What is middleware?

Middleware is a function that runs during the request-response cycle.

Examples:

- `express.json()` reads JSON request bodies.
- `cors()` controls which browser origins can access the API.
- Authentication middleware checks whether a user is logged in.
- Error middleware formats errors.

A middleware generally receives `req`, `res`, and `next`.

```js
const exampleMiddleware = (req, res, next) => {
  console.log(req.method, req.url);
  next();
};
```

`next()` passes control to the next middleware or route handler.

# 7. Database: MongoDB and Mongoose

## 7.1 MongoDB basics

MongoDB stores data as documents inside collections.

| Relational database | MongoDB |
|---|---|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| Primary key | `_id` |

Example document:

```json
{
  "_id": "generated-id",
  "title": "Complete project",
  "description": "Build the application",
  "status": "pending"
}
```

MongoDB automatically creates an `_id` for each document.

## 7.2 Connect to MongoDB

Create `backend/config/db.js`:

```js
import mongoose from "mongoose";

const connectDB = async () => {
  try {
    const connection = await mongoose.connect(process.env.MONGO_URI);
    console.log(`MongoDB connected: ${connection.connection.host}`);
  } catch (error) {
    console.error(`MongoDB connection failed: ${error.message}`);
    throw error;
  }
};

export default connectDB;
```

Create `backend/.env`:

```env
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/mern_project
CLIENT_URL=http://localhost:5173
```

For MongoDB Atlas, use the connection string from your Atlas cluster instead of the local URI. Keep credentials private.

## 7.3 Create a Mongoose model

A model defines the shape and validation rules for documents in a collection.

Create `backend/models/Task.js`:

```js
import mongoose from "mongoose";

const taskSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: [true, "Title is required"],
      trim: true,
      maxlength: [100, "Title cannot exceed 100 characters"]
    },
    description: {
      type: String,
      trim: true,
      default: ""
    },
    status: {
      type: String,
      enum: ["pending", "in-progress", "completed"],
      default: "pending"
    }
  },
  {
    timestamps: true
  }
);

const Task = mongoose.model("Task", taskSchema);

export default Task;
```

### Schema options

- `type`: data type.
- `required`: field must be provided.
- `default`: value used when none is supplied.
- `trim`: removes extra whitespace at the beginning and end.
- `enum`: restricts a field to allowed values.
- `maxlength`: limits string length.
- `timestamps`: automatically adds `createdAt` and `updatedAt`.

Common Mongoose types include `String`, `Number`, `Boolean`, `Date`, `Array`, and `mongoose.Schema.Types.ObjectId`.

# 8. Build REST APIs

A clean backend separates:

- **Routes:** define URL and HTTP method.
- **Controllers:** contain request handling and application logic.
- **Models:** interact with the database.

## 8.1 Create the controller

Create `backend/controllers/taskController.js`:

```js
import Task from "../models/Task.js";

// GET /api/tasks
export const getTasks = async (req, res, next) => {
  try {
    const tasks = await Task.find().sort({ createdAt: -1 });
    res.status(200).json(tasks);
  } catch (error) {
    next(error);
  }
};

// GET /api/tasks/:id
export const getTaskById = async (req, res, next) => {
  try {
    const task = await Task.findById(req.params.id);

    if (!task) {
      return res.status(404).json({ message: "Task not found" });
    }

    res.status(200).json(task);
  } catch (error) {
    next(error);
  }
};

// POST /api/tasks
export const createTask = async (req, res, next) => {
  try {
    const { title, description, status } = req.body;

    if (!title || !title.trim()) {
      return res.status(400).json({ message: "Title is required" });
    }

    const task = await Task.create({
      title,
      description,
      status
    });

    res.status(201).json(task);
  } catch (error) {
    next(error);
  }
};

// PUT /api/tasks/:id
export const updateTask = async (req, res, next) => {
  try {
    const allowedFields = ["title", "description", "status"];
    const updates = {};

    for (const field of allowedFields) {
      if (req.body[field] !== undefined) {
        updates[field] = req.body[field];
      }
    }

    const task = await Task.findByIdAndUpdate(
      req.params.id,
      updates,
      {
        new: true,
        runValidators: true
      }
    );

    if (!task) {
      return res.status(404).json({ message: "Task not found" });
    }

    res.status(200).json(task);
  } catch (error) {
    next(error);
  }
};

// DELETE /api/tasks/:id
export const deleteTask = async (req, res, next) => {
  try {
    const task = await Task.findByIdAndDelete(req.params.id);

    if (!task) {
      return res.status(404).json({ message: "Task not found" });
    }

    res.status(200).json({ message: "Task deleted successfully" });
  } catch (error) {
    next(error);
  }
};
```

### Understand the request object

- `req.params`: values from URL parameters, such as `:id`.
- `req.query`: values from the query string, such as `?status=pending`.
- `req.body`: data sent in the request body.
- `req.headers`: request metadata, such as an authorization token.

Example request:

```http
PUT /api/tasks/123
Content-Type: application/json
```

```json
{
  "title": "Updated task",
  "status": "completed"
}
```

`req.params.id` is `123`, while `req.body` contains the submitted fields.

## 8.2 Create routes

Create `backend/routes/taskRoutes.js`:

```js
import express from "express";
import {
  getTasks,
  getTaskById,
  createTask,
  updateTask,
  deleteTask
} from "../controllers/taskController.js";

const router = express.Router();

router.route("/")
  .get(getTasks)
  .post(createTask);

router.route("/:id")
  .get(getTaskById)
  .put(updateTask)
  .delete(deleteTask);

export default router;
```

## 8.3 HTTP methods and status codes

| Method | Purpose | Typical status |
|---|---|---|
| GET | Read data | 200 |
| POST | Create data | 201 |
| PUT | Replace/update data | 200 |
| PATCH | Partially update data | 200 |
| DELETE | Delete data | 200 or 204 |

Common status codes:

- `200 OK`: request succeeded.
- `201 Created`: a record was created.
- `204 No Content`: succeeded with no response body.
- `400 Bad Request`: invalid input.
- `401 Unauthorized`: authentication is missing or invalid.
- `403 Forbidden`: user is authenticated but not allowed.
- `404 Not Found`: resource does not exist.
- `409 Conflict`: request conflicts with existing data.
- `500 Internal Server Error`: unexpected server error.

## 8.4 Error middleware

Create `backend/middleware/errorMiddleware.js`:

```js
export const notFound = (req, res, next) => {
  res.status(404);
  next(new Error(`Route not found: ${req.originalUrl}`));
};

export const errorHandler = (err, req, res, next) => {
  const statusCode = res.statusCode >= 400 ? res.statusCode : 500;

  res.status(statusCode).json({
    message: err.message || "Server error"
  });
};
```

The error middleware should be registered after all API routes.

# 9. Frontend: React

## 9.1 Create the React application

From the root `mern-project` directory:

```bash
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm install lucide-react
```

Run the frontend:

```bash
npm run dev
```

Vite normally serves the application at `http://localhost:5173`.

## 9.2 React fundamentals

A React application is built from components.

```jsx
function Welcome() {
  return <h1>Welcome to my application</h1>;
}

export default Welcome;
```

### JSX

JSX lets you write HTML-like syntax inside JavaScript.

```jsx
const userName = "Aarav";

function App() {
  return <h1>Hello, {userName}</h1>;
}
```

### Props

Props pass information from a parent component to a child component.

```jsx
function UserCard({ name, role }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{role}</p>
    </div>
  );
}

// Usage:
<UserCard name="Aarav" role="Developer" />
```

### State with useState

State stores information that can change and cause the component to re-render.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

### Events

```jsx
function SaveButton() {
  const handleClick = () => {
    console.log("Button clicked");
  };

  return <button onClick={handleClick}>Save</button>;
}
```

### Rendering arrays

Use `map()` to display a list. Give each item a stable `key`.

```jsx
const names = ["Aarav", "Mira", "Kabir"];

function NameList() {
  return (
    <ul>
      {names.map((name) => (
        <li key={name}>{name}</li>
      ))}
    </ul>
  );
}
```

### Conditional rendering

```jsx
{isLoading ? <p>Loading...</p> : <p>Content loaded</p>}
```

### useEffect

`useEffect` runs side effects, such as fetching data when a component loads.

```jsx
import { useEffect } from "react";

useEffect(() => {
  // Side effect
}, []);
```

The empty dependency array means the effect is intended to run after the initial mount. If dependencies are included, the effect runs again when those values change.

# 10. Connect React to the backend

## 10.1 Configure the API URL

Create `frontend/.env`:

```env
VITE_API_URL=http://localhost:5000/api
```

Vite exposes variables prefixed with `VITE_` to frontend code. Never put private keys, database credentials, or server secrets in frontend environment variables.

Restart the Vite server after changing `.env`.

## 10.2 Create an API service

Create `frontend/src/services/taskService.js`:

```js
const API_URL = `${import.meta.env.VITE_API_URL}/tasks`;

const handleResponse = async (response) => {
  const data = await response.json().catch(() => ({}));

  if (!response.ok) {
    throw new Error(data.message || "Something went wrong");
  }

  return data;
};

export const getTasks = async () => {
  const response = await fetch(API_URL);
  return handleResponse(response);
};

export const createTask = async (taskData) => {
  const response = await fetch(API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(taskData)
  });

  return handleResponse(response);
};

export const updateTask = async (id, taskData) => {
  const response = await fetch(`${API_URL}/${id}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(taskData)
  });

  return handleResponse(response);
};

export const deleteTask = async (id) => {
  const response = await fetch(`${API_URL}/${id}`, {
    method: "DELETE"
  });

  return handleResponse(response);
};
```

A service file keeps HTTP communication separate from the UI.

## 10.3 Build the form component

Create `frontend/src/components/TaskForm.jsx`:

```jsx
import { useEffect, useState } from "react";

const emptyForm = {
  title: "",
  description: "",
  status: "pending"
};

function TaskForm({ onSubmit, editingTask, onCancel }) {
  const [formData, setFormData] = useState(emptyForm);

  useEffect(() => {
    if (editingTask) {
      setFormData({
        title: editingTask.title,
        description: editingTask.description || "",
        status: editingTask.status
      });
    } else {
      setFormData(emptyForm);
    }
  }, [editingTask]);

  const handleChange = (event) => {
    const { name, value } = event.target;

    setFormData((previous) => ({
      ...previous,
      [name]: value
    }));
  };

  const handleSubmit = (event) => {
    event.preventDefault();

    if (!formData.title.trim()) return;

    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="title"
        value={formData.title}
        onChange={handleChange}
        placeholder="Task title"
        required
      />

      <textarea
        name="description"
        value={formData.description}
        onChange={handleChange}
        placeholder="Description"
      />

      <select
        name="status"
        value={formData.status}
        onChange={handleChange}
      >
        <option value="pending">Pending</option>
        <option value="in-progress">In progress</option>
        <option value="completed">Completed</option>
      </select>

      <button type="submit">
        {editingTask ? "Update task" : "Create task"}
      </button>

      {editingTask && (
        <button type="button" onClick={onCancel}>
          Cancel
        </button>
      )}
    </form>
  );
}

export default TaskForm;
```

Important form concepts:

- `value` makes an input controlled by React state.
- `onChange` updates the state as the user types.
- `name` identifies which field changed.
- `event.preventDefault()` stops the browser's default form submission/reload.
- The spread operator preserves the other form fields.

# 11. Complete CRUD flow

Create `frontend/src/App.jsx`:

```jsx
import { useEffect, useState } from "react";
import TaskForm from "./components/TaskForm";
import {
  getTasks,
  createTask,
  updateTask,
  deleteTask
} from "./services/taskService";

function App() {
  const [tasks, setTasks] = useState([]);
  const [editingTask, setEditingTask] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  const loadTasks = async () => {
    try {
      setError("");
      const data = await getTasks();
      setTasks(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    loadTasks();
  }, []);

  const handleSubmit = async (formData) => {
    try {
      setError("");

      if (editingTask) {
        const updated = await updateTask(editingTask._id, formData);

        setTasks((previous) =>
          previous.map((task) =>
            task._id === updated._id ? updated : task
          )
        );

        setEditingTask(null);
      } else {
        const created = await createTask(formData);
        setTasks((previous) => [created, ...previous]);
      }
    } catch (err) {
      setError(err.message);
    }
  };

  const handleDelete = async (id) => {
    const confirmed = window.confirm("Delete this task?");

    if (!confirmed) return;

    try {
      await deleteTask(id);
      setTasks((previous) => previous.filter((task) => task._id !== id));

      if (editingTask?._id === id) {
        setEditingTask(null);
      }
    } catch (err) {
      setError(err.message);
    }
  };

  if (loading) return <p>Loading tasks...</p>;

  return (
    <main className="container">
      <h1>Task Manager</h1>

      {error && <p role="alert">{error}</p>}

      <TaskForm
        onSubmit={handleSubmit}
        editingTask={editingTask}
        onCancel={() => setEditingTask(null)}
      />

      <section>
        <h2>Your tasks</h2>

        {tasks.length === 0 ? (
          <p>No tasks found. Create your first task.</p>
        ) : (
          tasks.map((task) => (
            <article className="task-card" key={task._id}>
              <h3>{task.title}</h3>
              <p>{task.description}</p>
              <p>Status: {task.status}</p>

              <button onClick={() => setEditingTask(task)}>
                Edit
              </button>

              <button onClick={() => handleDelete(task._id)}>
                Delete
              </button>
            </article>
          ))
        )}
      </section>
    </main>
  );
}

export default App;
```

Create `frontend/src/index.css`:

```css
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f5f6f8;
  color: #222;
}

.container {
  width: min(900px, 92%);
  margin: 40px auto;
}

form,
.task-card {
  display: grid;
  gap: 12px;
  padding: 18px;
  margin-bottom: 18px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 10px;
}

input,
textarea,
select,
button {
  padding: 10px 12px;
  font: inherit;
}

button {
  width: fit-content;
  cursor: pointer;
}
```

Ensure `frontend/src/main.jsx` imports the stylesheet:

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

## 11.1 Run the complete application

Use two terminals.

**Terminal 1 — backend**

```bash
cd mern-project/backend
npm run dev
```

**Terminal 2 — frontend**

```bash
cd mern-project/frontend
npm run dev
```

Check:

- Backend health check: `http://localhost:5000/`
- API list: `http://localhost:5000/api/tasks`
- Frontend: `http://localhost:5173/`

The backend must be connected to MongoDB before API operations will work.

# 12. Validation, errors, and loading states

A good application should handle more than the successful case.

## 12.1 Validate on both sides

Frontend validation improves user experience. Backend validation protects the data because API requests can be sent without using your React interface.

Example frontend validation:

```js
if (!title.trim()) {
  setError("Please enter a title");
  return;
}

if (title.trim().length > 100) {
  setError("Title must be 100 characters or fewer");
  return;
}
```

The Mongoose schema should still enforce the same important constraints.

## 12.2 Show loading and error states

```jsx
if (loading) return <p>Loading...</p>;

if (error) return <p role="alert">{error}</p>;
```

For forms, also consider a `saving` state so users cannot submit the same operation repeatedly.

## 12.3 Keep API responses consistent

A successful list request can return an array:

```json
[
  {
    "_id": "abc123",
    "title": "Example task",
    "status": "pending"
  }
]
```

An error can return:

```json
{
  "message": "Task not found"
}
```

Choose a response format and use it consistently across your API.

# 13. Authentication and authorization

CRUD applications that store private user data usually need authentication.

- **Authentication:** verifies who the user is.
- **Authorization:** decides what that user is allowed to do.
- **JWT:** a signed token commonly used to represent an authenticated session.
- **Hashing:** stores a one-way password hash instead of the original password.

Install common packages:

```bash
cd backend
npm install bcryptjs jsonwebtoken
```

A user model can look like:

```js
import mongoose from "mongoose";

const userSchema = new mongoose.Schema(
  {
    name: { type: String, required: true, trim: true },
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true
    },
    password: { type: String, required: true, select: false }
  },
  { timestamps: true }
);

export default mongoose.model("User", userSchema);
```

## 13.1 Hash a password

```js
import bcrypt from "bcryptjs";

const hashedPassword = await bcrypt.hash(password, 12);
```

Compare a submitted password during login:

```js
const isMatch = await bcrypt.compare(password, user.password);
```

Never store plain-text passwords.

## 13.2 Sign a token

```js
import jwt from "jsonwebtoken";

const token = jwt.sign(
  { userId: user._id },
  process.env.JWT_SECRET,
  { expiresIn: "1d" }
);
```

Add a long, random `JWT_SECRET` to the backend `.env`. Do not expose it to React.

## 13.3 Authentication middleware

```js
import jwt from "jsonwebtoken";

export const protect = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith("Bearer ")) {
    return res.status(401).json({ message: "Not authorized" });
  }

  try {
    const token = authHeader.split(" ")[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    req.user = { id: decoded.userId };
    next();
  } catch {
    res.status(401).json({ message: "Invalid or expired token" });
  }
};
```

Protect a route:

```js
router.get("/my-data", protect, getMyData);
```

For private resources, always check ownership on the backend. A valid token alone does not mean the user can access every record.

**Security note:** token storage and session design need care. For browser applications, secure, `HttpOnly`, `SameSite` cookies are often preferable to storing long-lived tokens in `localStorage`. Cookie-based authentication also requires appropriate CSRF protections. Use HTTPS in production.

# 14. Relationships and references

MongoDB documents can contain embedded data or references to other documents.

Example: many tasks belong to one user.

```js
const taskSchema = new mongoose.Schema({
  title: { type: String, required: true },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
    required: true
  }
});
```

Query tasks belonging to one user:

```js
const tasks = await Task.find({ user: req.user.id });
```

Populate the referenced user when needed:

```js
const tasks = await Task.find().populate("user", "name email");
```

Use references when related data is managed independently or shared by multiple documents. Embedded documents can be useful when data is usually read and updated together.

# 15. Search, filtering, sorting, and pagination

Query parameters are useful for list pages.

Example URL:

```text
GET /api/tasks?status=pending&search=project&page=1&limit=10
```

Example controller logic:

```js
export const getTasks = async (req, res, next) => {
  try {
    const {
      status,
      search,
      page = 1,
      limit = 10
    } = req.query;

    const filter = {};

    if (status) {
      filter.status = status;
    }

    if (search) {
      filter.title = { $regex: search, $options: "i" };
    }

    const pageNumber = Math.max(1, Number(page));
    const pageSize = Math.min(100, Math.max(1, Number(limit)));

    const [tasks, total] = await Promise.all([
      Task.find(filter)
        .sort({ createdAt: -1 })
        .skip((pageNumber - 1) * pageSize)
        .limit(pageSize),
      Task.countDocuments(filter)
    ]);

    res.json({
      data: tasks,
      pagination: {
        total,
        page: pageNumber,
        limit: pageSize,
        totalPages: Math.ceil(total / pageSize)
      }
    });
  } catch (error) {
    next(error);
  }
};
```

If you use this version, replace the earlier `getTasks` controller with it. The frontend service must then read `response.data` instead of assuming the API returns an array directly.

For production search, consider indexes and carefully validate user-supplied search expressions.

# 16. File and image uploads

A common upload flow is:

1. React lets the user select a file.
2. React sends a `multipart/form-data` request.
3. Express parses the upload using middleware such as Multer.
4. The server validates file size and type.
5. The file is stored in object storage or an image service.
6. The database stores the resulting file URL and metadata.

Install Multer:

```bash
cd backend
npm install multer
```

Basic upload middleware:

```js
import multer from "multer";

const storage = multer.memoryStorage();

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 },
  fileFilter: (req, file, callback) => {
    const allowed = ["image/jpeg", "image/png", "image/webp"];

    if (!allowed.includes(file.mimetype)) {
      return callback(new Error("Unsupported file type"));
    }

    callback(null, true);
  }
});

export default upload;
```

Use it on a route:

```js
router.post("/upload", upload.single("image"), uploadController);
```

`memoryStorage()` makes the file available in `req.file.buffer`; it does not save it permanently. Send it to a suitable storage provider or use a carefully configured disk/object-storage solution. Validate actual file content on the server rather than trusting the MIME type alone.

# 17. Useful React and JavaScript concepts

## 17.1 Array methods

```js
const numbers = [1, 2, 3, 4, 5];

// Transform each element
const doubled = numbers.map((number) => number * 2);

// Keep matching elements
const even = numbers.filter((number) => number % 2 === 0);

// Find one matching element
const found = numbers.find((number) => number === 3);

// Calculate one result
const total = numbers.reduce((sum, number) => sum + number, 0);
```

Typical React use:

```jsx
tasks.map((task) => <p key={task._id}>{task.title}</p>)
```

## 17.2 Destructuring

```js
const user = { name: "Mira", age: 21 };
const { name, age } = user;

const [first, second] = [10, 20];
```

## 17.3 Spread operator

```js
const updatedUser = { ...user, age: 22 };
```

In React, avoid directly mutating state arrays or objects. Create a new value instead:

```js
setTasks((previous) => [...previous, newTask]);

setTasks((previous) =>
  previous.filter((task) => task._id !== id)
);
```

## 17.4 Async and await

An asynchronous function can wait for a Promise without blocking the entire JavaScript runtime.

```js
const loadData = async () => {
  try {
    const response = await fetch("/api/data");
    const data = await response.json();
    return data;
  } catch (error) {
    console.error(error);
  }
};
```

## 17.5 Modules

Export code from one file and import it elsewhere.

```js
// math.js
export const add = (a, b) => a + b;

// app.js
import { add } from "./math.js";
```

## 17.6 React hooks at a glance

| Hook | Main use |
|---|---|
| `useState` | Store component state |
| `useEffect` | Run side effects |
| `useContext` | Share values across components |
| `useRef` | Hold a mutable value or DOM reference |
| `useMemo` | Cache a computed value |
| `useCallback` | Cache a function reference |

Use hooks only inside React function components or custom hooks.

# 18. Environment variables and security

## 18.1 Backend `.gitignore`

Create `backend/.gitignore`:

```gitignore
node_modules/
.env
.env.*
!.env.example
coverage/
```

Create `frontend/.gitignore` if one is not already present:

```gitignore
node_modules/
dist/
.env
.env.*
!.env.example
```

Example environment templates can be committed without real secrets.

`backend/.env.example`:

```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
CLIENT_URL=http://localhost:5173
JWT_SECRET=replace_with_a_long_random_secret
```

`frontend/.env.example`:

```env
VITE_API_URL=http://localhost:5000/api
```

## 18.2 Essential security practices

- Never commit `.env` files or passwords.
- Never expose database credentials or JWT secrets in frontend code.
- Validate all data on the backend.
- Restrict CORS to trusted origins in production.
- Use HTTPS in production.
- Hash passwords using a password-hashing algorithm such as bcrypt.
- Check resource ownership before returning or modifying private records.
- Limit upload sizes and validate uploaded content.
- Keep dependencies updated.
- Avoid returning stack traces or internal secrets to clients.
- Add rate limiting to sensitive endpoints such as login and password reset.
- Use secure cookies and CSRF protection where applicable.

# 19. Testing and API debugging

## 19.1 Test backend endpoints first

Before connecting React, test your API using Postman or another API client.

| Action | Method | URL | Example body |
|---|---|---|---|
| Get all | GET | `/api/tasks` | — |
| Get one | GET | `/api/tasks/:id` | — |
| Create | POST | `/api/tasks` | `{ "title": "Learn MERN" }` |
| Update | PUT | `/api/tasks/:id` | `{ "title": "Learn React" }` |
| Delete | DELETE | `/api/tasks/:id` | — |

For POST and PUT, set `Content-Type: application/json`.

## 19.2 Debug in layers

When something fails, check in this order:

1. **Browser:** Is the React component rendering? Check the browser console.
2. **Network:** Is the correct URL and HTTP method being called? Inspect the request and response.
3. **Frontend service:** Is the request body correctly formatted?
4. **Backend route:** Does the URL match the route?
5. **Controller:** Is the expected data present in `req.body`, `req.params`, or `req.query`?
6. **Database:** Is MongoDB connected? Does the model validation pass?
7. **Response:** Is the server returning the expected status and JSON?

Useful temporary logs:

```js
console.log("Body:", req.body);
console.log("Params:", req.params);
console.log("Query:", req.query);
```

Remove unnecessary logs before deployment, especially if they could expose private data.

# 20. Deployment overview

A typical deployment separates the frontend, backend, and database.

| Layer | Common hosting choices |
|---|---|
| React frontend | Vercel, Netlify, static hosting |
| Node/Express backend | Render, Railway, a VPS, cloud hosting |
| MongoDB | MongoDB Atlas |

General deployment process:

1. Push the project to a Git repository.
2. Create a production MongoDB database.
3. Deploy the backend and configure its environment variables.
4. Configure production CORS with the frontend's real domain.
5. Deploy the frontend.
6. Set `VITE_API_URL` to the deployed backend API URL.
7. Test all CRUD operations, authentication, uploads, and error cases.
8. Check server logs and hosting health checks.

Do not use localhost URLs in production configuration. Do not commit secrets to Git.

# 21. How to adapt this guide to any project

The example uses `Task`, but the same architecture works for many applications.

## 21.1 Replace the resource

| Task manager example | Student manager example |
|---|---|
| `Task.js` | `Student.js` |
| `taskController.js` | `studentController.js` |
| `taskRoutes.js` | `studentRoutes.js` |
| `/api/tasks` | `/api/students` |
| `title` | `name` |
| `description` | `email` or `course` |
| `status` | `year` or `department` |

For a product catalogue, fields could be `name`, `price`, `description`, `category`, `stock`, and `imageUrl`.

For an expense tracker, fields could be `title`, `amount`, `category`, `date`, and `paymentMethod`.

## 21.2 Repeat the same workflow for each resource

For example, an e-commerce application might contain:

- User
- Product
- Category
- Cart
- Order
- Address

Each resource can have its own model, controller, and routes. Some resources also need relationships and business rules. An order, for example, should store a price snapshot so later product-price changes do not rewrite historical orders.

## 21.3 Add features incrementally

A useful progression:

1. Build one resource with complete CRUD.
2. Add search and filtering.
3. Add authentication.
4. Associate records with users.
5. Add form validation.
6. Add image upload if required.
7. Add pagination.
8. Improve UI and responsive design.
9. Add tests.
10. Deploy.

Do not build every advanced feature before the basic data flow works.

# 22. Common errors and fixes

| Problem | Likely cause | What to check |
|---|---|---|
| `Cannot find module` | Wrong import path or missing package | Check file path and run `npm install` |
| `ECONNREFUSED` | Server or database is not running | Check backend and MongoDB |
| MongoDB authentication error | Incorrect Atlas credentials | Check URI, password encoding, and database access |
| CORS error | Frontend origin is not allowed | Check backend CORS configuration |
| `req.body` is undefined | JSON middleware missing or wrong request format | Add `express.json()` and send JSON |
| API returns 404 | Wrong endpoint or route not mounted | Check route URL and `app.use()` |
| API returns 500 | Backend exception | Read terminal logs and error response |
| React page is blank | Component/import/runtime error | Check browser console |
| Changes do not appear | State not updated or stale data | Check state update and API response |
| Environment variable is undefined | Incorrect variable name or server not restarted | Check `.env` location and restart |
| `Cast to ObjectId failed` | Invalid MongoDB ID | Validate the ID before querying |
| Duplicate key error | Unique field already exists | Check indexes and handle conflicts |

# 23. Final project checklist

## Project structure

- [ ] Frontend and backend are separated.
- [ ] Routes, controllers, models, and configuration are organized.
- [ ] Reusable UI components are extracted where useful.

## Backend

- [ ] Express server starts correctly.
- [ ] MongoDB connection works.
- [ ] Models contain suitable validation.
- [ ] CRUD endpoints work.
- [ ] Invalid IDs and missing records are handled.
- [ ] Error middleware returns useful responses.

## Frontend

- [ ] React application renders correctly.
- [ ] Forms are controlled and validated.
- [ ] API calls work.
- [ ] Create, read, update, and delete work.
- [ ] Loading, empty, and error states are handled.
- [ ] UI works on mobile and desktop.

## Security and delivery

- [ ] Secrets are not committed.
- [ ] Backend validates all important input.
- [ ] Private records are protected by ownership checks.
- [ ] CORS is configured for the correct origins.
- [ ] Production environment variables are configured.
- [ ] Main flows have been tested after deployment.

---

## Quick reference: the complete MERN request flow

```text
React component
    |
    | User action
    v
React service (fetch / Axios)
    |
    | HTTP request + JSON
    v
Express route
    |
    v
Middleware (validation / authentication)
    |
    v
Controller (application logic)
    |
    v
Mongoose model
    |
    v
MongoDB
    |
    | Query result
    v
Controller
    |
    | JSON response + HTTP status
    v
React service
    |
    v
React state update
    |
    v
Updated UI
```

**Core idea:** most MERN applications use the same foundation: define data, create a Mongoose model, expose API routes, call those routes from React, and update the UI from the response. The domain-specific part is the data model and the business rules.
