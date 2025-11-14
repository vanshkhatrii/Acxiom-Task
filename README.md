import React, { useState, useContext, createContext } from "react";
import { BrowserRouter as Router, Route, Routes, Navigate, Link } from "react-router-dom";

// ================== Auth Context ==================
const AuthContext = createContext();

const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null); // {role: 'admin' | 'user'}
  const login = (username, password) => {
    if (username === "adm" && password === "adm") setUser({ role: "admin" });
    else if (username === "user" && password === "user") setUser({ role: "user" });
    else alert("Invalid credentials");
  };
  const logout = () => setUser(null);
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

const useAuth = () => useContext(AuthContext);

// ================== Mock Data ==================
const initialBooks = [
  { id: 1, name: "Science Book", author: "Author A", status: "Available" },
  { id: 2, name: "Economics Book", author: "Author B", status: "Issued" }
];
const initialMemberships = [
  { id: 101, name: "John Doe", startDate: "2025-01-01", endDate: "2025-07-01", status: "Active" }
];

// ================== Components ==================
const Navbar = () => {
  const { user, logout } = useAuth();
  return (
    <nav style={{ padding: "10px", background: "#eee" }}>
      {user && (
        <>
          <Link to="/">Home</Link> | <Link to="/reports">Reports</Link> |{" "}
          <Link to="/transactions">Transactions</Link>{" "}
          {user.role === "admin" && <>| <Link to="/maintenance">Maintenance</Link></>}
          | <button onClick={logout}>Logout</button>
        </>
      )}
    </nav>
  );
};

const Login = () => {
  const { login } = useAuth();
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  return (
    <div style={{ padding: "20px" }}>
      <h2>Library Management Login</h2>
      <input placeholder="User ID" value={username} onChange={(e) => setUsername(e.target.value)} />
      <br />
      <input
        type="password"
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <br />
      <button onClick={() => login(username, password)}>Login</button>
    </div>
  );
};

const Home = () => {
  const { user } = useAuth();
  return (
    <div style={{ padding: "20px" }}>
      <h2>{user.role === "admin" ? "Admin Home Page" : "User Home Page"}</h2>
      <p>Welcome! You have access to {user.role === "admin" ? "Maintenance, Reports, Transactions" : "Reports and Transactions"}.</p>
    </div>
  );
};

const Reports = () => (
  <div style={{ padding: "20px" }}>
    <h2>Reports</h2>
    <ul>
      <li>Master List of Books</li>
      <li>Master List of Movies</li>
      <li>Master List of Memberships</li>
      <li>Active Issues</li>
      <li>Overdue Returns</li>
      <li>Issue Requests</li>
    </ul>
  </div>
);

const Transactions = () => {
  const [bookName, setBookName] = useState("");
  const [issueDate, setIssueDate] = useState("");
  const [returnDate, setReturnDate] = useState("");
  const today = new Date().toISOString().split("T")[0];

  const handleIssue = () => {
    if (!bookName || !issueDate) {
      alert("Book name and Issue Date are mandatory");
      return;
    }
    if (new Date(issueDate) < new Date(today)) {
      alert("Issue date cannot be before today");
      return;
    }
    alert(`Book "${bookName}" issued successfully!`);
  };

  return (
    <div style={{ padding: "20px" }}>
      <h2>Transactions</h2>
      <h3>Issue Book</h3>
      <input placeholder="Book Name" value={bookName} onChange={(e) => setBookName(e.target.value)} />
      <br />
      <input type="date" value={issueDate} onChange={(e) => setIssueDate(e.target.value)} />
      <br />
      <input
        type="date"
        value={returnDate}
        onChange={(e) => setReturnDate(e.target.value)}
        placeholder="Return Date"
      />
      <br />
      <button onClick={handleIssue}>Confirm Issue</button>
    </div>
  );
};

const Maintenance = () => {
  const [memberships, setMemberships] = useState(initialMemberships);
  const [books, setBooks] = useState(initialBooks);
  const [newMember, setNewMember] = useState("");
  const [newBook, setNewBook] = useState("");

  const addMember = () => {
    if (!newMember) {
      alert("Member name required");
      return;
    }
    setMemberships([...memberships, { id: Date.now(), name: newMember, status: "Active" }]);
    setNewMember("");
  };

  const addBook = () => {
    if (!newBook) {
      alert("Book name required");
      return;
    }
    setBooks([...books, { id: Date.now(), name: newBook, author: "Unknown", status: "Available" }]);
    setNewBook("");
  };

  return (
    <div style={{ padding: "20px" }}>
      <h2>Maintenance (Admin Only)</h2>
      <h3>Add Membership</h3>
      <input placeholder="Member Name" value={newMember} onChange={(e) => setNewMember(e.target.value)} />
      <button onClick={addMember}>Add</button>
      <h3>Add Book</h3>
      <input placeholder="Book Name" value={newBook} onChange={(e) => setNewBook(e.target.value)} />
      <button onClick={addBook}>Add</button>
      <h4>Current Members:</h4>
      <ul>{memberships.map((m) => <li key={m.id}>{m.name}</li>)}</ul>
      <h4>Current Books:</h4>
      <ul>{books.map((b) => <li key={b.id}>{b.name} - {b.status}</li>)}</ul>
    </div>
  );
};

// ================== Protected Route ==================
const PrivateRoute = ({ children }) => {
  const { user } = useAuth();
  return user ? children : <Navigate to="/login" />;
};

// ================== App ==================
export default function App() {
  return (
    <AuthProvider>
      <Router>
        <Navbar />
        <Routes>
          <Route path="/login" element={<Login />} />
          <Route path="/" element={<PrivateRoute><Home /></PrivateRoute>} />
          <Route path="/reports" element={<PrivateRoute><Reports /></PrivateRoute>} />
          <Route path="/transactions" element={<PrivateRoute><Transactions /></PrivateRoute>} />
          <Route path="/maintenance" element={<PrivateRoute><Maintenance /></PrivateRoute>} />
        </Routes>
      </Router>
    </AuthProvider>
  );
}
