// **server.js**
const express = require('express');
const path = require('path');
const fs = require('fs');
const bodyParser = require('body-parser');

const app = express();
const PORT = process.env.PORT || 3000;
const dataFile = path.join(__dirname, 'data', 'wishes.json');

// Middleware
to ensure data folder exists
if (!fs.existsSync(path.dirname(dataFile))) {
  fs.mkdirSync(path.dirname(dataFile), { recursive: true });
}

app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'public')));

// API: Get all wishes\app.get('/api/wishes', (req, res) => {
  fs.readFile(dataFile, 'utf8', (err, data) => {
    if (err) return res.json([]);
    try { res.json(JSON.parse(data)); }
    catch { res.json([]); }
  });
});

// API: Add a new wish
app.post('/api/wishes', (req, res) => {
  const { name, msg } = req.body;
  if (!name || !msg) return res.status(400).json({ error: 'Name and message required' });

  fs.readFile(dataFile, 'utf8', (err, data) => {
    let wishes = [];
    if (!err) {
      try { wishes = JSON.parse(data); } catch {}
    }
    wishes.push({ name, msg, time: new Date().toISOString() });
    fs.writeFile(dataFile, JSON.stringify(wishes, null, 2), () => {
      res.json(wishes);
    });
  });
});

app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));


/* Directory Structure:

proposal-site/
├─ data/
│  └─ wishes.json        # stores guestbook entries
├─ public/
│  ├─ index.html         # Home page
│  ├─ story.html         # Our Story
│  ├─ gallery.html       # Gallery
│  ├─ proposal.html      # Proposal
│  ├─ countdown.html     # Countdown
│  ├─ css/
│  │  └─ styles.css      # shared styles
│  ├─ js/
│  │  └─ guestbook.js    # front-end calls to /api/wishes
│  └─ assets/            # images, video, fonts
├─ server.js             # this backend file
├─ package.json          # dependencies: express, body-parser
└─ README.md            # instructions

*/

// **public/js/guestbook.js**

document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('wishForm');
  const list = document.getElementById('wishesList');

  // Load initial wishes
  fetch('/api/wishes')
    .then(res => res.json())
    .then(wishes => {
      wishes.forEach(addToList);
    });

  form.addEventListener('submit', e => {
    e.preventDefault();
    const name = form.elements['name'].value;
    const msg = form.elements['message'].value;

    fetch('/api/wishes', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name, msg })
    })
      .then(res => res.json())
      .then(wishes => {
        addToList(wishes[wishes.length - 1]);
        form.reset();
      });
  });

  function addToList({ name, msg, time }) {
    const li = document.createElement('li');
    const date = new Date(time).toLocaleString();
    li.textContent = `${date} — ${name}: ${msg}`;
    list.prepend(li);
  }
});
