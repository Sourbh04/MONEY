# MONEY

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= title %></title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <header>
        <h1><a href="/">Money Tracker</a></h1>
        <nav>
            <a href="/">Home</a>
            <a href="/transaction/new">Add Transaction</a>
        </nav>
    </header>
    <div class="content">
        <%= content %>
    </div>
</body>
</html>



% include layout.ejs %>

<% content %>
<h2>Transactions</h2>
<div class="summary">
    <h3>Balance: $<%= balance %></h3>
    <p>Income: $<%= totalIncome %></p>
    <p>Expenses: $<%= totalExpense %></p>
</div>
<ul>
    <% transactions.forEach(function(transaction) { %>
        <li class="<%= transaction.type %>">
            <%= transaction.description %> - $<%= transaction.amount %> (<%= transaction.type %>)
        </li>
    <% }) %>
</ul>
<% endcontent %>

<% include layout.ejs %>

<% content %>
<h2>Add Transaction</h2>
<form action="/transaction" method="POST">
    <label for="description">Description:</label>
    <input type="text" id="description" name="description" required>

    <label for="amount">Amount:</label>
    <input type="number" id="amount" name="amount" required>

    <label for="type">Type:</label>
    <select id="type" name="type" required>
        <option value="income">Income</option>
        <option value="expense">Expense</option>
    </select>

    <input type="submit" value="Add Transaction">
</form>
<% endcontent %>

body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    margin: 0;
    padding: 0;
}

header {
    background-color: #333;
    color: #fff;
    padding: 10px 0;
    text-align: center;
}

header h1 a {
    color: #fff;
    text-decoration: none;
}

nav {
    margin-top: 10px;
}

nav a {
    color: #fff;
    text-decoration: none;
    margin: 0 15px;
}

.content {
    padding: 20px;
    max-width: 800px;
    margin: 20px auto;
    background-color: #fff;
    box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}

h2 {
    margin-top: 0;
}

input[type="text"],
input[type="number"],
select {
    width: 100%;
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

input[type="submit"] {
    background-color: #28a745;
    color: white;
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

input[type="submit"]:hover {
    background-color: #218838;
}

ul {
    list-style-type: none;
    padding: 0;
}

ul li {
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

li.income {
    background-color: #d4edda;
}

li.expense {
    background-color: #f8d7da;
}

.summary {
    margin-bottom: 20px;
    padding: 10px;
    background-color: #e9ecef;
    border-radius: 4px;
}

.summary h3 {
    margin-top: 0;
}
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const app = express();

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/moneyTrackerDB', { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('Connected to MongoDB'))
    .catch(err => console.error('Could not connect to MongoDB...', err));

// Middleware
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static('public'));
app.set('view engine', 'ejs');

// Define a schema and model for transactions
const transactionSchema = new mongoose.Schema({
    description: String,
    amount: Number,
    type: String,
});

const Transaction = mongoose.model('Transaction', transactionSchema);

// Routes
app.get('/', (req, res) => {
    Transaction.find({}, (err, transactions) => {
        if (err) {
            console.log(err);
        } else {
            let balance = 0;
            let totalIncome = 0;
            let totalExpense = 0;

            transactions.forEach((transaction) => {
                if (transaction.type === 'income') {
                    totalIncome += transaction.amount;
                } else if (transaction.type === 'expense') {
                    totalExpense += transaction.amount;
                }
            });

            balance = totalIncome - totalExpense;

            res.render('home', {
                title: 'Home',
                transactions: transactions,
                balance: balance,
                totalIncome: totalIncome,
                totalExpense: totalExpense
            });
        }
    });
});

app.get('/transaction/new', (req, res) => {
    res.render('new-transaction', { title: 'Add Transaction' });
});

app.post('/transaction', (req, res) => {
    const { description, amount, type } = req.body;

    const newTransaction = new Transaction({
        description,
        amount: parseFloat(amount),
        type,
    });

    newTransaction.save()
        .then(() => res.redirect('/'))
        .catch(err => res.status(400).send('Error: ' + err));
});

// Start the server
const port = 3000;
app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});
