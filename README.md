<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sinking2026 Coop Tracker</title>
<style>
body{font-family:Arial;padding:10px}
h1,h2{margin:15px 0}
table{border-collapse:collapse;width:100%;table-layout:fixed}
th,td{border:1px solid #333;padding:2px;text-align:center;font-size:10px}
input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button{-webkit-appearance:none;margin:0}
input[type=number]{-moz-appearance:textfield}
input[type="number"], input[type="text"], input[type="date"]{width:36px;text-align:center;font-size:10px}
.paid{background:#c8f7c5}
.unpaid{background:#f7c5c5}
button{padding:5px 10px;margin:5px}
</style>
</head>
<body>

<h1>Sinking2026 Coop Tracker</h1>

<!-- Dashboard Link -->
<p>
  <a href="https://lrdu-innodatalms.talentlms.com/plus/dashboard" target="_blank">
    Open Sinking2026 Dashboard
  </a>
</p>

<!-- Member Contributions -->
<h2>Member Contributions</h2>
<table id="contributionsTable">
  <thead>
    <tr>
      <th>Member</th>
      <th colspan="25">Months</th>
      <th>Total</th>
    </tr>
  </thead>
  <tbody></tbody>
  <tfoot>
    <tr>
      <td colspan="25"><b>OVERALL TOTAL</b></td>
      <td id="overallTotal">0</td>
    </tr>
  </tfoot>
</table>

<!-- Borrowers Table -->
<h2>Borrowers</h2>
<button onclick="addBorrowerRow()">Add Borrower</button>
<table id="borrowersTable">
  <thead>
    <tr>
      <th>Date</th><th>Name</th><th>Borrow</th><th>%</th><th>Income</th>
      <th>Months</th><th>Balance</th><th>1st</th><th>2nd</th><th>3rd</th><th>4th</th><th>5th</th>
      <th>Total+I</th><th>Due</th><th>Status</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
const STORAGE_KEY="sinking2026_all";
const members=["MERVIN","JOANNEMAY","JOLANE","CHRISTYLL ATON","HAPON","JEANNETTE","RAQUEL","JESSANEN","JUNALYN","BRYLLE","JOBERT","MELJIN"];
const contribBody=document.querySelector('#contributionsTable tbody');
const monthsCount=25;
const borrowersBody=document.querySelector('#borrowersTable tbody');
const overallTotalCell=document.getElementById('overallTotal');

// ===== Generate Member Contributions Table =====
members.forEach(name=>{
  const tr=document.createElement('tr');
  tr.innerHTML=`<td>${name}</td>`+
    Array(monthsCount).fill().map(()=>`<td><input type="number" min="0" oninput="updateContrib(this)"></td>`).join('')+
    `<td class="total">0</td>`;
  contribBody.appendChild(tr);
});

function updateContrib(input){
  const row=input.closest('tr');
  let sum=0;
  row.querySelectorAll('input').forEach(i=>sum+=parseFloat(i.value)||0);
  row.querySelector('.total').textContent=sum;
  updateOverallTotal();
  saveAll();
}

// ===== Borrowers Table =====
function addBorrowerRow(){
  const tr=document.createElement('tr');
  tr.innerHTML=`
    <td><input type="date" value="${new Date().toISOString().split('T')[0]}"></td>
    <td><input type="text" oninput="updateBorrower(this)"></td>
    <td><input type="number" min="0" value="0" oninput="updateBorrower(this)"></td>
    <td><input type="number" min="0" value="0" oninput="updateBorrower(this)"></td>
    <td><input type="number" readonly></td>
    <td><input type="number" min="1" value="1" oninput="updateBorrower(this)"></td>
    <td class="balance">0</td>
    <td><input type="number" value="0" oninput="updateBorrower(this)"></td>
    <td><input type="number" value="0" oninput="updateBorrower(this)"></td>
    <td><input type="number" value="0" oninput="updateBorrower(this)"></td>
    <td><input type="number" value="0" oninput="updateBorrower(this)"></td>
    <td><input type="number" value="0" oninput="updateBorrower(this)"></td>
    <td class="totalI">0</td>
    <td class="due"></td>
    <td><select onchange="updateBorrower(this)"><option value='Unpaid'>Unpaid</option><option value='Paid'>Paid</option></select></td>
  `;
  borrowersBody.appendChild(tr);
  updateBorrower(tr);
}

function updateBorrower(el){
  const tr=el.closest('tr')||el;
  const borrow=parseFloat(tr.cells[2].querySelector('input').value)||0;
  const pct=parseFloat(tr.cells[3].querySelector('input').value)||0;
  const months=parseInt(tr.cells[5].querySelector('input').value)||1;

  const income=borrow*(pct/100)*months;
  tr.cells[4].querySelector('input').value=income.toFixed(2);
  const totalI=borrow+income;
  tr.querySelector('.totalI').textContent=totalI.toFixed(2);

  let sumInstall=0;
  for(let i=7;i<=11;i++) sumInstall+=parseFloat(tr.cells[i].querySelector('input').value)||0;
  const balance=totalI-sumInstall;
  tr.querySelector('.balance').textContent=balance.toFixed(2);

  // Due date
  const borrowDate=new Date(tr.cells[0].querySelector('input').value);
  borrowDate.setMonth(borrowDate.getMonth()+months);
  tr.querySelector('.due').textContent=borrowDate.toISOString().split('T')[0];

  // Status
  const select=tr.querySelector('select');
  if(balance<=0){select.value='Paid';tr.className='paid';}
  else{select.value='Unpaid';tr.className='unpaid';}

  updateOverallTotal();
  saveAll();
}

// ===== Overall Total =====
function updateOverallTotal(){
  let contribSum=0;
  contribBody.querySelectorAll('tr').forEach(r=>{
    r.querySelectorAll('input').forEach(i=>contribSum+=parseFloat(i.value)||0);
  });

  let borrowSum=0;
  borrowersBody.querySelectorAll('tr').forEach(r=>{
    borrowSum+=parseFloat(r.cells[2].querySelector('input').value)||0;
  });

  overallTotalCell.textContent=(contribSum-borrowSum).toFixed(2);
}

// ===== Save / Load =====
function saveAll(){
  const contribData=[...contribBody.querySelectorAll('tr')].map(r=>[...r.querySelectorAll('input')].map(i=>i.value));
  const borrowData=[...borrowersBody.querySelectorAll('tr')].map(r=>({
    date:r.cells[0].querySelector('input').value,
    name:r.cells[1].querySelector('input').value,
    borrow:r.cells[2].querySelector('input').value,
    pct:r.cells[3].querySelector('input').value,
    income:r.cells[4].querySelector('input').value,
    months:r.cells[5].querySelector('input').value,
    balance:r.cells[6].textContent,
    installments:[...Array(5)].map((_,i)=>r.cells[7+i].querySelector('input').value),
    totalI:r.querySelector('.totalI').textContent,
    due:r.querySelector('.due').textContent,
    status:r.querySelector('select').value
  }));
  localStorage.setItem(STORAGE_KEY,JSON.stringify({contribData,borrowData}));
}

function loadAll(){
  const saved=JSON.parse(localStorage.getItem(STORAGE_KEY));
  if(!saved) return;
  saved.contribData.forEach((row,i)=>{
    const tr=contribBody.children[i];
    row.forEach((v,j)=>tr.querySelectorAll('input')[j].value=v);
    tr.querySelectorAll('input').forEach(input=>input.dispatchEvent(new Event('input')));
  });
  saved.borrowData.forEach(data=>{
    addBorrowerRow();
    const tr=borrowersBody.lastElementChild;
    tr.cells[0].querySelector('input').value=data.date;
    tr.cells[1].querySelector('input').value=data.name;
    tr.cells[2].querySelector('input').value=data.borrow;
    tr.cells[3].querySelector('input').value=data.pct;
    tr.cells[4].querySelector('input').value=data.income;
    tr.cells[5].querySelector('input').value=data.months;
    data.installments.forEach((v,i)=>tr.cells[7+i].querySelector('input').value=v);
    tr.querySelector('.totalI').textContent=data.totalI;
    tr.querySelector('.due').textContent=data.due;
    tr.querySelector('select').value=data.status;
    tr.className=data.status==='Paid'?'paid':'unpaid';
    updateBorrower(tr);
  });
}

window.addEventListener('load',loadAll);
</script>

</body>
</html>
