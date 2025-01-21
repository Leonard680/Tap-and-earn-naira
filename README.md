# Tap-and-earn-naira
In this app you tap and earn naira and even crypto
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Points Tapping</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background: #f4f4f4;
    }
    .header, .login {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      background: #333;
      color: white;
      padding: 10px 20px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      z-index: 1000;
    }
    .header .balance {
      font-size: 18px;
    }
    .main {
      margin-top: 60px;
      padding: 20px;
      text-align: center;
    }
    .circle-button {
      width: 150px;
      height: 150px;
      border-radius: 50%;
      background: #4caf50;
      color: white;
      font-size: 18px;
      font-weight: bold;
      border: none;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      margin: 20px auto;
    }
    .circle-button:hover {
      background: #45a049;
    }
    .section {
      margin-top: 40px;
      text-align: left;
    }
    textarea, input {
      width: 100%;
      padding: 10px;
      margin-bottom: 10px;
      font-size: 16px;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background: #0056b3;
    }
    .withdraw-requests, .leaderboard {
      margin-top: 20px;
      background: #ffffff;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 5px;
    }
    .countdown {
      color: red;
      font-size: 14px;
    }
  </style>
</head>
<body>
  <div id="login-section" class="login">
    <div>Login with Username:</div>
    <input type="text" id="username" placeholder="Enter your username" />
    <button onclick="login()">Login</button>
  </div>

  <div id="app" style="display: none;">
    <div class="header">
      <div>Welcome to Tap & Earn</div>
      <div class="balance">Balance: <span id="balance">0</span> Points</div>
    </div>
    <div class="main">
      <button class="circle-button" onclick="earnPoints()">Tap Here</button>
      <div class="section">
        <h3>Send Withdrawal Request</h3>
        <input type="text" id="bankName" placeholder="Bank" />
        <input type="text" id="accountNumber" placeholder="Account Number" />
        <input type="text" id="accountName" placeholder="Account Name" />
        <input type="number" id="withdrawPoints" placeholder="Points Amount" />
        <button onclick="sendRequest()">Send Request</button>
      </div>
      <div class="withdraw-requests">
        <h3>All Withdrawal Requests</h3>
        <ul id="requestList"></ul>
      </div>
      <div class="leaderboard">
        <h3>Leaderboard</h3>
        <ul id="leaderboardList"></ul>
      </div>
    </div>
  </div>
  <script>
    let currentUser = null;
    let users = JSON.parse(localStorage.getItem('users')) || {};
    let withdrawalRequests = JSON.parse(localStorage.getItem('withdrawalRequests')) || [];

    function login() {
      const username = document.getElementById('username').value.trim();
      if (!username) {
        alert('Please enter a username.');
        return;
      }

      currentUser = username;
      if (!users[currentUser]) {
        users[currentUser] = { balance: 0 };
        saveUsers();
      }
      document.getElementById('login-section').style.display = 'none';
      document.getElementById('app').style.display = 'block';
      updateBalance();
      renderWithdrawalRequests();
      renderLeaderboard();
    }

    function earnPoints() {
      users[currentUser].balance += 2; // 2 points per tap
      saveUsers();
      updateBalance();
    }

    function sendRequest() {
      const now = new Date();
      const hour = now.getHours();

      if (hour < 16 || hour >= 18) {
        alert('Withdrawals are only allowed between 4:00 PM and 6:00 PM.');
        return;
      }

      const bankName = document.getElementById('bankName').value.trim();
      const accountNumber = document.getElementById('accountNumber').value.trim();
      const accountName = document.getElementById('accountName').value.trim();
      const withdrawPoints = parseInt(document.getElementById('withdrawPoints').value.trim());

      if (!bankName || !accountNumber || !accountName || isNaN(withdrawPoints)) {
        alert('Please fill in all the fields correctly.');
        return;
      }

      if (withdrawPoints < 5000) {
        alert('Minimum withdrawal is 5000 points!');
        return;
      }

      if (users[currentUser].balance < withdrawPoints) {
        alert('Insufficient balance!');
        return;
      }

      const timestamp = new Date().toLocaleString();
      withdrawalRequests.push({
        username: currentUser,
        bankName,
        accountNumber,
        accountName,
        withdrawPoints,
        timestamp,
        expiry: Date.now() + 30 * 60 * 60 * 1000, // 30 hours from now
      });

      users[currentUser].balance -= withdrawPoints;
      saveUsers();
      saveRequests();
      updateBalance();
      renderWithdrawalRequests();
      alert('Withdrawal request sent successfully!');
    }

    function deleteRequest(index) {
      const request = withdrawalRequests[index];
      if (request.username === currentUser || currentUser === 'admin') {
        withdrawalRequests.splice(index, 1);
        saveRequests();
        renderWithdrawalRequests();
        alert('Request deleted successfully!');
      } else {
        alert('You can only delete your own requests!');
      }
    }

    function autoDeleteRequests() {
      const now = Date.now();
      withdrawalRequests = withdrawalRequests.filter(request => now < request.expiry);
      saveRequests();
      renderWithdrawalRequests();
    }

    function saveUsers() {
      localStorage.setItem('users', JSON.stringify(users));
      renderLeaderboard();
    }

    function saveRequests() {
      localStorage.setItem('withdrawalRequests', JSON.stringify(withdrawalRequests));
    }

    function updateBalance() {
      document.getElementById('balance').innerText = users[currentUser]?.balance || 0;
    }

    function renderWithdrawalRequests() {
      const requestList = document.getElementById('requestList');
      requestList.innerHTML = '';
      withdrawalRequests.forEach((request, index) => {
        const timeLeft = Math.max(0, request.expiry - Date.now());
        const hoursLeft = Math.floor(timeLeft / (1000 * 60 * 60));
        const minutesLeft = Math.floor((timeLeft % (1000 * 60 * 60)) / (1000 * 60));

        const li = document.createElement('li');
        li.innerHTML = `
          <strong>${request.username === currentUser ? 'Your' : request.username}'s Request</strong><br>
          Bank: ${request.bankName}<br>
          Acct No: ${request.accountNumber}<br>
          Acct Name: ${request.accountName}<br>
          Points: ${request.withdrawPoints}<br>
          Date: ${request.timestamp}<br>
          <span class="countdown">Expires in: ${hoursLeft}h ${minutesLeft}m</span>
          <button onclick="deleteRequest(${index})">Delete</button>
        `;
        requestList.appendChild(li);
      });
    }

    function renderLeaderboard() {
      const leaderboardList = document.getElementById('leaderboardList');
      leaderboardList.innerHTML = '';

      const sortedUsers = Object.entries(users)
        .sort(([, a], [, b]) => b.balance - a.balance)
        .slice(0, 15);

      sortedUsers.forEach(([username, user]) => {
        const li = document.createElement('li');
        li.innerText = `${username}: ${user.balance} Points`;
        leaderboardList.appendChild(li);
      });
    }

    setInterval(autoDeleteRequests, 60 * 1000); // Check every minute
  </script>
</body>
</html>
