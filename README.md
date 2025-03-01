sh

Branding and Networking	5:30pm	7:30pm	Meeting Link
Master Class	2025-02-20	Master Class 2 - UI/UX Design Principles	5:30pm	7:30pm	Meeting Link
Master Class	2025-02-27	Master Class 3 - Containerization in Web Development

java script 

const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const dotenv = require("dotenv");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcryptjs");
const { Server } = require("socket.io");
const { createServer } = require("http");

dotenv.config();
const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, { cors: { origin: "*" } });

app.use(cors());
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });

// User Schema
const UserSchema = new mongoose.Schema({
  username: String,
  email: String,
  password: String,
});
const User = mongoose.model("User", UserSchema);

// Expense Schema
const ExpenseSchema = new mongoose.Schema({
  userId: String,
  title: String,
  amount: Number,
  category: String,
  date: Date,
});
const Expense = mongoose.model("Expense", ExpenseSchema);

// Signup API
app.post("/signup", async (req, res) => {
  const { username, email, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  const newUser = new User({ username, email, password: hashedPassword });
  await newUser.save();
  res.status(201).json({ message: "User registered!" });
});

// Login API
app.post("/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (user && (await bcrypt.compare(password, user.password))) {
    const token = jwt.sign({ userId: user._id }, "secretkey");
    res.json({ token });
  } else {
    res.status(400).json({ error: "Invalid credentials" });
  }
});

// Get Expenses
app.get("/expenses", async (req, res) => {
  const expenses = await Expense.find();
  res.json(expenses);
});

// Create Expense
app.post("/expenses", async (req, res) => {
  const newExpense = new Expense(req.body);
  await newExpense.save();
  io.emit("expenseUpdated");
  res.status(201).json(newExpense);
});

// Update Expense
app.put("/expenses/:id", async (req, res) => {
  await Expense.findByIdAndUpdate(req.params.id, req.body);
  io.emit("expenseUpdated");
  res.json({ message: "Expense updated!" });
});

// Delete Expense
app.delete("/expenses/:id", async (req, res) => {
  await Expense.findByIdAndDelete(req.params.id);
  io.emit("expenseUpdated");
  res.json({ message: "Expense deleted!" });
});

// Start Server
const PORT = process.env.PORT || 5000;
httpServer.listen(PORT, () => console.log(`Server running on port ${PORT}`));

sh


npx create-react-app client  
cd client  
npm install axios socket.io-client react-router-dom @mui/material @emotion/react @emotion/styled

import React, { useEffect, useState } from "react";
import axios from "axios";
import io from "socket.io-client";

const socket = io("http://localhost:5000");

function App() {
  const [expenses, setExpenses] = useState([]);

  useEffect(() => {
    fetchExpenses();
    socket.on("expenseUpdated", fetchExpenses);
  }, []);

  const fetchExpenses = async () => {
    const { data } = await axios.get("http://localhost:5000/expenses");
    setExpenses(data);
  };

  return (
    <div>
      <h2>Expense Tracker</h2>
      <ul>
        {expenses.map((exp) => (
          <li key={exp._id}>{exp.title} - ${exp.amount}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;

FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
EXPOSE 5000

FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
EXPOSE 3000

version: "3"
services:
  backend:
    build: ./backend
    ports:
      - "5000:5000"
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"

      <meta name="description" content="Track your expenses with real-time updates" />
<meta name="keywords" content="expense tracker, budget management" />

<meta property="og:title" content="Expense Manager" />
<meta property="og:description" content="Track your expenses easily with real-time updates" />
<meta property="og:image" content="https://example.com/preview.png" />
