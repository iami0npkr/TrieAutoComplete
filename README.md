# MERN Stack Autocomplete Application Using Trie Data Structure

## **Introduction**

This project implements an autocomplete feature using a Trie data structure within a MERN stack application. The frontend is built with React, styled using Bootstrap, and the backend is powered by Express and MongoDB.

## **Project Structure**

The project is organized into two main directories: `client` for the frontend and `server` for the backend.

```
mern-autocomplete/
│
├── server/
│   ├── models/
│   │   └── Word.js
│   ├── routes/
│   │   └── autocomplete.js
│   ├── trie.js
│   ├── server.js
│   └── .env
│
└── client/
    ├── public/
    │   └── index.html
    ├── src/
    │   ├── components/
    │   │   └── Autocomplete.js
    │   ├── App.js
    │   ├── index.js
    │   └── App.css
    └── package.json
```

---

## **Backend Setup**

### **1. Initialize the Backend**

Create a directory for the server and initialize a Node.js project.

```bash
mkdir server
cd server
npm init -y
npm install express mongoose cors dotenv
```

### **2. Create a `.env` File**

The `.env` file stores environment variables. It should contain your MongoDB connection string and the port number.

```env
MONGO_URI=your_mongodb_connection_string
PORT=5000
```

### **3. `server.js`**

This file sets up the Express server and connects to MongoDB.

```javascript
// server.js
require('dotenv').config();
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const autocompleteRoute = require('./routes/autocomplete');

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.log('MongoDB connection error:', err));

// Routes
app.use('/api', autocompleteRoute);

// Start the server
const port = process.env.PORT || 5000;
app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

### **4. `models/Word.js`**

Defines the Mongoose model for storing words.

```javascript
// models/Word.js
const mongoose = require('mongoose');

const wordSchema = new mongoose.Schema({
  text: {
    type: String,
    required: true,
  },
});

module.exports = mongoose.model('Word', wordSchema);
```

### **5. `trie.js`**

Implements the Trie data structure used for the autocomplete functionality.

```javascript
// trie.js
class TrieNode {
  constructor() {
    this.children = {};
    this.isEndOfWord = false;
  }
}

class Trie {
  constructor() {
    this.root = new TrieNode();
  }

  insert(word) {
    let node = this.root;
    for (let char of word) {
      if (!node.children[char]) {
        node.children[char] = new TrieNode();
      }
      node = node.children[char];
    }
    node.isEndOfWord = true;
  }

  autocomplete(prefix) {
    let node = this.root;
    for (let char of prefix) {
      if (!node.children[char]) return [];
      node = node.children[char];
    }
    return this._findAllWords(node, prefix);
  }

  _findAllWords(node, prefix) {
    let words = [];
    if (node.isEndOfWord) words.push(prefix);
    for (let char in node.children) {
      words = words.concat(this._findAllWords(node.children[char], prefix + char));
    }
    return words;
  }
}

module.exports = Trie;
```

### **6. `routes/autocomplete.js`**

Defines the route for fetching autocomplete suggestions and adding new words.

```javascript
// routes/autocomplete.js
const express = require('express');
const Trie = require('../trie');
const Word = require('../models/Word');

const router = express.Router();
const trie = new Trie();

// Build the Trie with words from the database
Word.find().then(words => {
  words.forEach(word => trie.insert(word.text));
});

// API route for autocomplete
router.get('/search', (req, res) => {
  const { q } = req.query;
  const suggestions = trie.autocomplete(q);
  res.json(suggestions);
});

// API route to add new words
router.post('/add-word', async (req, res) => {
  const { text } = req.body;
  if (!text) return res.status(400).json({ error: 'Word is required' });

  try {
    const newWord = new Word({ text });
    await newWord.save();
    trie.insert(text); // Add word to the Trie
    res.status(201).json({ message: 'Word added successfully' });
  } catch (error) {
    res.status(500).json({ error: 'Failed to add word' });
  }
});

