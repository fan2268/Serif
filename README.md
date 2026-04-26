# Serif
Mony exchange app
Login / Signup pages
Dashboard user: balance + transactions + manual conversion
Admin panel: update exchange rate + view all transactions
Payment page: Stripe integration + automatic ETB credit

1️⃣ Folder Structure (Frontend)
frontend/
├── src/
│   ├── App.js
│   ├── index.js
│   └── components/
│       ├── Login.js
│       ├── Signup.js
│       ├── Dashboard.js
│       ├── Admin.js
│       └── Payment.js


2️⃣ App.js
import React, { useState } from 'react';
import Login from './components/Login';
import Signup from './components/Signup';
import Dashboard from './components/Dashboard';
import Admin from './components/Admin';
import Payment from './components/Payment';

function App() {
  const [page, setPage] = useState('login');
  const [user, setUser] = useState(null);

  return (
    <div>
      {page === 'login' && <Login goSignup={() => setPage('signup')} onLogin={(u) => { setUser(u); setPage('dashboard'); }} />}
      {page === 'signup' && <Signup goLogin={() => setPage('login')} />}
      {page === 'dashboard' && user && (
        <>
          <Dashboard username={user} />
          <Payment username={user} />
          <button onClick={() => { setUser(null); setPage('login'); }}>Logout</button>
        </>
      )}
      {page === 'admin' && <Admin />}
    </div>
  );
}

export default App;


3️⃣ Login.js
import React, { useState } from 'react';
import axios from 'axios';

export default function Login({ goSignup, onLogin }) {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const login = async () => {
    try {
      const res = await axios.post('http://localhost:5000/login', { username, password });
      onLogin(res.data.username);
    } catch (err) { alert(err.response.data); }
  };

  return (
    <div>
      <h2>Login</h2>
      <input placeholder="Username" value={username} onChange={e=>setUsername(e.target.value)} />
      <input type="password" placeholder="Password" value={password} onChange={e=>setPassword(e.target.value)} />
      <button onClick={login}>Login</button>
      <p>Don't have an account? <span style={{color:'blue', cursor:'pointer'}} onClick={goSignup}>Signup</span></p>
    </div>
  );
}


4️⃣ Signup.js
import React, { useState } from 'react';
import axios from 'axios';

export default function Signup({ goLogin }) {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const signup = async () => {
    try {
      await axios.post('http://localhost:5000/signup', { username, password });
      alert('Signup successful! Please login.');
      goLogin();
    } catch (err) { alert(err.response.data); }
  };

  return (
    <div>
      <h2>Signup</h2>
      <input placeholder="Username" value={username} onChange={e=>setUsername(e.target.value)} />
      <input type="password" placeholder="Password" value={password} onChange={e=>setPassword(e.target.value)} />
      <button onClick={signup}>Signup</button>
      <p>Already have an account? <span style={{color:'blue', cursor:'pointer'}} onClick={goLogin}>Login</span></p>
    </div>
  );
}


5️⃣ Dashboard.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

export default function Dashboard({ username }) {
  const [rate, setRate] = useState(0);
  const [amount, setAmount] = useState('');
  const [direction, setDirection] = useState('USD_TO_ETB');
  const [result, setResult] = useState('');
  const [balance, setBalance] = useState(0);
  const [transactions, setTransactions] = useState([]);

  const fetchData = async () => {
    const rateRes = await axios.get('http://localhost:5000/rate');
    setRate(rateRes.data.rate);
    const txRes = await axios.get('http://localhost:5000/transactions');
    const userTx = txRes.data.filter(t => t.username === username);
    setTransactions(userTx);
    const bal = userTx.reduce((sum, t) => sum + (t.direction==='USD_TO_ETB'? t.result: 0), 0);
    setBalance(bal);
  };

  useEffect(()=>{ fetchData(); }, []);

  const convert = async () => {
    const res = await axios.post('http://localhost:5000/convert', { username, amount, direction });
    setResult(res.data.result);
    fetchData();
  };

  return (
    <div>
      <h2>Dashboard ({username})</h2>
      <p>Current Rate: 1 USD = {rate} ETB</p>
      <p>Balance: {balance.toFixed(2)} ETB</p>
      <input placeholder="Amount" value={amount} onChange={e=>setAmount(e.target.value)} />
      <select value={direction} onChange={e=>setDirection(e.target.value)}>
        <option value="USD_TO_ETB">USD → ETB</option>
        <option value="ETB_TO_USD">ETB → USD</option>
      </select>
      <button onClick={convert}>Convert</button>
      <h3>Result: {result}</h3>

      <h3>Transactions</h3>
      <ul>
        {transactions.map((t,i)=><li key={i}>{t.amount} {t.direction} = {t.result.toFixed(2)} ETB</li>)}
      </ul>
    </div>
  );
}


6️⃣ Admin.js
import React, { useState } from 'react';
import axios from 'axios';

export default function Admin() {
  const [newRate, setNewRate] = useState('');

  const updateRate = async () => {
    try {
      await axios.post('http://localhost:5000/admin/update-rate', { newRate });
      alert('Rate updated');
    } catch (err) { alert(err.response.data); }
  };

  return (
    <div>
      <h2>Admin Panel</h2>
      <input placeholder="New USD → ETB rate" value={newRate} onChange={e=>setNewRate(e.target.value)} />
      <button onClick={updateRate}>Update Rate</button>
    </div>
  );
}
