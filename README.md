# devops_lab
Form Validation (React)
import React, { useState } from "react";

function App() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [error, setError] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();
    if (name.trim() === "" || email.trim() === "") {
      setError("All fields are required!");
    } else if (!email.includes("@")) {
      setError("Enter a valid email!");
    } else {
      setError("");
      alert(`Welcome, ${name}!`);
    }
  };

  return (
    <div style={{ textAlign: "center", marginTop: "50px" }}>
      <h2>Form Validation Example</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Enter name"
          value={name}
          onChange={(e) => setName(e.target.value)}
        /><br /><br />
        <input
          type="email"
          placeholder="Enter email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        /><br /><br />
        <button type="submit">Submit</button>
      </form>
      <p style={{ color: "red" }}>{error}</p>
    </div>
  );
}

export default App;

BMI Calculator (React)
import React, { useState } from "react";

function App() {
  const [weight, setWeight] = useState("");
  const [height, setHeight] = useState("");
  const [bmi, setBmi] = useState(null);

  const calculateBMI = () => {
    if (weight && height) {
      const h = height / 100;
      const result = (weight / (h * h)).toFixed(2);
      setBmi(result);
    }
  };

  return (
    <div style={{ textAlign: "center", marginTop: "50px" }}>
      <h2>BMI Calculator</h2>
      <input
        type="number"
        placeholder="Weight (kg)"
        value={weight}
        onChange={(e) => setWeight(e.target.value)}
      /><br /><br />
      <input
        type="number"
        placeholder="Height (cm)"
        value={height}
        onChange={(e) => setHeight(e.target.value)}
      /><br /><br />
      <button onClick={calculateBMI}>Calculate</button>
      {bmi && <h3>Your BMI: {bmi}</h3>}
    </div>
  );
}

export default App;

 Simple CRUD Application (Local State Only)
import React, { useState } from "react";

function App() {
  const [item, setItem] = useState("");
  const [list, setList] = useState([]);

  const addItem = () => {
    if (item.trim() === "") return;
    setList([...list, item]);
    setItem("");
  };

  const deleteItem = (index) => {
    const updated = list.filter((_, i) => i !== index);
    setList(updated);
  };

  return (
    <div style={{ textAlign: "center", marginTop: "50px" }}>
      <h2>Simple CRUD App</h2>
      <input
        type="text"
        placeholder="Enter item"
        value={item}
        onChange={(e) => setItem(e.target.value)}
      />
      <button onClick={addItem}>Add</button>

      <ul>
        {list.map((val, i) => (
          <li key={i}>
            {val} <button onClick={() => deleteItem(i)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;

Fetching Information from an API

Using the JSONPlaceholder fake API (no key required):

import React, { useEffect, useState } from "react";

function App() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/users")
      .then((res) => res.json())
      .then((data) => setUsers(data));
  }, []);

  return (
    <div style={{ textAlign: "center", marginTop: "50px" }}>
      <h2>Fetch API Example</h2>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name} — {user.email}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;


# ---------- Stage 1: Build React App ----------
FROM node:18-alpine AS build

# Set working directory inside container
WORKDIR /app

# Copy package files first (for caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the app
COPY . .

# Build the React app for production
RUN npm run build

# ---------- Stage 2: Serve the App ----------
FROM nginx:alpine

# Copy build output from previous stage to Nginx HTML folder
COPY --from=build /app/dist /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]



pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Monishapmj/practice.git'
            }
        }

        stage('Use Node') {
            steps {
                bat '''
                    node -v
                    npm -v
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('reactapp') {   // 👈 change this to your actual folder name
                    bat 'npm install'
                }
            }
        }

        stage('Build Project') {
            steps {
                dir('reactapp') {   // 👈 same folder name
                    bat 'npm run build'
                }
            }
        }

        stage('Archive Build Output') {
            steps {
                archiveArtifacts artifacts: 'reactapp/dist/**', fingerprint: true
            }
        }
    }

    post {
        success {
            echo '✅ Build Successful'
        }
        failure {
            echo '❌ Build Failed'
        }
    }
}

