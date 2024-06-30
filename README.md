npx react-native init LoversBed
cd LoversBed

import React, { useState } from 'react';
import { SafeAreaView, TextInput, Button, Text, FlatList, View, StyleSheet } from 'react-native';

const App = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [posts, setPosts] = useState([]);
  
  const handleLogin = async () => {
    // Call the login API here
    // For now, we'll just set some dummy posts
    setPosts([
      { id: '1', content: 'Hello from LoversBed!' },
      { id: '2', content: 'Enjoy your stay.' }
    ]);
  };

  return (
    <SafeAreaView style={styles.container}>
      <TextInput 
        style={styles.input}
        placeholder="Username"
        value={username}
        onChangeText={setUsername}
      />
      <TextInput 
        style={styles.input}
        placeholder="Password"
        secureTextEntry
        value={password}
        onChangeText={setPassword}
      />
      <Button title="Login" onPress={handleLogin} />
      <FlatList 
        data={posts}
        keyExtractor={item => item.id}
        renderItem={({ item }) => (
          <View style={styles.post}>
            <Text>{item.content}</Text>
          </View>
        )}
      />
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    padding: 16
  },
  input: {
    height: 40,
    borderColor: 'gray',
    borderWidth: 1,
    marginBottom: 12,
    padding: 8
  },
  post: {
    padding: 16,
    borderBottomColor: 'gray',
    borderBottomWidth: 1
  }
});

export default App;

mkdir LoversBedServer
cd LoversBedServer
npm init -y
npm install express mongoose bcryptjs jsonwebtoken

// index.js
const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const app = express();
app.use(express.json());

const mongoURI = 'mongodb://localhost:27017/loversbed';
mongoose.connect(mongoURI, { useNewUrlParser: true, useUnifiedTopology: true });

const userSchema = new mongoose.Schema({
  username: String,
  password: String
});

const User = mongoose.model('User', userSchema);

app.post('/register', async (req, res) => {
  const { username, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  const newUser = new User({ username, password: hashedPassword });
  await newUser.save();
  res.status(201).send('User registered');
});

app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username });
  if (!user || !(await bcrypt.compare(password, user.password))) {
    return res.status(401).send('Invalid credentials');
  }
  const token = jwt.sign({ id: user._id }, 'secretKey', { expiresIn: '1h' });
  res.json({ token });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});