module.exports = router;
```

---

## **Frontend Setup**

### **1. Create a React App**

Navigate back to the root directory and create a React app.

```bash
npx create-react-app client
cd client
npm install bootstrap axios react-bootstrap
```

### **2. Modify `index.js`**

Import Bootstrap CSS in `src/index.js`.

```javascript
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import 'bootstrap/dist/css/bootstrap.min.css';
import App from './App';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

### **3. `App.js`**

The main component that renders the `Autocomplete` component.

```javascript
// src/App.js
import React from 'react';
import Autocomplete from './components/Autocomplete';
import './App.css';

function App() {
  return (
    <div className="App">
      <h1 className="text-center my-4">Autocomplete Feature Using Trie</h1>
      <Autocomplete />
    </div>
  );
}

export default App;
```

### **4. `components/Autocomplete.js`**

Handles the autocomplete logic and UI.

```javascript
// src/components/Autocomplete.js
import React, { useState } from 'react';
import axios from 'axios';
import { InputGroup, FormControl, ListGroup, Button, Form } from 'react-bootstrap';

function Autocomplete() {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const [newWord, setNewWord] = useState('');

  const fetchSuggestions = async (q) => {
    if (q.length === 0) {
      setSuggestions([]);
      return;
    }
    const response = await axios.get(`http://localhost:5000/api/search?q=${q}`);
    setSuggestions(response.data);
  };

  const onChangeHandler = (e) => {
    const value = e.target.value;
    setQuery(value);
    fetchSuggestions(value);
  };

  const onNewWordChangeHandler = (e) => {
    setNewWord(e.target.value);
  };

  const addWordHandler = async (e) => {
    e.preventDefault();
    if (newWord.trim()) {
      try {
        const response = await axios.post('http://localhost:5000/api/add-word', { text: newWord });
        alert('Word added successfully');
        setNewWord('');
      } catch (error) {
        console.error(error.response.data);  // Log the error response
        alert('Failed to add word');
      }
    }
  };

  return (
    <div className="autocomplete-container">
      <h2>Add a New Word</h2>
      <Form onSubmit={addWordHandler}>
        <InputGroup className="mb-3">
          <FormControl
            placeholder="Enter a word"
            value={newWord}
            onChange={onNewWordChangeHandler}
            aria-label="New Word"
          />
          <Button type="submit">Add Word</Button>
        </InputGroup>
      </Form>

      <h2>Search Words</h2>
      <InputGroup>
        <FormControl
          placeholder="Search..."
          value={query}
          onChange={onChangeHandler}
          aria-label="Search"
        />
      </InputGroup>

      {suggestions.length > 0 && (
        <ListGroup className="suggestions-list">
          {suggestions.map((suggestion, index) => (
            <ListGroup.Item key={index}>{suggestion}</ListGroup.Item>
          ))}
        </ListGroup>
      )}
    </div>
  );
}

export default Autocomplete;
```

### **5. `App.css`**

CSS to style the autocomplete component.

```css
/* src/App.css */
.autocomplete-container {
  position: relative;
  max-width: 400px;
  margin: auto;
  margin-top: 20px;
}

.suggestions-list {
  position: absolute;
  width: 100%;
  z-index: 1000;
}

.suggestions-list .list-group-item {
  cursor: pointer;
}

h2 {
  margin-bottom: 10px;
}

.mb-3 {
  margin-bottom

: 1rem !important;
}
```

---

## **Running the Project**

### **1. Start the Backend**

Navigate to the `server` directory and start the Express server.

```bash
cd server
node server.js
```

### **2. Start the Frontend**

Navigate to the `client` directory and start the React app.

```bash
cd client
npm start
```

### **3. Access the Application**

Open your browser and navigate to `http://localhost:3000`.

---

## **Conclusion**

This project demonstrates how to implement a Trie-based autocomplete feature in a MERN stack application. It includes the ability to add new words to the Trie and search for suggestions based on user input.

---
 
