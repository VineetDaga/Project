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

```javascript
// Importing the Express framework to create an HTTP server
import express from 'express';       
 // Importing cookie-parser middleware to handle cookies
import cookieParser from 'cookie-parser';
// Importing CORS middleware to handle cross-origin requests
import cors from 'cors';             

 // Creating an instance of the Express application
const app = express();              

// Enabling CORS (Cross-Origin Resource Sharing)
app.use(cors({
    // Setting the allowed origin from environment variables
    origin : process.env.CORS_ORIGIN,    
    // Enabling credentials (cookies, authorization headers) to be sent
    credentials : true,                  
}));

// Parsing incoming requests with JSON payloads, limiting the request size to 16kb
app.use(express.json({limit : "16kb"}));

// Parsing incoming requests with URL-encoded payloads, with a size limit of 16kb
app.use(express.urlencoded({extended : true, limit : "16kb"}));

// Serving static files from the 'public' directory
app.use(express.static('public'));

// Using cookie-parser to parse cookies from incoming requests and make them available in req.cookies
app.use(cookieParser());

// Exporting the app instance to use it in other files
export {app}; 
```

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

## 2. Cookie Parser
Cookies are small pieces of data stored on the client (browser), often used for session management, user authentication, and tracking user preferences. When a client makes a request to the server, cookies are sent along with the request.

The `cookie-parser` middleware in Express makes it easier to read cookies sent by the client and access their values in the server.

### Why Cookies are Used:
- **Session Management:** Storing session IDs to track logged-in users.
- **User Preferences:** Storing settings like language preferences or theme choices.
- **Authentication:** Cookies can hold tokens (e.g., JWT or session tokens) that the server uses to verify the user's identity.

### How `cookie-parser` Works:
```javascript
app.use(cookieParser());
```
- When a client sends a request to the server, any cookies stored in the browser are automatically included in the request headers.
- The `cookie-parser` middleware parses these cookies from the request and makes them available in `req.cookies`.

### Example:
If a client sends a request with the header:
```javascript
cookie: sessionId=abcd1234; theme=dark
```
The cookie-parser middleware will parse this and provide it as a JavaScript object:

``` javascript
req.cookies = {
    sessionId: 'abcd1234',
    theme: 'dark'
};
```

You can now easily access the values of sessionId or theme in your route handlers.

### Summary
* **CORS:** Ensures your backend can handle requests from different origins, usually a frontend on a different domain. It also allows the transmission of credentials like cookies between the frontend and backend.
* **Cookie-Parser:** Parses cookies sent in HTTP requests and makes them accessible to the server for managing sessions, user data, or preferences

## What are Middlewares in JavaScript (Backend)?

**Middleware** is a function in JavaScript backend frameworks like **Express.js** that sits between the request from the client and the response from the server. It processes or handles the request before passing it to the next middleware function or route handler. Middleware functions can modify the `request` and `response` objects, terminate the request-response cycle, or pass control to the next middleware function in the stack.

In Express.js, middleware can be used for various purposes such as:
- **Logging** request details.
- **Authenticating** users.
- **Parsing** incoming request bodies (JSON, URL-encoded data).
- **Serving static files**.
- **Handling errors**.

Middleware functions are executed in sequence, and they either:
1. **End the request-response cycle** (by sending a response to the client), or
2. **Pass control** to the next middleware function using `next()`.

### How Middleware Functions Work in JavaScript Backend (Express.js)

A middleware function takes three arguments:
- `req`: The request object (information about the HTTP request).
- `res`: The response object (allows you to send a response back to the client).
- `next`: A function that allows the request to proceed to the next middleware in the chain.

### Types of Middleware in Express.js

1. **Application-Level Middleware**: Applied at the application level for all routes or specific routes.

    ```js
    const express = require('express');
    const app = express();

    // Application-level middleware
    app.use((req, res, next) => {
        console.log('Request URL:', req.url);
        next(); // Pass control to the next middleware or route handler
    });
    ```

2. **Router-Level Middleware**: Works similarly to application-level middleware, but applied to specific route instances.

    ```js
    const express = require('express');
    const router = express.Router();

    // Router-level middleware
    router.use((req, res, next) => {
        console.log('Request Method:', req.method);
        next();
    });

    app.use('/api', router);
    ```

