<!DOCTYPE html>
<html>
<head>
<title>Path Innovators HR Dashboard</title>

<meta name="viewport" content="width=device-width, initial-scale=1.0">

<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<style>
*{font-family:'Poppins';box-sizing:border-box;}
body{margin:0;background:#f4f6f9;}

.header{background:navy;color:white;text-align:center;padding:15px;}
.header img{width:60px;height:60px;border-radius:50%;}
.header h1{margin:5px 0;font-size:22px;}

.container{max-width:1000px;margin:10px auto;background:white;padding:15px;border-radius:12px;}

input,select,button,textarea{
width:100%;padding:10px;margin:6px 0;border-radius:6px;border:1px solid #ccc;font-size:14px;
}

button{background:navy;color:white;border:none;cursor:pointer;}
button:hover{background:#001f5c;}

.next-btn{background:green;}
.clear-btn{background:red;}
.download-btn{background:purple;}

.table-container{overflow-x:auto;}

table{width:100%;border-collapse:collapse;min-width:600px;}
th{background:navy;color:white;}
th,td{border:1px solid #ddd;padding:8px;text-align:center;font-size:14px;}

.sent{color:green;font-weight:bold;}
.pending{color:red;}

@media(max-width:600px){
.header h1{font-size:18px;}
}
</style>
</head>

<body>

<div class="header">
<img src="logo.png">
<h1>Path Innovators</h1>
</div>

<div class="container">

<h3>📊 Upload Excel</h3>
<input type="file" id="upload">

<h3>➕ Manual Entry</h3>
<input type="text" id="manualName" placeholder="Candidate Name">
<input type="text" id="manualPhone" placeholder="Mobile Number">
<button onclick="addManual()">Add Candidate</button>

<input type="text" id="search" placeholder="Search candidate..." onkeyup="searchData()">

<label><input type="checkbox" onclick="selectAll(this)"> Select All</label>

<div class="table-container">
<table id="table"></table>
</div>

<h3>💬 Message</h3>

<select id="template">
<option value="interview">Interview</option>
<option value="offer">Offer</option>
<option value="reject">Rejection</option>
</select>

<textarea id="messageBox" rows="4">Hello {name}, Welcome to Path Innovators.</textarea>

<button onclick="applyTemplate()">Apply Template</button>
<button onclick="sendBulk()">📤 Start Sending</button>
<button class="next-btn" onclick="sendNext()">➡️ Next Candidate</button>
<button class="download-btn" onclick="downloadReport()">📥 Download Report</button>
<button class="clear-btn" onclick="clearData()">🗑 Clear Screen Data</button>

<h3>📜 Sent History</h3>
<div class="table-container">
<table id="historyTable"></table>
</div>

</div>

<script>

let data = [];
let historyData = [];
let queue = [];
let currentIndex = 0;

// 🔴 PASTE YOUR LATEST DEPLOYED URL HERE
const API_URL = "https://script.google.com/macros/s/AKfycbz5aNFzMMo8hk5iIsMHMJ8L1OHUiCHzS3pEXDYPm-9UtVQy7XRvbnAWSEoHKQ_Z5-J4kA/exec";


// ================= FORMAT PHONE =================
function formatPhone(phone){
phone = phone.toString().replace(/[^0-9]/g,"");
if(phone.length === 10){
phone = "91" + phone;
}
return phone;
}


// ================= EXCEL UPLOAD =================
document.getElementById('upload').addEventListener('change', function(e){
let reader = new FileReader();

reader.onload = function(e){
let workbook = XLSX.read(e.target.result, {type:'binary'});
let sheet = workbook.Sheets[workbook.SheetNames[0]];
let json = XLSX.utils.sheet_to_json(sheet);

json.forEach(row=>{
let phone = formatPhone(row.Phone || row.phone);

if(phone && !data.some(d => d.Phone === phone)){
data.push({
Name: row.Name || row.name || "Unknown",
Phone: phone,
status: "Pending"
});
}
});

displayTable();
};

reader.readAsBinaryString(e.target.files[0]);
});


// ================= MANUAL ENTRY =================
function addManual(){
let name = document.getElementById("manualName").value.trim();
let phone = formatPhone(document.getElementById("manualPhone").value);

if(!name || !phone){
alert("Enter name & phone");
return;
}

if(data.some(d => d.Phone === phone)){
alert("Candidate already exists!");
return;
}

data.push({Name:name, Phone:phone, status:"Pending"});
displayTable();

document.getElementById("manualName").value="";
document.getElementById("manualPhone").value="";
}


// ================= DISPLAY TABLE =================
function displayTable(){
let table = document.getElementById("table");

if(data.length === 0){
table.innerHTML = "<tr><td colspan='4'>No Data Available</td></tr>";
return;
}

table.innerHTML = "<tr><th>Select</th><th>Name</th><th>Phone</th><th>Status</th></tr>";

data.forEach((row,index)=>{
table.innerHTML += `
<tr>
<td><input type="checkbox" class="select" data-index="${index}"></td>
<td>${row.Name}</td>
<td>${row.Phone}</td>
<td class="${row.status==='Sent'?'sent':'pending'}">${row.status}</td>
</tr>`;
});
}


// ================= HISTORY DISPLAY =================
function displayHistory(){
let table = document.getElementById("historyTable");

if(historyData.length === 0){
table.innerHTML = "<tr><td>No History Yet</td></tr>";
return;
}

table.innerHTML = "<tr><th>Name</th><th>Phone</th><th>Message</th><th>Date</th></tr>";

historyData.forEach(row=>{
table.innerHTML += `
<tr>
<td>${row.name}</td>
<td>${row.phone}</td>
<td>${row.message}</td>
<td>${row.date}</td>
</tr>`;
});
}


// ================= SEARCH =================
function searchData(){
let val = document.getElementById("search").value.toLowerCase();

document.querySelectorAll("#table tr").forEach((row,i)=>{
if(i===0) return;
row.style.display = row.cells[1].innerText.toLowerCase().includes(val) ? "" : "none";
});
}


// ================= SELECT =================
function selectAll(source){
document.querySelectorAll(".select").forEach(cb=>cb.checked=source.checked);
}


// ================= TEMPLATE =================
function applyTemplate(){
let msgs = {
interview:"Hello {name}, You are shortlisted for interview at Path Innovators.",
offer:"Hello {name}, Congratulations! You are selected at Path Innovators.",
reject:"Hello {name}, Thank you for applying to Path Innovators."
};

document.getElementById("messageBox").value = msgs[document.getElementById("template").value];
}


// ================= SEND =================
function sendBulk(){

let selected = document.querySelectorAll(".select:checked");
let template = document.getElementById("messageBox").value;

if(selected.length === 0){
alert("Select candidates");
return;
}

queue = [];

selected.forEach(cb=>{
let person = data[cb.dataset.index];
let msg = template.replace("{name}", person.Name);
queue.push({phone: person.Phone, msg, person});
});

currentIndex = 0;

alert("Send message, then click NEXT");

sendCurrent();
}


function sendCurrent(){

if(currentIndex >= queue.length){
alert("All messages completed");
return;
}

let item = queue[currentIndex];

window.open(`https://wa.me/${item.phone}?text=${encodeURIComponent(item.msg)}`, "_blank");

// update UI
item.person.status = "Sent";
displayTable();

// local history
let record = {
name: item.person.Name,
phone: item.phone,
message: item.msg,
date: new Date().toLocaleString()
};

historyData.push(record);
displayHistory();

// 🔥 GOOGLE SHEET SAVE
fetch(API_URL, {
method: "POST",
headers: {"Content-Type": "application/json"},
body: JSON.stringify({
name: record.name,
phone: record.phone,
message: record.message,
status: "Sent"
})
})
.then(res => res.text())
.then(data => console.log("Saved:", data))
.catch(err => console.error("Error:", err));

}


function sendNext(){
currentIndex++;
sendCurrent();
}


// ================= DOWNLOAD =================
function downloadReport(){

if(historyData.length === 0){
alert("No data to download");
return;
}

let ws = XLSX.utils.json_to_sheet(historyData);
let wb = XLSX.utils.book_new();

XLSX.utils.book_append_sheet(wb, ws, "Report");

XLSX.writeFile(wb, "HR_Report.xlsx");
}


// ================= CLEAR =================
function clearData(){
if(confirm("Clear all data?")){
data=[];
historyData=[];
displayTable();
displayHistory();
}
}

</script>

</body>
</html>
