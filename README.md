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
 