3. **Built-in Middleware**: Express.js has some built-in middleware for common tasks:
   - `express.json()`: Parses incoming JSON requests.
   - `express.urlencoded()`: Parses URL-encoded data.
   - `express.static()`: Serves static files like HTML, CSS, and images.

    ```js
    // Built-in middleware to serve static files
    app.use(express.static('public'));

    // Built-in middleware to parse JSON
    app.use(express.json());
    ```

4. **Third-Party Middleware**: Middleware developed by third parties for additional functionality, such as:
   - `cookie-parser`: Parses cookies from incoming requests.
   - `cors`: Enables Cross-Origin Resource Sharing (CORS).

    ```js
    const cookieParser = require('cookie-parser');
    const cors = require('cors');

    // Using third-party middleware
    app.use(cors());
    app.use(cookieParser());
    ```

5. **Error-Handling Middleware**: A special type of middleware designed to handle errors. It has four arguments: `err`, `req`, `res`, `next`.

    ```js
    // Error-handling middleware
    app.use((err, req, res, next) => {
        console.error(err.stack);
        res.status(500).send('Something went wrong!');
    });
    ```

### Middleware Example

In this example, we have three middleware functions:
- **Logger**: Logs each request's method and URL.
- **Authenticator**: Simulates authentication by checking a query parameter.
- **Request Handler**: Sends a response.

