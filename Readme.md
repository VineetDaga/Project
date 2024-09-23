# VideoTube

This is a project which i am using to learn more about backend development in javascript 

## Lesson 1 - How to Connect a Database with MERN and Debugging It

There are two key things to keep in mind when connecting the database with the backend:

+ **Database Connection Errors**: Errors may occur when connecting to the database, such as incorrect credentials, network issues, or server misconfigurations. These errors can be effectively caught and handled using a `try-catch` block.
+ **Latency and Async/Await**: The database may be hosted on a server located far from the backend (e.g., in another continent), which could lead to latency issues. Using `async/await` allows the application to handle asynchronous code efficiently without blocking other operations while waiting for the database connection to establish.

---

### Database Connection Methods

There are two ways to connect the database:

+ **Writing the entire connection code inside `index.js`**: This approach may work for small projects, but it can clutter the `index.js` file as your project grows.
+ **Writing the connection code in a separate file and importing it into `index.js`**: This approach improves the modularity and cleanliness of the code. It makes the application easier to maintain and allows the separation of concerns.

The second approach is preferred for better code organization.

---

### Approach 1: Writing the Entire Code for Connecting the Database in a Single File (`index.js`)

In this approach, the entire database connection logic is written inside `index.js`. Below is an example of how this can be implemented:

```javascript
// Import necessary modules
import express from "express";
import mongoose from "mongoose";
import { DB_NAME } from "./constant.js"; 
// Importing the database name from a constants file

// Initializing the Express app
const app = express(); 

// Async IIFE (Immediately Invoked Function Expression) for database connection
(async () => {
    try {
        // Connect to the MongoDB database using mongoose, with environment variables
        await mongoose.connect(`${process.env.MONGODB_URI}/${DB_NAME}`);

        // Error handling for the app
        app.on('error', (error) => {
            console.log("Error: ", error);
            throw error;
        });

        // Start the server and listen on the defined port
        app.listen(process.env.PORT, () => {
            console.log(`App is listening on PORT ${process.env.PORT}`);
        });

    } catch (error) {
        // Catch any errors during the connection and log them
        console.error("ERROR: ", error);
        // Re-throw the error after logging it
        throw error; 
    }
})();
```
### Why Use This Approach?

- **Asynchronous Database Connection**: Using `async/await` prevents blocking the application while waiting for the database to connect. This ensures that other parts of the application can continue running smoothly.

- **Error Handling**: By wrapping the connection logic in a `try-catch` block, any errors are caught and handled gracefully. This prevents unexpected crashes and allows for proper debugging when things go wrong.

- **Port Listening**: The `app.listen()` method ensures that the app is only started on the specified port after the database connection has been successfully established, avoiding issues with starting the app before the database is ready.

- **Modular Code**: While this approach works for smaller applications, as projects grow, it is advisable to move the database connection code into a separate file. This enhances modularity, making the codebase easier to maintain, refactor, and scale.

### Approach 2: Writing the Database Connection Code in a Separate File

This approach improves code modularity and cleanliness by moving the database connection logic to a separate file, making it easier to maintain and scale as the project grows.


```javascript
// Import necessary modules
import mongoose from "mongoose";
import { DB_NAME } from "../constant.js"; 
// Importing the database name constant from another file

// Function to connect to MongoDB
const connectDB = async () => {
    try {
        // Connect to the MongoDB database using mongoose
        const connectionInstant = await mongoose.connect(`${process.env.MONGODB_URI}/${DB_NAME}`);

        // Log success message with the host of the connected database
        console.log(`\n MongoDB connected !! DB HOST : ${connectionInstant.connection.host}`);
        
    } catch (error) {
        // Log the error and exit the process if connection fails
        console.log("MONGODB connection Failed: ", error);
        process.exit(1);  // Exit the application with a failure code
    }
}

// Export the connectDB function to be used in other files
export default connectDB;
```
### Benefits of This Approach:
- **Separation of Concerns:** Moving the database connection logic to a separate file keeps the index.js file clean and focused on starting the server.
- **Modularity:** By separating the database connection, the function can be reused and tested independently.
- **Maintainability:** Easier to update or change the database connection logic without affecting the rest of the application.


### Server Initialization and Database Connection

In this section, we combine the database connection with the server startup process. This approach ensures that the server only starts listening for requests after the database connection is successfully established.


```javascript
// Import required modules
import dotenv from 'dotenv';  // To load environment variables
import connectDB from './db/index.js';  // Importing the connectDB function for database connection
import { app } from './app.js';  // Importing the Express app

// Configure dotenv to load environment variables from a specific path
dotenv.config({
    path: './env'  // Specify the path to the .env file
});

// Connect to the database and then start the server
connectDB()
    .then(() => {
        // If the database connection is successful, start the server
        app.listen(process.env.PORT || 8000, () => {
            console.log(`Server is running on port ${process.env.PORT || 8000}`);
        });

        // Error handling for server-level errors
        app.on('error', (error) => {
            console.log("Error: ", error);
            throw error;
        });
    })
    .catch((error) => {
        // If the database connection fails, log the error and throw it
        console.log("MONGO DB connection failed!!!: ", error);
        throw error;
    });

```
### Benefits of This Approach:

