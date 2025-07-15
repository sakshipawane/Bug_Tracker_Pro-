# Bug_Tracker_Pro-

const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const dotenv = require('dotenv');
const ticketRoutes = require('./routes/tickets');
const userRoutes = require('./routes/users');

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect(process.env.MONGO_URI)
  .then(() => console.log('MongoDB connected'))
  .catch((err) => console.error(err));

app.use('/api/tickets', ticketRoutes);
app.use('/api/users', userRoutes);

app.listen(5000, () => console.log('Server running on port 5000'));


const mongoose = require('mongoose');

const TicketSchema = new mongoose.Schema({
  title: String,
  description: String,
  status: { type: String, enum: ['Open', 'In Progress', 'Closed'], default: 'Open' },
  createdBy: String,
  assignedTo: String,
  createdAt: { type: Date, default: Date.now },
});

module.exports = mongoose.model('Ticket', TicketSchema);


const express = require('express');
const Ticket = require('../models/Ticket');
const router = express.Router();

router.get('/', async (req, res) => {
  const tickets = await Ticket.find();
  res.json(tickets);
});

router.post('/', async (req, res) => {
  const ticket = new Ticket(req.body);
  await ticket.save();
  res.status(201).json(ticket);
});

router.put('/:id', async (req, res) => {
  const updated = await Ticket.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.json(updated);
});

router.delete('/:id', async (req, res) => {
  await Ticket.findByIdAndDelete(req.params.id);
  res.json({ message: 'Ticket deleted' });
});

module.exports = router;

import React, { useEffect, useState } from 'react';
import axios from 'axios';

function App() {
  const [tickets, setTickets] = useState([]);
  const [form, setForm] = useState({ title: '', description: '', assignedTo: '' });

  const fetchTickets = async () => {
    const res = await axios.get('http://localhost:5000/api/tickets');
    setTickets(res.data);
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await axios.post('http://localhost:5000/api/tickets', form);
    setForm({ title: '', description: '', assignedTo: '' });
    fetchTickets();
  };

  const updateStatus = async (id, newStatus) => {
    await axios.put(`http://localhost:5000/api/tickets/${id}`, { status: newStatus });
    fetchTickets();
  };

  useEffect(() => {
    fetchTickets();
  }, []);

  return (
    <div className="p-6 max-w-2xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">🐞 BugTracker Pro</h1>

      <form onSubmit={handleSubmit} className="mb-6">
        <input
          className="border p-2 mr-2"
          placeholder="Title"
          value={form.title}
          onChange={(e) => setForm({ ...form, title: e.target.value })}
        />
        <input
          className="border p-2 mr-2"
          placeholder="Description"
          value={form.description}
          onChange={(e) => setForm({ ...form, description: e.target.value })}
        />
        <input
          className="border p-2 mr-2"
          placeholder="Assigned To"
          value={form.assignedTo}
          onChange={(e) => setForm({ ...form, assignedTo: e.target.value })}
        />
        <button className="bg-blue-600 text-white p-2">Create Ticket</button>
      </form>

      <ul>
        {tickets.map((ticket) => (
          <li key={ticket._id} className="border p-3 mb-2 rounded">
            <h3 className="font-semibold">{ticket.title}</h3>
            <p>{ticket.description}</p>
            <p>Assigned to: {ticket.assignedTo}</p>
            <p>Status: {ticket.status}</p>
            <div className="mt-2">
              <button
                className="bg-yellow-500 text-white px-2 mr-2"
                onClick={() => updateStatus(ticket._id, 'In Progress')}
              >In Progress</button>
              <button
                className="bg-green-600 text-white px-2"
                onClick={() => updateStatus(ticket._id, 'Closed')}
              >Close</button>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