```js
const express = require('express');
const app = express();

// Middleware 1: Logger
app.use((req, res, next) => {
    console.log(`${req.method} ${req.url}`);
    next(); // Proceed to the next middleware
});

// Middleware 2: Authenticator
app.use((req, res, next) => {
    if (req.query.auth === 'true') {
        next(); // Proceed to the next middleware
    } else {
        res.status(403).send('Authentication required');
    }
});

// Middleware 3: Request handler
app.get('/', (req, res) => {
    res.send('Welcome, authenticated user!');
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

### Flow of the Above Example:
1. The **Logger** middleware logs the method and URL of every incoming request.
2. The **Authenticator** middleware checks if the request contains the `auth=true` query parameter. If not, it sends a 403 response.
3. If authentication passes, the final middleware **handles the request** and sends a response.

### Summary of How Middleware Works:
- **Middleware functions** can be applied globally or to specific routes.
- They **modify requests** (e.g., parsing, adding data) and **send responses** or pass control to the next function.
- Middleware can be **custom** or **pre-built** (like parsing, logging, CORS handling).
- They run in sequence and **terminate the request** or pass it on using `next()`.

### Conclusion
Middleware is the backbone of most Express.js applications, allowing for organized request handling, clean separation of concerns, and the modular addition of features like logging, authentication, and error handling.

## asyncHandler.js
The code you provided defines an **asynchronous error-handling middleware wrapper** for Express.js route handlers. This wrapper is designed to handle asynchronous functions (like `async`/`await` code) and catch any errors that might occur during the execution of these functions, passing those errors to the Express error-handling mechanism.

### Purpose of `asyncHandler`

In Express.js, route handlers can often involve asynchronous operations (e.g., reading from a database or calling an external API). These asynchronous operations can throw errors, and if not properly handled, can cause issues such as unhandled promise rejections.

Normally, when an error occurs in an `async` function, it must be caught manually using a `try-catch` block. This can lead to repetitive code in each route handler. The purpose of `asyncHandler` is to avoid writing `try-catch` blocks in every route and instead centralize error handling for asynchronous code in a reusable utility.

### Explanation of the Code

#### First Version of `asyncHandler`:
```js
const asyncHandler = (requestHandler) => {
    (req, res, next) => {
        Promise.resolve(requestHandler(req, res, next))
        .catch((err) => next(err));
    }
};
```

This version of the `asyncHandler` is a wrapper function for asynchronous route handlers. Here's how it works:

1. **Input**:
   - `asyncHandler` takes a `requestHandler` (the actual route handler function you want to execute).
   
2. **Promise.resolve**:
   - It wraps the `requestHandler` in a `Promise.resolve`. If `requestHandler` returns a promise (which all `async` functions do), it will either resolve or reject that promise.
   
3. **Error Handling**:
   - If the `requestHandler`'s promise is rejected (i.e., an error occurs), the `.catch()` block captures the error and passes it to the `next(err)` function. This passes the error to Express's built-in error-handling middleware, ensuring that errors don't crash the application and are properly handled.

4. **Execution**:
   - This wrapper executes the `requestHandler` and deals with any errors by forwarding them to the next middleware in the chain (which is typically an error handler).

#### Second Version of `asyncHandler`:
```js
const asyncHandler = (fn) => async (req, res, next) => {
    try {
        await fn(req, res, next);
    } catch (error) {
        res.status(error.code || 500).json({
            success: false,
            message: error.message,
        });
    }
};
```

This second version works similarly but is written in a more explicit `async`/`await` style:

1. **Input**:
   - Like the first version, it takes a function `fn` (the route handler) as input.

2. **Async/Await**:
   - It uses `async`/`await` to run the `fn` (the actual route handler). `await fn(req, res, next)` ensures that the code inside `fn` is executed asynchronously, waiting for any asynchronous operations to complete.

3. **Error Handling (Try-Catch)**:
   - The `try-catch` block is used to catch any errors that occur during the execution of `fn`. If an error is thrown, it is caught by the `catch` block.
   
4. **Custom Error Response**:
   - When an error occurs, it responds with an error status (`error.code` or `500` by default) and a JSON response containing the error message. This is a user-friendly way of returning error details to the client.

### How the Wrapper is Used

#### Without `asyncHandler`:
Without this wrapper, each route handler that involves asynchronous code would need to manually handle errors, like so:

```js
app.get('/example', async (req, res, next) => {
    try {
        const data = await someAsyncOperation();
        res.json(data);
    } catch (err) {
        next(err); // Manually passing errors to the error-handling middleware
    }
});
```

This leads to a lot of repetitive error handling with `try-catch` blocks.

#### With `asyncHandler`:
Using `asyncHandler`, the route can be simplified. You no longer need to write a `try-catch` block for every route handler:

```js
app.get('/example', asyncHandler(async (req, res, next) => {
    const data = await someAsyncOperation();
    res.json(data);
}));
```

The `asyncHandler` will automatically catch any errors from `someAsyncOperation()` and pass them to the error-handling middleware without the need for `try-catch` inside every route handler.

### Key Benefits

1. **Centralized Error Handling**: The `asyncHandler` wrapper makes it easier to centralize error handling logic for asynchronous code, ensuring consistency across your application.
   
2. **Cleaner Route Handlers**: You don't need to write repetitive `try-catch` blocks inside every asynchronous route. This results in cleaner and more readable code.

3. **Integration with Express Error-Handling**: When `asyncHandler` catches an error, it automatically forwards it to Express's built-in error-handling middleware by calling `next(err)`. This ensures that errors are handled according to your application's global error-handling logic.

### How the Error is Passed to Express

If an error occurs inside an `async` route handler, `asyncHandler` catches the error and calls `next(err)`.

- When `next(err)` is called, Express will automatically skip all remaining non-error middleware and go directly to error-handling middleware, such as:

```js
app.use((err, req, res, next) => {
    res.status(err.status || 500).json({
        success: false,
        message: err.message || 'Internal Server Error',
    });
});
```

This ensures that errors are handled gracefully and can be logged or responded to appropriately.

### Summary

- `asyncHandler` is a utility function that wraps asynchronous route handlers, allowing you to automatically catch errors and pass them to Express's error-handling middleware.
- It simplifies route handlers by removing the need for `try-catch` blocks.
- The first version uses `Promise.resolve` and `.catch` to handle promise-based errors, while the second version uses `async`/`await` and `try-catch`.
- Both versions help ensure that any errors in asynchronous code are centrally managed and don't crash the application.

## Overview of the `ApiError` Class
```javascript
class ApiError extends Error{
    constructor(
        statusCode,
        message = "Something Went Wrong",
        errors = [],
        stack = "",
        
    ){
        super(message)
        this.statusCode = statusCode;
        this.errors = errors;
        this.data = null;
        this.message = message;
        this.success = false;

        if(stack){
            this.stack = stack
        }else{
            Error.captureStackTrace(this,this.constructor)
        }
    }
}

