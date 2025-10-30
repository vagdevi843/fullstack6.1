# fullstack6.1
// server.js
const express = require("express");
const jwt = require("jsonwebtoken");
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());

// ======= 1️⃣ LOGGING MIDDLEWARE =======
app.use((req, res, next) => {
  const time = new Date().toISOString();
  console.log(`[${time}] ${req.method} ${req.url}`);
  next(); // move to the next middleware or route
});

// ======= 2️⃣ AUTHENTICATION MIDDLEWARE =======
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers["authorization"];
  const token = authHeader && authHeader.split(" ")[1]; // Format: "Bearer <token>"

  if (!token) {
    return res.status(401).json({ message: "Access denied! Token missing" });
  }

  jwt.verify(token, "SECRET_KEY_123", (err, user) => {
    if (err) {
      return res.status(403).json({ message: "Invalid or expired token" });
    }
    req.user = user; // store decoded user data in request
    next();
  });
};

// ======= 3️⃣ DUMMY USER DATABASE =======
const users = [
  { id: 1, username: "admin", password: "12345", role: "admin" },
  { id: 2, username: "user1", password: "12345", role: "user" },
];

// ======= 4️⃣ LOGIN ROUTE (Generate JWT Token) =======
app.post("/login", (req, res) => {
  const { username, password } = req.body;
  const found = users.find(
    (u) => u.username === username && u.password === password
  );

  if (!found) {
    return res.status(401).json({ message: "Invalid credentials" });
  }

  const token = jwt.sign(
    { id: found.id, username: found.username, role: found.role },
    "SECRET_KEY_123",
    { expiresIn: "1h" }
  );

  res.json({ message: "Login successful", token });
});

// ======= 5️⃣ PROTECTED ROUTE =======
app.get("/dashboard", authenticateToken, (req, res) => {
  res.json({
    message: `Welcome ${req.user.username}!`,
    role: req.user.role,
    time: new Date().toLocaleString(),
  });
});

// ======= 6️⃣ PUBLIC ROUTE =======
app.get("/", (req, res) => {
  res.send("🏦 Welcome to the Secure Banking API (Public Route)");
});

// ======= 7️⃣ START SERVER =======
const PORT = 5000;
app.listen(PORT, () =>
  console.log(`🚀 Server running on http://localhost:${PORT}`)
);
