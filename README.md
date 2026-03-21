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

.container{max-width:1000px;margin:15px auto;background:white;padding:15px;border-radius:12px;}

input,select,button,textarea{
width:100%;padding:10px;margin:6px 0;border-radius:6px;border:1px solid #ccc;
}

button{background:navy;color:white;border:none;cursor:pointer;}
button:hover{background:#001f5c;}

table{width:100%;border-collapse:collapse;}
th{background:navy;color:white;}
th,td{border:1px solid #ddd;padding:8px;text-align:center;}

.sent{color:green;}
.pending{color:red;}
</style>
</head>

<body>

<div class="header">
<img src="logo.png">
<h1>Path Innovators</h1>
</div>

<div class="container">

<h3>Upload Excel</h3>
<input type="file" id="upload">

<input type="text" id="search" placeholder="Search..." onkeyup="searchData()">

<label><input type="checkbox" onclick="selectAll(this)"> Select All</label>

<table id="table"></table>

<h3>Message</h3>

<select id="template">
<option value="interview">Interview</option>
<option value="offer">Offer</option>
<option value="reject">Rejection</option>
</select>

<textarea id="messageBox" rows="4">Hello {name}, Welcome to Path Innovators.</textarea>

<button onclick="applyTemplate()">Apply Template</button>
<button onclick="sendBulk()">Send Selected</button>

</div>

<script>
let data = [];

document.getElementById('upload').addEventListener('change', function(e){
let reader = new FileReader();

reader.onload = function(e){
let workbook = XLSX.read(e.target.result, {type:'binary'});
let sheet = workbook.Sheets[workbook.SheetNames[0]];
let json = XLSX.utils.sheet_to_json(sheet);

// Normalize data safely
data = json.map(row => ({
Name: row.Name || row.name || "Unknown",
Phone: row.Phone || row.phone || "",
status: "Pending"
}));

displayTable();
};

reader.readAsBinaryString(e.target.files[0]);
});

function displayTable(){
let table = document.getElementById("table");

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

function searchData(){
let value = document.getElementById("search").value.toLowerCase();

let rows = document.querySelectorAll("#table tr");

rows.forEach((row,i)=>{
if(i===0) return;

let name = row.cells[1].innerText.toLowerCase();
row.style.display = name.includes(value) ? "" : "none";
});
}

function selectAll(source){
document.querySelectorAll(".select").forEach(cb => cb.checked = source.checked);
}

function applyTemplate(){
let type = document.getElementById("template").value;
let msg="";

if(type==="interview"){
msg="Hello {name}, You are shortlisted for an interview at Path Innovators.";
}
else if(type==="offer"){
msg="Hello {name}, Congratulations! You are selected at Path Innovators.";
}
else{
msg="Hello {name}, Thank you for applying.";
}

document.getElementById("messageBox").value = msg;
}

function sendBulk(){
let selected = document.querySelectorAll(".select:checked");
let template = document.getElementById("messageBox").value;

if(!template){
alert("Message is empty!");
return;
}

selected.forEach((cb,i)=>{
let person = data[cb.dataset.index];

let phone = person.Phone.toString().replace(/[^0-9]/g,"");

if(!phone){
alert("Invalid phone for " + person.Name);
return;
}

let msg = template.replace("{name}", person.Name);

setTimeout(()=>{
window.open(`https://wa.me/${phone}?text=${encodeURIComponent(msg)}`, "_blank");
person.status = "Sent";
displayTable();
}, i * 1500);
});
}
</script>

</body>
</html>
