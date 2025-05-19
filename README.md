# LoanApp.
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Loan Tracker</title>
  <style>
    :root {
      --bg: #f5f3ff;
      --text: #1c1c1c;
      --card: #ffffff;
      --accent-gradient: linear-gradient(135deg, #8e2de2, #4a00e0);
      --button-bg: #7b2cbf;
    }
    body.dark {
      --bg: #1e1b2e;
      --text: #eee;
      --card: #2c234d;
      --accent-gradient: linear-gradient(135deg, #a18cd1, #fbc2eb);
      --button-bg: #a259ff;
    }

    body {
      margin: 0;
      font-family: 'Segoe UI', sans-serif;
      background: var(--bg);
      color: var(--text);
      transition: 0.4s;
    }

    .toggle {
      position: fixed;
      top: 15px;
      right: 20px;
      background: var(--button-bg);
      color: white;
      border: none;
      border-radius: 20px;
      padding: 8px 15px;
      cursor: pointer;
      font-weight: bold;
      box-shadow: 0 2px 8px rgba(0,0,0,0.2);
    }

    .center {
      text-align: center;
      padding-top: 100px;
    }

    .big-plus {
      font-size: 80px;
      color: white;
      background: var(--accent-gradient);
      border-radius: 50%;
      width: 100px;
      height: 100px;
      line-height: 100px;
      margin: auto;
      cursor: pointer;
      transition: transform 0.3s ease;
      box-shadow: 0 10px 20px rgba(0,0,0,0.3);
    }

    .big-plus:hover {
      transform: scale(1.1);
    }

    .card {
      background: var(--card);
      max-width: 600px;
      margin: 20px auto;
      padding: 25px;
      border-radius: 15px;
      box-shadow: 0 4px 15px rgba(0,0,0,0.1);
      animation: fadeIn 0.4s ease;
    }

    @keyframes fadeIn {
      from {opacity: 0; transform: translateY(20px);}
      to {opacity: 1; transform: translateY(0);}
    }

    h2 {
      margin-top: 0;
      text-align: center;
    }

    input {
      width: 100%;
      padding: 10px;
      margin: 10px 0;
      border: none;
      border-radius: 10px;
      background: #f0f0f0;
      transition: 0.2s;
    }

    body.dark input {
      background: #3b3260;
      color: white;
    }

    input:focus {
      outline: 2px solid #a259ff;
    }

    button {
      background: var(--button-bg);
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 10px;
      cursor: pointer;
      margin: 10px 5px 0 0;
      font-weight: bold;
      transition: background 0.2s;
    }

    button:hover {
      background: #5e1fb0;
    }

    .summary-row {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 20px;
      margin-top: 20px;
    }

    .summary-box {
      flex: 1 1 150px;
      background: var(--accent-gradient);
      color: white;
      padding: 15px;
      border-radius: 15px;
      text-align: center;
      box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    }

    .overview-table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }

    .overview-table th, .overview-table td {
      padding: 12px;
      text-align: center;
      border-bottom: 1px solid #ccc;
    }

    .footer {
      text-align: center;
      font-size: 12px;
      margin: 30px 10px 10px;
      color: gray;
    }

    .actions button {
      margin: 0 2px;
      background: #7b2cbf;
    }

    .actions button.delete {
      background: #d0006f;
    }
  </style>
</head>
<body>

  <button class="toggle" onclick="toggleTheme()">Toggle Mode</button>

  <div class="center">
    <div class="big-plus" onclick="showForm()">+</div>
  </div>

  <div class="card" id="formCard" style="display:none;">
    <h2 id="formTitle">Add Loan Entry</h2>
    <input type="text" id="name" placeholder="Name" />
    <input type="number" id="loan" placeholder="Loan Amount (?)" oninput="calc()" />
    <input type="number" id="interest" placeholder="Interest (?)" oninput="calc()" />
    <input type="text" id="total" placeholder="Total Amount (?)" readonly />

    <div id="ewiInputs"></div>

    <input type="text" id="paid" placeholder="Total Paid (?)" readonly />
    <input type="text" id="balance" placeholder="Balance (?)" readonly />

    <button onclick="saveEntry()">Save</button>
    <button onclick="showOverview()">Overview</button>
  </div>

  <div class="card" id="overviewCard" style="display:none;">
    <h2>All Entries</h2>
    <table class="overview-table">
      <thead>
        <tr>
          <th>Name</th>
          <th>Loan</th>
          <th>Interest</th>
          <th>Total</th>
          <th>Paid</th>
          <th>Balance</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody id="overviewBody"></tbody>
    </table>
  </div>

  <div id="summaryContainer"></div>

  <div class="footer">This app is made by Yugan.</div>

  <script>
    let entries = JSON.parse(localStorage.getItem("loanEntries") || "[]");
    let editingIndex = null;

    function showForm(index = null) {
      document.getElementById("formCard").style.display = "block";
      document.getElementById("overviewCard").style.display = "none";
      document.getElementById("formTitle").innerText = index !== null ? "Edit Loan Entry" : "Add Loan Entry";
      editingIndex = index;

      if (index !== null) {
        const e = entries[index];
        document.getElementById("name").value = e.name;
        document.getElementById("loan").value = e.loan;
        document.getElementById("interest").value = e.interest;
        renderEWI(e.ewis);
      } else {
        document.getElementById("name").value = "";
        document.getElementById("loan").value = "";
        document.getElementById("interest").value = "";
        renderEWI([]);
      }

      calc();
    }

    function renderEWI(ewis = []) {
      const container = document.getElementById("ewiInputs");
      container.innerHTML = "";
      for (let i = 0; i < 10 || i < ewis.length; i++) {
        const input = document.createElement("input");
        input.type = "number";
        input.className = "ewi";
        input.placeholder = `EWI ${i + 1}`;
        input.value = ewis[i] || "";
        input.oninput = calc;
        container.appendChild(input);
      }
    }

    function calc() {
      const loan = parseFloat(document.getElementById("loan").value) || 0;
      const interest = parseFloat(document.getElementById("interest").value) || 0;
      const total = loan + interest;
      document.getElementById("total").value = total.toFixed(2);

      let paid = 0;
      const ewis = document.querySelectorAll(".ewi");
      ewis.forEach(e => paid += parseFloat(e.value) || 0);
      document.getElementById("paid").value = paid.toFixed(2);
      document.getElementById("balance").value = (total - paid).toFixed(2);
    }

    function saveEntry() {
      const name = document.getElementById("name").value.trim();
      const loan = parseFloat(document.getElementById("loan").value) || 0;
      const interest = parseFloat(document.getElementById("interest").value) || 0;
      const total = parseFloat(document.getElementById("total").value) || 0;
      const paid = parseFloat(document.getElementById("paid").value) || 0;
      const balance = parseFloat(document.getElementById("balance").value) || 0;
      const ewis = Array.from(document.querySelectorAll(".ewi")).map(e => parseFloat(e.value) || 0);

      if (!name) return alert("Enter name");

      const entry = { name, loan, interest, total, paid, balance, ewis };

      if (editingIndex !== null) {
        entries[editingIndex] = entry;
      } else {
        entries.push(entry);
      }

      localStorage.setItem("loanEntries", JSON.stringify(entries));
      showOverview();
    }

    function showOverview() {
      document.getElementById("formCard").style.display = "none";
      document.getElementById("overviewCard").style.display = "block";

      const tbody = document.getElementById("overviewBody");
      tbody.innerHTML = "";

      const summary = document.getElementById("summaryContainer");
      summary.innerHTML = `<div class="summary-row">` + entries.map(entry => `
        <div class="summary-box">
          <strong>${entry.name}</strong><br>
          Paid: ?${entry.paid}<br>
          Balance: ?${entry.balance}
        </div>
      `).join('') + `</div>`;

      entries.forEach((e, i) => {
        tbody.innerHTML += `
          <tr>
            <td>${e.name}</td>
            <td>?${e.loan}</td>
            <td>?${e.interest}</td>
            <td>?${e.total}</td>
            <td>?${e.paid}</td>
            <td>?${e.balance}</td>
            <td class="actions">
              <button onclick="showForm(${i})">Edit</button>
              <button class="delete" onclick="deleteEntry(${i})">Delete</button>
            </td>
          </tr>`;
      });
    }

    function deleteEntry(i) {
      if (confirm("Delete this entry?")) {
        entries.splice(i, 1);
        localStorage.setItem("loanEntries", JSON.stringify(entries));
        showOverview();
      }
    }

    function toggleTheme() {
      document.body.classList.toggle("dark");
    }

    showOverview();
  </script>
</body>
</html>
