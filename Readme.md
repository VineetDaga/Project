# DATABASE CONNECTION
There are two things we have to keep in mind while connecting the database with the backend 
1. there can be error when connecting the database to the backend => this can be solved by using try catch block
2. and the database is always on other continent => this can be solved by using async and await

there are two ways to connect the database
1. writing the whole code in index.js 
2. writing in another file and then importing it in the index.js

2nd approach is preffered as it makes code clean and enhance the modularity

# APPROACH 1
all done in the index file 

import express from "express";
import mongoose from "mongoose";
import {DB_NAME} from "./constant.js"
const app  = express();

( async () =>{
    try {
        await mongoose.connect(`${process.env.MONGODB_URI}/${DB_NAME}`);
        app.on('error',()=>{
            console.log("Error : ",error);
            throw error;
        })
        app.listen(process.env.PORT, () =>{
            console.log(`App is listening on PORT ${process.env.PORT}`);
        })
    } catch (error) {
        console.error("ERROR : ", error);
        throw error;
    }
})()