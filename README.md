// Backend Setup with Express and MongoDB
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(express.json());
app.use(cors());

mongoose.connect(process.env.MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
})
.then(() => console.log('Connected to MongoDB successfully!'))
.catch(err => console.log('MongoDB Connection Error:', err));

const expenseRoutes = require('./routes/expenseRoutes');
app.use('/api/expenses', expenseRoutes);

app.listen(5000, () => console.log('Server is running on port 5000'));

// Expense Model Schema
tconst mongoose = require('mongoose');
const ExpenseSchema = new mongoose.Schema({
    title: { type: String, required: true },
    amount: { type: Number, required: true },
    category: { type: String, required: true },
    date: { type: Date, default: Date.now },
});
module.exports = mongoose.model('Expense', ExpenseSchema);

// Routes for Managing Expenses
const express = require('express');
const Expense = require('../models/Expense');
const router = express.Router();

router.post('/', async (req, res) => {
    try {
        const { title, amount, category } = req.body;
        const newExpense = new Expense({ title, amount, category });
        await newExpense.save();
        res.status(201).json(newExpense);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

router.get('/', async (req, res) => {
    try {
        const expenses = await Expense.find();
        res.json(expenses);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

module.exports = router;

// Frontend React Component
import React, { useEffect, useState } from 'react';
import axios from 'axios';
import './App.css';

function App() {
    const [expenses, setExpenses] = useState([]);
    const [form, setForm] = useState({ title: '', amount: '', category: '' });

    useEffect(() => {
        axios.get('http://localhost:5000/api/expenses')
            .then(response => setExpenses(response.data))
            .catch(error => console.error('Error fetching expenses:', error));
    }, []);

    const handleChange = (e) => {
        setForm({ ...form, [e.target.name]: e.target.value });
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            const response = await axios.post('http://localhost:5000/api/expenses', form);
            setExpenses([...expenses, response.data]);
            setForm({ title: '', amount: '', category: '' });
        } catch (error) {
            console.error('Error adding expense:', error);
        }
    };

    return (
        <div className="container">
            <h1>Personal Expense Tracker</h1>
            <form onSubmit={handleSubmit}>
                <input type="text" name="title" placeholder="Expense Title" value={form.title} onChange={handleChange} required />
                <input type="number" name="amount" placeholder="Amount" value={form.amount} onChange={handleChange} required />
                <input type="text" name="category" placeholder="Category" value={form.category} onChange={handleChange} required />
                <button type="submit">Add Expense</button>
            </form>
            <ul>
                {expenses.map(expense => (
                    <li key={expense._id}>💰 {expense.title} - ${expense.amount} ({expense.category})</li>
                ))}
            </ul>
        </div>
    );
}

export default App;


I've made the code look more natural and human-written by improving readability, adding meaningful comments, and making log messages more descriptive. Let me know if you need further refinements!

