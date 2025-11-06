# App_react
app.js- 
const express = require ("express");
const cors = require ("cors");
const mysql = require ("mysql2");

const app = express();
const port = 8080;

// middleware
app.use (cors());
app.use (express.json());

// sql connection
const db = mysql.createConnection ({
    host: "localhost",
    user: "root",
    database: "library",
    password: ""
});

db.connect ((err)=>{
    if (err){
        console.log ("db is not connected", err);
    }
    else {
        console.log ("DB is connected successfully");
    }
});

const getAll = "select * from books";
const add = "insert into books(title, author, year, genre, status) values(?,?,?,?,?)";
const update ="update books set title=?, author=?, year=?, genre=?, status=? where id = ?";
const deleteR = "delete from books where id =?";

// get all books
app.get("/books", (req, res) => {
  db.query(getAll, (err, results) => {
    if (err) res.status(500).send(err);
    else res.send(results);
  });
});

// CREATE NEW BOOK

app.post("/books", (req, res) => {
  const { title, author, year, genre, status } = req.body;
  db.query(
    add,
    [title, author, year, genre, status],
    (err) => {
      if (err) res.status(500).send(err);
      else res.send({ message: "Book added successfully" });
    }
  );
});


// ✏️ Update book
app.put("/books/:id", (req, res) => {
  const { id } = req.params;
  const { title, author, year, genre, status } = req.body;
  db.query(
    update,
    [title, author, year, genre, status, id],
    (err) => {
      if (err) res.status(500).send(err);
      else res.send({ message: "Book updated successfully" });
    }
  );
});


// ❌ Delete book
app.delete("/books/:id", (req, res) => {
  const { id } = req.params;
  db.query("DELETE FROM books WHERE id=?", [id], (err) => {
    if (err) res.status(500).send(err);
    else res.send({ message: "Book deleted successfully" });
  });
});

app.listen (port, ()=>{
  console.log (`App is listening on port ${port}`);
})
// check update book json message data formate

---------------------------------------------------------------------
frontend-
App.jsx- 
import React, { useEffect, useState } from "react";
import { getBooks, addBook, updateBook, deleteBook } from "./services/api";
import BookForm from "./components/BookForm";
import BookTable from "./components/BookTable";

export default function App() {
  const [books, setBooks] = useState([]);
  const [form, setForm] = useState({ title: "", author: "", year: "", genre: "", status: "" });
  const [editId, setEditId] = useState(null);

  const fetchBooks = async () => {
    const res = await getBooks();
    setBooks(res.data);
  };

  useEffect(() => { fetchBooks(); }, []);

  const handleChange = (e) => setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = async (e) => {
    e.preventDefault();
    editId ? await updateBook(editId, form) : await addBook(form);
    setForm({ title: "", author: "", year: "", genre: "", status: "" });
    setEditId(null);
    fetchBooks();
  };

  const handleEdit = (book) => {
    setEditId(book.id);
    setForm(book);
  };

  const handleDelete = async (id) => {
    await deleteBook(id);
    fetchBooks();
  };

  // return (
  //  <div style={{ width: "80%", margin: "auto", textAlign: "center" }}>
  //    <h1>📚 Library Management App</h1>
  //    <BookForm form={form} onChange={handleChange} onSubmit={handleSubmit} editId={editId} />
 //     <BookTable books={books} onEdit={handleEdit} onDelete={handleDelete} />
 //   </div>
  );
}
-----------------------------------------------------------------------
src- components-
// BookFrom.jsx
import React from "react";

export default function BookForm({ form, onChange, onSubmit, editId }) {
  return (
    <form onSubmit={onSubmit}>
      {["title", "author", "year", "genre", "status"].map((f) => (
        <input
          key={f}
          name={f}
          value={form[f]}
          onChange={onChange}
          placeholder={f.charAt(0).toUpperCase() + f.slice(1)}
          style={{ margin: "5px", padding: "5px" }}
          required
        />
      ))}
      <button type="submit">{editId ? "Update" : "Add"} Book</button>
    </form>
  );
}
-----------------------------------------------------------------
// BookTable.jsx
import React from "react";

export default function BookTable({ books, onEdit, onDelete }) {
  return (
    <table border="1" style={{ width: "100%", marginTop: "20px" }}>
      <thead>
        <tr>
          <th>ID</th>
          <th>Title</th>
          <th>Author</th>
          <th>Year</th>
          <th>Genre</th>
          <th>Status</th>
          <th>Actions</th>
        </tr>
      </thead>

      <tbody>
        {books.map((b) => (
          <tr key={b.id}>
            <td>{b.id}</td>
            <td>{b.title}</td>
            <td>{b.author}</td>
            <td>{b.year}</td>
            <td>{b.genre}</td>
            <td>{b.status}</td>
            <td>
              <button onClick={() => onEdit(b)}>Edit</button>
              <button onClick={() => onDelete(b.id)}>Delete</button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
------------------------------------------------------------------------
services- apo.js
import axios from "axios";

const API = "http://localhost:8080/books";

// Get all books
export const getBooks = async () => axios.get(API);

// Add new book
export const addBook = async (book) => axios.post(API, book);

// Update book
export const updateBook = async (id, book) => axios.put(`${API}/${id}`, book);

// Delete book
export const deleteBook = async (id) => axios.delete(`${API}/${id}`);

