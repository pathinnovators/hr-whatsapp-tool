<!DOCTYPE html>
<html>
<head>
  <title>Path Innovators HR Dashboard</title>

  <!-- Poppins Font -->
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap" rel="stylesheet">

  <style>
    * {
      font-family: 'Poppins', sans-serif;
      box-sizing: border-box;
    }

    body {
      margin: 0;
      background: #f4f6f9;
    }

    /* HEADER */
    .header {
      background: navy;
      color: white;
      text-align: center;
      padding: 15px;
    }

    .header img {
      width: 60px;
      height: 60px;
      border-radius: 50%;
      display: block;
      margin: 0 auto 8px;
    }

    .header h2 {
      margin: 0;
      font-size: 20px;
    }

    /* MAIN CONTAINER */
    .container {
      max-width: 1000px;
      margin: 15px auto;
      background: white;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }

    /* FORM ELEMENTS */
    input, select, textarea, button {
      width: 100%;
      padding: 10px;
      margin: 6px 0;
      border-radius: 6px;
      border: 1px solid #ccc;
      font-size: 14px;
    }

    button {
      background: navy;
      color: white;
      border: none;
      cursor: pointer;
    }

    /* BUTTON GROUP */
    .btn-group {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
    }

    .btn-group button {
      flex: 1;
    }

    /* SELECT ALL */
    .select-all {
      display: flex;
      align-items: center;
      gap: 8px;
      margin: 10px 0;
    }

    .select-all input {
      width: auto;
    }

    /* TABLE */
    .table-container {
      overflow-x: auto;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      min-width: 600px;
    }

    th {
      background: navy;
      color: white;
    }

    th, td {
      border: 1px solid #ddd;
      padding: 8px;
      text-align: center;
    }

    .sent { color: green; font-weight: bold; }
    .pending { color: red; }

    footer {
      text-align: center;
      padding: 10px;
      color: gray;
      font-size: 13px;
    }

    /* MOBILE */
    @media(max-width:600px){
      .header h2 {
        font-size: 16px;
      }

      table {
        font-size: 12px;
      }
    }
  </style>
</head>

<body>

<!-- HEADER -->
<div class="header">
  <img src="logo.png">
  <h2>Path Innovators</h2>
</div>

<div class="container">

  <h3>📱 Manual Send</h3>
  <input id="manualName" placeholder="Candidate Name">
  <input id="manualPhone" placeholder="Phone (91XXXXXXXXXX)">
  <button onclick="sendManual()">Send</button>

  <hr>

  <h3>📊 Live Candidate Data</h3>

  <input type="text" id="search" placeholder="Search Candidate..." onkeyup="searchData()">

  <!-- SELECT ALL -->
  <div class="select-all">
    <input type="checkbox" id="selectAll" onclick="selectAll(this)">
    <label for="selectAll">Select All</label>
  </div>

  <!-- TABLE -->
  <div class="table-container">
    <table id="table"></table>
  </div>

  <h3>💬 Message</h3>
  <select id="template">
    <option value="interview">Interview</option>
    <option value="offer">Offer</option>
    <option value="reject">Rejection</option>
  </select>

  <textarea id="messageBox" rows="4"></textarea>

  <!-- BUTTON GROUP -->
  <div class="btn-group">
    <button onclick="applyTemplate()">Apply</button>
    <button onclick="previewMessage()">Preview</button>
    <button onclick="sendBulk()">Send</button>
  </div>

</div>

<footer>© Path Innovators 2026</footer>

<script>
let data = [];
let filteredData = [];

// GOOGLE SHEETS
fetch("https://opensheet.elk.sh/YOUR_SHEET_ID/Sheet1")
  .then(res => res.json())
  .then(res => {
    data = res;
    data.forEach(d => d.status = "Pending");
    filteredData = data;
    displayTable();
  });

// TABLE
function displayTable(){
  let table = document.getElementById("table");

  table.innerHTML = `
    <tr>
      <th>Select</th>
      <th>Name</th>
      <th>Phone</th>
      <th>Email</th>
      <th>Status</th>
    </tr>
  `;

  filteredData.forEach((row, index)=>{
    table.innerHTML += `
      <tr>
        <td><input type="checkbox" class="select" data-index="${index}"></td>
        <td>${row.Name}</td>
        <td>${row.Phone}</td>
        <td>${row.Email}</td>
        <td class="${row.status === 'Sent' ? 'sent' : 'pending'}">${row.status}</td>
      </tr>
    `;
  });
}

// SEARCH
function searchData(){
  let value = document.getElementById("search").value.toLowerCase();
  filteredData = data.filter(d => d.Name.toLowerCase().includes(value));
  displayTable();
}

// SELECT ALL
function selectAll(source){
  document.querySelectorAll(".select").forEach(cb => cb.checked = source.checked);
}

// TEMPLATE
function applyTemplate(){
  let type = document.getElementById("template").value;

  let msg = "";
  if(type === "interview"){
    msg = "Hello {name}, You are shortlisted for an interview at Path Innovators.";
  }
  else if(type === "offer"){
    msg = "Hello {name}, Congratulations! You are selected at Path Innovators.";
  }
  else{
    msg = "Hello {name}, Thank you for applying. Currently we are not proceeding.";
  }

  document.getElementById("messageBox").value = msg;
}

// PREVIEW
function previewMessage(){
  alert(document.getElementById("messageBox").value);
}

// BULK SEND
function sendBulk(){
  let selected = document.querySelectorAll(".select:checked");
  let template = document.getElementById("messageBox").value;

  selected.forEach((cb, i)=>{
    let person = filteredData[cb.dataset.index];
    let phone = person.Phone.toString().replace(/[^0-9]/g,"");
    let msg = template.replace("{name}", person.Name);

    let url = "https://api.whatsapp.com/send?phone=" + phone + "&text=" + encodeURIComponent(msg);

    setTimeout(()=>{
      window.open(url, "_blank");
      person.status = "Sent";
      displayTable();
    }, i * 1500);
  });
}

// MANUAL
function sendManual(){
  let name = document.getElementById("manualName").value;
  let phone = document.getElementById("manualPhone").value;

  if(!name || !phone){
    alert("Enter details");
    return;
  }

  phone = phone.replace(/[^0-9]/g,"");

  let msg = document.getElementById("messageBox").value.replace("{name}", name);

  let url = "https://api.whatsapp.com/send?phone=" + phone + "&text=" + encodeURIComponent(msg);

  window.open(url, "_blank");
}
</script>

</body>
</html>
