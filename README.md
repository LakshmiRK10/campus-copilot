
// Campus Copilot Backend (Without Firebase)

const express = require("express");
const cors = require("cors");
const bodyParser = require("body-parser");
const fs = require("fs");
const path = require("path");

const app = express();
app.use(cors());
app.use(bodyParser.json());

const DATA_FILE = path.join(__dirname, "events.json");

let db = { users: {} };
if (fs.existsSync(DATA_FILE)) {
  db = JSON.parse(fs.readFileSync(DATA_FILE));
}

function saveDB() {
  fs.writeFileSync(DATA_FILE, JSON.stringify(db, null, 2));
}

app.post("/addevent", (req, res) => {
  const { userId, title, date, time } = req.body;
  if (!userId || !title || !date || !time) {
    return res.status(400).json({ error: "Missing fields" });
  }

  if (!db.users[userId]) db.users[userId] = [];

  const event = {
    id: Date.now().toString(),
    title,
    date,
    time,
    reminderSent: false
  };

  db.users[userId].push(event);
  saveDB();

  res.status(200).json({ message: "Event added", event });
});

app.get("/getevents/:userId", (req, res) => {
  const userId = req.params.userId;
  const events = db.users[userId] || [];
  res.status(200).json(events);
});

app.delete("/deleteevent/:userId/:eventId", (req, res) => {
  const { userId, eventId } = req.params;

  if (!db.users[userId]) {
    return res.status(404).json({ error: "User not found" });
  }

  db.users[userId] = db.users[userId].filter(event => event.id !== eventId);
  saveDB();

  res.status(200).json({ message: "Event deleted" });
});

app.use(express.static(path.join(__dirname, "frontend")));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Campus Copilot backend running on port ${PORT}`);
});