1. **Sequential Execution**:  
   The server only starts after the database connection is established, ensuring there’s no attempt to handle requests without a connected database.

2. **Error Handling**:  
   Proper error handling at both the database and server levels ensures that any issues are caught, logged, and the application does not run in an invalid state.

3. **Environment Configuration**:  
   Using `dotenv` allows for secure management of environment variables such as the database URI and port, ensuring sensitive information is not hardcoded in the source code.

## Lesson 2: Custom API response and Error Handling

This code is a basic setup for an Express.js server that includes middleware for handling CORS, parsing JSON and URL-encoded data, serving static files, and parsing cookies.

Here's a breakdown of each part:

### 1. Importing Dependencies
```javascript
import express from 'express';
import cookieParser from 'cookie-parser';
import cors from 'cors';
```
* **express:** The framework for building the Node.js server.
* **cookieParser:** Middleware to parse cookies attached to client requests.
* **cors:** Middleware to handle Cross-Origin Resource Sharing (CORS), allowing client applications from different domains to access your server.

### 2. Creating an Express Application
```javascript
const app = express();
```
* `app` is an instance of an Express application, which you will use to define routes, middleware, and other server logic.

### 3. Setting Up Middleware
1. CORS Configuration
     ```javascript
        app.use(cors({
        origin : process.env.CORS_ORIGIN,
        credentials : true,
    }));
    ```
- **CORS configuration:** This ensures that your API can handle requests from different origins.
    - `origin: process.env.CORS_ORIGIN:` Specifies the allowed origin(s). This value is pulled from the environment variable `CORS_ORIGIN`.
    - `credentials: true:` This option enables sending cookies and other credentials with the request.
2. JSON and URL-Encoded Data Parsing
```javascript
app.use(express.json({limit : "16kb"}));
app.use(express.urlencoded({extended : true, limit : "16kb"}));
```

- `express.json({limit: "16kb"})`: This middleware parses incoming JSON requests, limiting the payload to 16KB to prevent large uploads.
- `express.urlencoded({extended: true, limit: "16kb"})`: This middleware parses incoming URL-encoded data (typically from HTML forms). The `extended: true` option allows for rich objects and arrays to be encoded into the URL-encoded format.
3. Serving Static Files
```javascript
app.use(express.static('public'));
```
- This middleware serves static files (such as images, CSS, or JavaScript files) from the `public` directory.
4. Cookie Parsing
```javascript
app.use(cookieParser());
```
- `cookieParser()`: This middleware parses cookies from the incoming requests and makes them available on `req.cookies`.
### 4. Exporting the App
```javascript
export { app };
```
- The Express app is exported for use in another file, typically where the server would be started (e.g., `server.js`).

## CORS and Cookie Parser in Express

### 1. CORS (Cross-Origin Resource Sharing)
CORS is a security mechanism that allows servers to control which origins (domains) can access their resources. By default, web browsers restrict cross-origin requests (i.e., requests made from a different domain than the one serving the website). This is part of the browser's security model, known as the **same-origin policy**.

### Why CORS is Needed:
Imagine you have a frontend application hosted on `http://frontend.com`, and your backend API is hosted on `http://api.backend.com`. When the frontend application tries to make an HTTP request to the backend API, the browser will block it because they are on different origins.

To allow such requests, the server (`http://api.backend.com`) must explicitly specify which domains (origins) can access its resources. This is where **CORS** comes into play.

### How CORS Works in the Code:
You can use the `cors` middleware to set up CORS rules:

```javascript
app.use(cors({
    origin: process.env.CORS_ORIGIN,
    credentials: true,
}));
```
* **origin:** Specifies the allowed origins. This is often set dynamically based on an environment variable `(process.env.CORS_ORIGIN)`. For example, it could be set to `http://frontend.com`, meaning that only requests from this domain will be allowed.
* **credentials:** Set to `true` to allow cookies and authentication headers to be sent with requests. This is necessary if the frontend needs to make authenticated requests to the backend.

CORS essentially tells the browser: "It's okay to make a request from this origin." Without this, the browser would block the request due to security policies.

Example:
* **Allowed Origin:** If the CORS_ORIGIN environment variable is set to `http://frontend.com`, requests from `http://frontend.com` to the backend will be allowed.
* **Disallowed Origin:** If a request comes from `http://malicious.com`, it will be blocked because `http://malicious.com` is not an allowed origin.
