ဒီ script တွေကို မိမိ project structure အရ မှန်ကန်တဲ့ folder အတွင်းမှာ ထည့်ရန်လိုအပ်ပါသည်။ အောက်မှာ project structure နှင့် script တွေကို ဘယ်နေရာမှာ ထည့်ရမယ်ဆိုတာကို အသေးစိတ်ရှင်းပြပေးပါမယ်။

Project Structure

my-social-network/
│
├── backend/               # Backend (Node.js, Express, Database)
│   ├── app.js             # Main server file (Backend code)
│   ├── models/            # MongoDB models (User, Chat)
│   │   └── user.js        # User model
│   │   └── chat.js        # Chat model
│   └── routes/            # Routes for APIs (Register, Login, etc.)
│       └── authRoutes.js  # Authentication Routes
│       └── chatRoutes.js  # Chat Routes (Real-time chat)
│
├── frontend/              # Frontend (HTML, CSS, JavaScript)
│   ├── index.html         # Homepage HTML template
│   ├── style.css          # Styles for the website
│   └── app.js             # Frontend JS (Real-time chat, Login)
│
├── config/                # Configuration files (Crypto wallet, Multi-language)
│   ├── crypto.js          # Crypto wallet setup
│   ├── language.js        # Multi-language configuration
│   └── serverConfig.js    # Server configurations
│
├── database/              # MongoDB configuration and setup scripts
│   └── init.js            # Initialize MongoDB connection
│
└── assets/                # Images, Emojis, GIFs
    ├── emojis/            # Emoji images
    └── gifs/              # GIF images

1. Frontend Setup (HTML, CSS, JS)

frontend/index.html

This is the main HTML file. The structure of the website will go here.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Social Network</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <h1>Welcome to My Social Network</h1>
    </header>
    <section id="chatSection">
        <!-- Chat functionality will go here -->
    </section>
    <footer>
        <p>© 2024 My Social Network</p>
    </footer>
    <script src="app.js"></script>
</body>
</html>

frontend/style.css

Style your website with CSS here.

body {
    font-family: Arial, sans-serif;
    background-color: #f1f1f1;
    margin: 0;
    padding: 0;
}

header {
    background-color: #000;
    color: white;
    padding: 15px;
    text-align: center;
}

footer {
    background-color: #000;
    color: white;
    padding: 10px;
    text-align: center;
}

frontend/app.js

JavaScript file for frontend functionality like chat and user interactions.

document.getElementById('chatSection').innerHTML = "<p>Welcome to the chat room!</p>";

2. Backend Setup (Node.js & Express)

backend/app.js

The main backend file that starts the server and handles routes.

const express = require('express');
const socket = require('socket.io');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const app = express();
const port = 3000;

// Set up express app
app.use(express.json());
app.use(express.static('../frontend'));

// Database connection
mongoose.connect('mongodb://localhost/social-network', { useNewUrlParser: true, useUnifiedTopology: true });

// Chat route
app.post('/api/chat', (req, res) => {
    res.send({ message: 'Chat functionality is working' });
});

// Real-time chat setup with socket.io
const server = app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});

// Handle WebSocket connections for real-time chat
const io = socket(server);
io.on('connection', (socket) => {
    console.log('User connected');
    socket.on('disconnect', () => {
        console.log('User disconnected');
    });
});

backend/models/user.js

The MongoDB schema for the user.

const mongoose = require('mongoose');

// User model
const userSchema = new mongoose.Schema({
    username: { type: String, required: true },
    email: { type: String, required: true },
    password: { type: String, required: true },
});

const User = mongoose.model('User', userSchema);

module.exports = User;

backend/models/chat.js

The MongoDB schema for the chat messages.

const mongoose = require('mongoose');

// Chat model
const chatSchema = new mongoose.Schema({
    sender: { type: String, required: true },
    message: { type: String, required: true },
    timestamp: { type: Date, default: Date.now },
});

const Chat = mongoose.model('Chat', chatSchema);

module.exports = Chat;

3. Database Setup

database/init.js

Set up MongoDB connection.

const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/social-network', { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('Database connected'))
    .catch(err => console.log('Database connection error: ', err));

4. Authentication (JWT Token)

In the backend/app.js, add routes for user registration and login using JWT:

backend/routes/authRoutes.js

For user registration and login:

const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/user');
const router = express.Router();

// Registration
router.post('/register', async (req, res) => {
    const { username, email, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({ username, email, password: hashedPassword });
    await user.save();
    res.status(201).send({ message: 'User registered' });
});

// Login
router.post('/login', async (req, res) => {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    if (user && await bcrypt.compare(password, user.password)) {
        const token = jwt.sign({ userId: user._id }, 'secretKey', { expiresIn: '1h' });
        res.status(200).send({ token });
    } else {
        res.status(400).send({ message: 'Invalid credentials' });
    }
});

module.exports = router;

5. Crypto Wallet (MetaMask Example)

frontend/app.js

Integrate a crypto wallet (e.g., MetaMask) into your website.

if (window.ethereum) {
    ethereum.request({ method: 'eth_requestAccounts' }).then(accounts => {
        console.log('Connected Account:', accounts[0]);
    });
}

6. Socket.IO Real-Time Chat

In the frontend/app.js and backend/app.js, integrate real-time chat with Socket.IO.

frontend/app.js

const socket = io.connect('http://localhost:3000');

socket.emit('send_message', { message: 'Hello' });

socket.on('receive_message', function(data) {
    console.log(data.message);
});

backend/app.js

const io = socket(server);
io.on('connection', (socket) => {
    console.log('User connected');
    socket.on('send_message', (data) => {
        io.emit('receive_message', { message: data.message });
    });
});

7. Language & Emoji/GIF Setup

You can implement language support by using libraries like i18next and integrate emoji/GIF features by using external APIs.

8. Deployment

For deployment, you can use Docker or cloud services like Heroku, AWS, or DigitalOcean.


---

Final Notes

All the frontend files (HTML, CSS, JS) will go inside the frontend/ folder.

Backend code (API routes, models) should go inside the backend/ folder.

The database initialization script is in the database/ folder.


