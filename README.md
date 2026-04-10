const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const bcrypt = require("bcryptjs");

const app = express();
app.use(cors());
app.use(express.json());

// CONNECT DB
mongoose.connect("mongodb://127.0.0.1:27017/flaven");

// USER MODEL
const User = mongoose.model("User", {
  fullName: String,
  phone: { type: String, unique: true },
  password: String,
  balance: { type: Number, default: 1000 }, // bonus
  level: { type: String, default: "L0" },
  role: { type: String, default: "user" }, // superadmin, admin, user
  referralCode: String,
  referredBy: String,
});

// REGISTER
app.post("/register", async (req, res) => {
  const { fullName, phone, password } = req.body;

  const hashed = await bcrypt.hash(password, 10);

  const userCount = await User.countDocuments();

  const user = new User({
    fullName,
    phone,
    password: hashed,
    role: userCount === 0 ? "superadmin" : "user",
    referralCode: Math.random().toString(36).substring(2, 8),
  });

  await user.save();
  res.json({ message: "Registered successfully", user });
});

// LOGIN
app.post("/login", async (req, res) => {
  const { phone, password } = req.body;

  const user = await User.findOne({ phone });
  if (!user) return res.status(400).send("User not found");

  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(400).send("Wrong password");

  res.json(user);
});

// START SERVER
app.listen(5000, () => console.log("Server running on port 5000"));