export {ApiError};
```


The `ApiError` class you have defined extends the built-in JavaScript `Error` class, which is commonly used to handle and represent errors in JavaScript applications. The goal of this class is to provide a structured and standardized way to manage errors that occur in your application, particularly in an API or backend service. Let's break down the functionality in detail:

### 1. **Class Definition**:
```js
class ApiError extends Error {
```
- The `ApiError` class inherits from the native `Error` class using `extends`. This means that `ApiError` will have the properties and methods of `Error` but can also have additional custom behavior.
- The class is specifically designed to be used in an API or backend environment where errors need to carry additional context, such as an HTTP status code or detailed error messages.

### 2. **Constructor**:
```js
constructor(
    statusCode,
    message = "Something Went Wrong",
    errors = [],
    stack = ""
)
```
- **statusCode**: Represents the HTTP status code related to the error (e.g., `404` for Not Found, `500` for Internal Server Error). This is essential for providing context to the client or consumer of the API on the type of error.
  
- **message**: A human-readable error message. It defaults to `"Something Went Wrong"` if no custom message is provided. This is passed to the parent `Error` class through the `super()` call, which ensures that the `message` property is set on the error instance.

- **errors**: This is an array meant to store additional error details, such as field-specific validation errors or any additional context that might be useful for debugging or for the client consuming the API. It defaults to an empty array.

- **stack**: The stack trace that shows the origin of the error in the code. If a custom stack trace is provided, it will be assigned to the error; otherwise, the standard stack trace will be captured by calling `Error.captureStackTrace`.

### 3. **Super Call**:
```js
super(message);
```
- The `super()` method is called to pass the error message to the parent `Error` class, ensuring that the standard `Error` behavior is retained and that the error message is set correctly on the instance.

### 4. **Custom Properties**:
```js
this.statusCode = statusCode;
this.errors = errors;
this.data = null;
this.message = message;
this.success = false;
```
- **statusCode**: Stores the HTTP status code passed to the constructor. This is used to provide more specific context to the client about what went wrong.

- **errors**: Stores any additional error information or validation errors, often useful for debugging or reporting more specific issues to the client.

- **data**: Defaults to `null` and can be used later to hold any data associated with the error if needed. This could be additional error-related data.

- **message**: This is the error message that can be customized when creating an instance of `ApiError`. It overrides the parent `Error`'s `message` property (though `super(message)` ensures the `Error` class's functionality is preserved).

- **success**: Always set to `false` for errors, since an error generally means the request failed. This is a common pattern in APIs to indicate whether the operation succeeded or failed.

### 5. **Handling Stack Trace**:
```js
if (stack) {
    this.stack = stack;
} else {
    Error.captureStackTrace(this, this.constructor);
}
```
- If a `stack` trace is provided, the constructor assigns it to the `this.stack` property, allowing for custom stack traces to be attached to the error.

- If no stack trace is provided, the `Error.captureStackTrace()` method is called to generate the stack trace automatically. This captures the point in the code where the error was instantiated, allowing for easier debugging.

### 6. **Example Usage**:
You would use the `ApiError` class in your code like this:

```js
import { ApiError } from './ApiError';

app.get('/some-endpoint', async (req, res, next) => {
    try {
        // Some logic that might throw an error
        throw new ApiError(404, "Resource Not Found");
    } catch (error) {
        next(error); // Passes the error to the error-handling middleware
    }
});
```
In this example:
- If a 404 error occurs (e.g., the requested resource isn't found), the `ApiError` class is used to throw an error with a specific status code and message.
- This error can then be caught and passed to Express's error-handling middleware, which can respond to the client with the appropriate status code and message.

### 7. **Error Handling Middleware**:
In an Express app, you would typically have an error-handling middleware to capture and respond to errors like this:

```js
app.use((err, req, res, next) => {
    if (err instanceof ApiError) {
        res.status(err.statusCode).json({
            success: err.success,
            message: err.message,
            errors: err.errors,
        });
    } else {
        res.status(500).json({
            success: false,
            message: "Internal Server Error",
        });
    }
});
```
- If an `ApiError` is thrown, the middleware checks if the error is an instance of `ApiError`. It then uses the `statusCode`, `message`, and `errors` to form a structured response to the client.
- If the error is not an instance of `ApiError`, it falls back to a generic `500 Internal Server Error`.

### Summary

- The `ApiError` class extends the default `Error` class to include custom properties such as `statusCode`, `errors`, and `success`, which are useful for API error responses.
- It allows you to handle both standard and custom errors in a structured way and makes it easier to standardize error responses in an API.
- The stack trace is either provided or automatically captured, and all errors thrown using this class can be passed to centralized error-handling middleware for further processing and response.

## Overview of the `ApiResponse` Class

```javascript
class ApiResponse {
    constructor(statusCode,data,message = "Success"){
        this.statusCode = statusCode;
        this.data = data;
        this.message = message
        this.success = statusCode < 400;
    }
}

export {ApiResponse};
```

The `ApiResponse` class is a utility designed to standardize responses sent from an API. It simplifies the process of creating consistent and structured responses, which can enhance the clarity of communication between the server and clients.

#### Key Components:

1. **Constructor Parameters**:
   - **statusCode**: An HTTP status code indicating the result of the request (e.g., `200` for success, `404` for not found).
   - **data**: The main payload of the response, which can contain the requested data or any relevant information.
   - **message**: A custom message that describes the response. It defaults to "Success" if not provided.

2. **Properties**:
   - **statusCode**: Stores the HTTP status code of the response.
   - **data**: Holds the data being returned to the client.
   - **message**: A descriptive message regarding the response (e.g., "Success" or any custom message).
   - **success**: A boolean indicating whether the response indicates a successful operation. This is determined by checking if the `statusCode` is less than `400` (i.e., status codes in the `200` range).

#### Example Usage:
```javascript
// Creating a successful response
const successResponse = new ApiResponse(200, { user: "John Doe" });

// Creating an error response (e.g., bad request)
const errorResponse = new ApiResponse(400, null, "Bad Request");

// Response structure
console.log(successResponse);
// Output: { statusCode: 200, data: { user: 'John Doe' }, message: 'Success', success: true }
```

### Summary
The `ApiResponse` class provides a standardized way to structure API responses, improving the consistency of responses sent from a server. By encapsulating the status code, data, message, and success status, it simplifies the handling of API responses in a clear and manageable format.

## Lecture 9: User and Video model with hooks and JWT

## **USERMODEL**
Here’s a detailed explanation of the User model code, breaking down each part for a comprehensive understanding:

### 1. Imports

```javascript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';
import mongoose, { Schema } from 'mongoose';
```

- **`jsonwebtoken`**: A library used to create and verify JSON Web Tokens (JWT). This is essential for handling user authentication in web applications.
  
- **`bcrypt`**: A library that provides hashing functions for passwords. It helps securely store passwords by converting them into a fixed-length hash that cannot be easily reversed.

- **`mongoose`**: An Object Data Modeling (ODM) library for MongoDB and Node.js, which provides a straightforward way to define schemas, interact with MongoDB, and manage data models.

### 2. User Schema Definition

```javascript
const userSchema = new Schema({
    username: {
        type: String,
        required: true,
        unique: true,
        lowercase: true,
        trim: true,
        index: true,
    },
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true,
        trim: true,
    },
    fullName: {
        type: String,
        required: true,
        trim: true,
        index: true,
    },
    avatar: {
        type: String, //cloudinary URL for avatar
        required: true,
    },
    coverImage: {
        type: String, //cloudinary URL for cover image
    },
    password: {
        type: String,
        required: [true, "Please provide password to proceed"],
    },
    watchHistory: [{
        type: Schema.Types.ObjectId,
        ref: "Video",
    }],
    refreshToken: {
        type: String,
    }
}, {
    timestamps: true,
});
```

- **Schema Definition**: This defines the structure of the user document in the MongoDB collection. Each property corresponds to a field in the database.

  - **`username`**: 
    - Type: String
    - **`required`**: Ensures a username is provided.
    - **`unique`**: No two users can share the same username.
    - **`lowercase`**: Stores usernames in lowercase.
    - **`trim`**: Removes any leading or trailing whitespace.
    - **`index`**: Creates an index for faster searches based on username.

  - **`email`**: Similar constraints as `username`, ensuring uniqueness and proper formatting.

  - **`fullName`**: Requires a name, also indexed for quicker searches.

  - **`avatar`**: A URL pointing to the user's avatar image, required for the user profile.

  - **`coverImage`**: Optional URL for a cover image, providing additional customization.

  - **`password`**: Required field that will store the hashed password.

  - **`watchHistory`**: An array of ObjectIDs referencing the `Video` model, allowing the tracking of videos watched by the user.

  - **`refreshToken`**: A string for storing the user's refresh token, used in session management.

  - **`timestamps`**: Automatically adds `createdAt` and `updatedAt` fields to the schema, providing insights into when the document was created or last updated.

### 3. Pre-save Middleware

```javascript
userSchema.pre("save", async function(next) {
    if (!this.isModified("password")) return next();
    this.password = await bcrypt.hash(this.password, 10);
});
```

- **Purpose**: This middleware function is executed before saving a user document to the database.

- **Functionality**:
  - **`this.isModified("password")`**: Checks if the password field has been modified. If not, it calls `next()` to continue with the save operation without any changes.
  - If the password is modified, it hashes the password using bcrypt's `hash` method with a salt round of 10, ensuring that the password is stored securely.

### 4. Instance Methods

#### 4.1. Password Validation

```javascript
userSchema.methods.isPasswordCorrect = async function(password) {
    return await bcrypt.compare(password, this.password);
}
```

- **Purpose**: This method checks whether a provided plain-text password matches the hashed password stored in the database.

- **How it Works**:
  - Uses `bcrypt.compare` to compare the given password with the hashed password (`this.password`).
  - Returns `true` if the passwords match, enabling user authentication.

#### 4.2. Generate Access Token

```javascript
userSchema.methods.generateAccessToken = function() {
    return jwt.sign(
        {
            _id: this._id,
            email: this.email,
            username: this.username,
            fullName: this.fullName,
        },
        process.env.ACCESS_TOKEN_SECRET, {
            expiresIn: process.env.ACCESS_TOKEN_EXPIRY
        }
    );
}
```

- **Purpose**: Generates a JWT for the user, allowing them to authenticate requests to protected routes.

- **How it Works**:
  - The JWT includes the user's ID, email, username, and full name in its payload.
  - The token is signed with a secret stored in environment variables, with an expiration time specified in the same environment variables.
  - The generated token can then be sent to the client, allowing the user to make authenticated requests.

#### 4.3. Generate Refresh Token

```javascript
userSchema.methods.generateRefreshToken = function() {
    return jwt.sign(
        {
            _id: this._id,
        },
        process.env.REFRESH_TOKEN_SECRET, {
            expiresIn: process.env.REFRESH_TOKEN_EXPIRY
        }
    );
}
```

- **Purpose**: Similar to the access token, this method generates a refresh token for the user.

- **How it Works**:
  - The refresh token contains the user’s ID and is signed with a different secret, also stored in environment variables.
  - The refresh token typically has a longer expiration time than the access token and is used to obtain new access tokens without requiring the user to log in again.

### 5. Exporting the Model

```javascript
export const User = mongoose.model("User", userSchema);
```

- This line exports the User model, allowing it to be imported and used in other parts of the application.
- The model provides an interface for creating, reading, updating, and deleting user documents in the MongoDB collection.

### Summary of Features

1. **User Management**: The schema defines how user data is stored and validated in the database, ensuring integrity and structure.
   
2. **Password Security**: Passwords are hashed before storage, making it difficult for attackers to retrieve plain-text passwords even if they gain access to the database.

3. **Authentication**: The model supports generating access and refresh tokens, enabling secure, stateless authentication in web applications.

4. **Data Relationships**: The `watchHistory` field allows for complex data interactions, linking users to their watched videos.

5. **Middleware**: Pre-save hooks enhance functionality by ensuring that passwords are always hashed before saving to the database.

This User model provides a solid foundation for user authentication and management in a Node.js application, leveraging the capabilities of Mongoose, JWT, and bcrypt to ensure security and performance.

## **Video Model**
Here's a detailed explanation of the `Video` model code, breaking down each part for a comprehensive understanding:

### 1. Imports

```javascript
import mongoose, { Schema } from "mongoose";
import mongooseAggregatePaginate from "mongoose-aggregate-paginate-v2";
```

- **`mongoose`**: An Object Data Modeling (ODM) library for MongoDB and Node.js, which provides a schema-based solution to model application data.
  
- **`mongooseAggregatePaginate`**: A plugin for Mongoose that allows you to perform pagination on aggregation queries, enhancing the capabilities of data retrieval in your application.

### 2. Video Schema Definition

```javascript
const videoSchema = new Schema({
    videoFile: {
        type: String, // cloudinary URL for the video file
        required: true,
    },
    thumbnail: {
        type: String, // cloudinary URL for the thumbnail image
        required: true,
    },
    title: {
        type: String,
        required: true,
    },
    description: {
        type: String,
        required: true,
    },
    owner: {
        type: Schema.Types.ObjectId,
        ref: "User", // Reference to the User model
    },
    duration: {
        type: Number, // Duration of the video in seconds
        required: true,
    },
    views: {
        type: Number,
        default: 0, // Default value for views
    },
    isPublished: {
        type: Boolean,
        required: true,
    }
}, { timestamps: true });
```

- **Schema Definition**: This defines the structure of the video document in the MongoDB collection. Each property corresponds to a field in the database.

  - **`videoFile`**: 
    - Type: String
    - **`required`**: Ensures that a video file URL is provided. Typically this would be a URL to a video file hosted on a service like Cloudinary.

  - **`thumbnail`**: 
    - Type: String
    - **`required`**: Similar to `videoFile`, it stores the URL for the video's thumbnail image.

  - **`title`**: 
    - Type: String
    - **`required`**: Represents the title of the video.

  - **`description`**: 
    - Type: String
    - **`required`**: A brief description of the video's content.

  - **`owner`**: 
    - Type: Schema.Types.ObjectId
    - **`ref`**: "User" indicates that this field references a user document from the `User` model. It establishes a relationship between videos and their owners.

  - **`duration`**: 
    - Type: Number
    - **`required`**: Stores the duration of the video in seconds.

  - **`views`**: 
    - Type: Number
    - **`default`**: Initialized to 0. It counts how many times the video has been viewed.

  - **`isPublished`**: 
    - Type: Boolean
    - **`required`**: Indicates whether the video is published and visible to users.

- **`{ timestamps: true }`**: Automatically adds `createdAt` and `updatedAt` fields to the schema. This is useful for tracking when the document was created or last modified.

### 3. Plugin Integration

```javascript
videoSchema.plugin(mongooseAggregatePaginate);
```

- **Purpose**: This line integrates the `mongooseAggregatePaginate` plugin into the video schema. It enables the capability to perform aggregation queries with pagination support.
  
- **How it Works**: 
  - With this plugin, you can execute aggregation queries that will return paginated results. This is especially useful when dealing with large datasets, allowing you to retrieve a subset of documents efficiently.

### 4. Exporting the Model

```javascript
export const Video = mongoose.model("Video", videoSchema);
```

- This line exports the `Video` model, allowing it to be imported and used in other parts of the application.
- The model provides an interface for creating, reading, updating, and deleting video documents in the MongoDB collection.

### Summary of Features

1. **Video Management**: The schema defines how video data is structured and stored in the database, ensuring integrity and a consistent format.

2. **File Storage**: By using URLs (likely from Cloudinary), the application can efficiently manage video and image files without bloating the database with large binary data.

3. **Owner Reference**: The schema links videos to users, enabling functionalities like tracking which user uploaded a specific video.

4. **Views Tracking**: The `views` field helps maintain engagement metrics for videos, allowing for features like popular videos or statistics.

5. **Publication Status**: The `isPublished` field allows for easy management of video visibility, facilitating features like drafts or private videos.

6. **Aggregation and Pagination**: The integrated plugin enhances data retrieval capabilities, enabling efficient handling of large sets of videos with ease.

This `Video` model serves as a robust foundation for managing video content within a Node.js application, leveraging the capabilities of Mongoose to facilitate structured data storage and retrieval while integrating with video storage services.