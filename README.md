# webhook-repo
app.py
from flask import Flask, request, jsonify
from pymongo import MongoClient
from datetime import datetime
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

# MongoDB setup
client = MongoClient("mongodb://localhost:27017/")
db = client["webhookDB"]
collection = db["github_events"]

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.json
    record = {}

    if 'pusher' in data:
        # PUSH event
        record = {
            "author": data["pusher"]["name"],
            "to_branch": data["ref"].split("/")[-1],
            "timestamp": datetime.utcnow().strftime('%d %B %Y - %I:%M %p UTC'),
            "action": "push"
        }

    elif 'pull_request' in data:
        pr = data["pull_request"]
        action_type = data["action"]
        record = {
            "author": pr["user"]["login"],
            "from_branch": pr["head"]["ref"],
            "to_branch": pr["base"]["ref"],
            "timestamp": datetime.utcnow().strftime('%d %B %Y - %I:%M %p UTC'),
            "action": "merge" if pr.get("merged", False) else "pull_request"
        }

    if record:
        collection.insert_one(record)
        return jsonify({"status": "saved"}), 200
    else:
        return jsonify({"status": "ignored"}), 204

@app.route('/events', methods=['GET'])
def get_events():
    events = list(collection.find({}, {'_id': 0}))
    return jsonify(events)

if __name__ == '__main__':
    app.run(debug=True, port=5000)

    index.html
    HTML
    <!DOCTYPE html>
<html>
<head>
  <title>GitHub Activity Feed</title>
  <style>
    body { font-family: sans-serif; background: #f4f4f4; padding: 20px; }
    h2 { margin-bottom: 10px; }
    .event {
      background: white;
      border-radius: 8px;
      padding: 15px;
      margin-bottom: 10px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
    }
  </style>
</head>
<body>
  <h2>Latest GitHub Events</h2>
  <div id="events"></div>

  <script>
    async function fetchEvents() {
      const response = await fetch("http://localhost:5000/events");
      const data = await response.json();
      const container = document.getElementById('events');
      container.innerHTML = '';

      data.reverse().forEach(event => {
        let text = '';
        if (event.action === "push") {
          text = `${event.author} pushed to ${event.to_branch} on ${event.timestamp}`;
        } else if (event.action === "pull_request") {
          text = `${event.author} submitted a pull request from ${event.from_branch} to ${event.to_branch} on ${event.timestamp}`;
        } else if (event.action === "merge") {
          text = `${event.author} merged branch ${event.from_branch} to ${event.to_branch} on ${event.timestamp}`;
        }

        const div = document.createElement('div');
        div.className = 'event';
        div.innerText = text;
        container.appendChild(div);
      });
    }

    fetchEvents();
    setInterval(fetchEvents, 15000); // every 15 seconds
  </script>
</body>
</html>

README.md
Markdown
# 🌐 webhook-repo

This repo receives GitHub webhook data from `action-repo` and saves it to MongoDB.

## Features
- Built using Flask
- Saves push, pull request, and merge events
- Frontend auto-refreshes every 15s

## How to Run

1. Install dependencies:
```bash
pip install flask pymongo flask-cors

Bash
python app.py
