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
