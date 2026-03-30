<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Employee Attendance & Salary Manager</title>
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
  <style>
    :root {
      --primary: #4a6ea9;
      --success: #27ae60;
      --danger: #e74c3c;
      --warning: #f1c40f;
      --info: #3498db;
      --light: #f8f9fa;
      --dark: #2c3e50;
    }
    body { font-family: system-ui, sans-serif; margin: 0; padding: 20px; background: #f5f7fa; }
    .container { max-width: 1200px; margin: auto; background: white; padding: 0; border-radius: 10px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
    .header { background: var(--primary); color: white; padding: 15px 20px; border-radius: 10px 10px 0 0; display: flex; justify-content: space-between; align-items: center; }
    .header h1 { margin: 0; font-size: 20px; }
    .nav-buttons { display: flex; gap: 10px; }
    .nav-buttons button { background: rgba(255,255,255,0.2); color: white; border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; }
    .nav-buttons button.active { background: white; color: var(--primary); }
    .section { padding: 20px; display: none; }
    .section.active { display: block; }
    .filter-bar { display: flex; gap: 10px; flex-wrap: wrap; margin-bottom: 20px; align-items: center; }
    .filter-bar select, .filter-bar input, .filter-bar button { padding: 8px; border: 1px solid #ddd; border-radius: 5px; }
    .filter-bar button { background: var(--primary); color: white; border: none; cursor: pointer; }
    .summary-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 15px; margin-bottom: 20px; }
    .summary-card { background: white; border: 1px solid #eee; border-radius: 8px; padding: 15px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
    .summary-card h3 { margin: 0 0 10px 0; color: var(--dark); font-size: 14px; }
    .summary-card .value { font-size: 24px; font-weight: bold; color: var(--primary); }
    .summary-card .label { font-size: 12px; color: #666; }
    .salary-breakdown { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 20px; }
    .breakdown-col { background: white; border: 1px solid #eee; border-radius: 8px; padding: 15px; }
    .breakdown-row { display: flex; justify-content: space-between; margin: 5px 0; font-size: 14px; }
    .breakdown-row.deduction { color: var(--danger); }
    .breakdown-row.addition { color: var(--success); }
    .final-calculation { background: #e8f5e9; border-radius: 8px; padding: 15px; margin-top: 20px; }
    .final-calculation h3 { margin: 0 0 10px 0; color: var(--success); }
    .attendance-table { width: 100%; border-collapse: collapse; margin-top: 20px; }
    .attendance-table th { background: var(--light); padding: 10px; text-align: left; border-bottom: 2px solid #ddd; }
    .attendance-table td { padding: 10px; border-bottom: 1px solid #eee; }
    .attendance-table tr.weekend { background: #fff9c4; }
    .attendance-table tr.absent { background: #ffebee; }
    .attendance-table .late { color: var(--danger); font-weight: bold; }
    .attendance-table .ot { color: var(--success); }
    .btn { background: var(--success); color: white; border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; margin: 5px 0; }
    .btn.secondary { background: var(--info); }
    .btn.danger { background: var(--danger); }
    .deduction-rules { background: #fff3cd; padding: 10px; border-radius: 5px; margin-bottom: 15px; font-size: 14px; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>📋 Employee Attendance & Salary Manager</h1>
      <div class="nav-buttons">
        <button class="active" onclick="showSection('dashboard')">Dashboard</button>
        <button onclick="showSection('employees')">Employees</button>
        <button onclick="showSection('import')">Import Data</button>
        <button onclick="showSection('reports')">Reports</button>
      </div>
    </div>

    <!-- DASHBOARD SECTION -->
    <div id="dashboard" class="section active">
      <div class="filter-bar">
        <select id="empSelectDashboard">
          <option value="">Select Employee</option>
        </select>
        <select id="monthSelect">
          <option value="01">January</option>
          <option value="02">February</option>
          <option value="03">March</option>
          <option value="04">April</option>
          <option value="05">May</option>
          <option value="06">June</option>
          <option value="07">July</option>
          <option value="08">August</option>
          <option value="09">September</option>
          <option value="10">October</option>
          <option value="11">November</option>
          <option value="12">December</option>
        </select>
        <input type="number" id="yearSelect" value="2024" min="2020" max="2030">
        <button onclick="loadEmployeeDashboard()">🔍 Show</button>
      </div>

      <div id="employeeDashboard" style="display: none;">
        <h2 id="dashboardTitle"></h2>

        <div class="summary-grid">
          <div class="summary-card">
            <h3>Scheduled In/Out</h3>
            <div class="value" id="scheduledInOut"></div>
          </div>
          <div class="summary-card">
            <h3>Attendance Days</h3>
            <div class="value">
              <span id="presentDays">0</span> Present | 
              <span id="absentDays">0</span> Absent | 
              <span id="halfDays">0</span> Half Day
            </div>
          </div>
          <div class="summary-card">
            <h3>Time Summary</h3>
            <div class="value">
              <span id="totalLate">00:00</span> Late | 
              <span id="totalOT">00:00</span> OT
            </div>
          </div>
          <div class="summary-card">
            <h3>Extra Days</h3>
            <div class="value">
              <span id="extraDays">0</span> Total
            </div>
          </div>
        </div>

        <div class="salary-breakdown">
          <div class="breakdown-col">
            <h3>Salary Details</h3>
            <div class="breakdown-row">
              <span>Monthly Salary</span>
              <span id="monthlySalary">₹0.00</span>
            </div>
            <div class="breakdown-row">
              <span>Per Day Salary</span>
              <span id="perDaySalary">₹0.00</span>
            </div>
            <div class="breakdown-row">
              <span>Per Hour Salary</span>
              <span id="perHourSalary">₹0.00</span>
            </div>
            <div class="breakdown-row">
              <span>Per Minute Salary</span>
              <span id="perMinSalary">₹0.00</span>
            </div>
          </div>
          <div class="breakdown-col">
            <h3>Deductions & Additions</h3>
            <div class="breakdown-row deduction">
              <span>Late Deduction</span>
              <span id="lateDeduction">- ₹0.00</span>
            </div>
            <div class="breakdown-row deduction">
              <span>Absence Deduction</span>
              <span id="absenceDeduction">- ₹0.00</span>
            </div>
            <div class="breakdown-row deduction">
              <span>Half Day Deduction</span>
              <span id="halfDayDeduction">- ₹0.00</span>
            </div>
            <div class="breakdown-row addition">
              <span>Extra Days Addition</span>
              <span id="extraAddition">+ ₹0.00</span>
            </div>
          </div>
        </div>

        <div class="final-calculation">
          <h3>Final Calculation</h3>
          <div class="breakdown-row">
            <span>Monthly Salary</span>
            <span id="finalSalary">₹0.00</span>
          </div>
          <div class="breakdown-row deduction">
            <span>Total Deductions</span>
            <span id="totalDeductions">- ₹0.00</span>
          </div>
          <div class="breakdown-row" style="font-weight: bold; font-size: 16px; margin-top: 10px;">
            <span>Net Payable</span>
            <span id="netPayable">₹0.00</span>
          </div>
        </div>

        <h3>Daily Attendance</h3>
        <table class="attendance-table">
          <thead>
            <tr>
              <th>Date</th>
              <th>In Time</th>
              <th>Out Time</th>
              <th>Late</th>
              <th>OT</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody id="dailyAttendanceTable"></tbody>
        </table>
      </div>
    </div>

    <!-- EMPLOYEES SECTION -->
    <div id="employees" class="section">
      <h2>Add Employee</h2>
      <div class="filter-bar">
        <input type="text" id="empName" placeholder="Employee Name">
        <input type="text" id="empCode" placeholder="Employee Code">
        <input type="number" id="empSalary" placeholder="Monthly Salary (₹)">
        <button onclick="addEmployee()">Add Employee</button>
      </div>
      <h3>Employee List</h3>
      <table class="attendance-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Code</th>
            <th>Salary</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody id="empTable"></tbody>
      </table>
    </div>

    <!-- IMPORT SECTION -->
    <div id="import" class="section">
      <h2>Import Punch Data</h2>
      <label class="btn secondary" style="width: auto;">
        📂 Choose File
        <input type="file" id="importFile" accept=".xlsx,.csv,.txt" style="display: none;" onchange="handleImportFile(event)">
      </label>
      <div id="importPreview" style="margin-top: 20px;"></div>
      <button onclick="processImport()" class="btn" style="margin-top: 10px;">Process Import</button>
    </div>

    <!-- REPORTS SECTION -->
    <div id="reports" class="section">
      <h2>Monthly Salary Report</h2>
      <div class="filter-bar">
        <select id="reportMonth">
          <option value="01">January</option>
          <option value="02">February</option>
          <option value="03">March</option>
          <option value="04">April</option>
          <option value="05">May</option>
          <option value="06">June</option>
          <option value="07">July</option>
          <option value="08">August</option>
          <option value="09">September</option>
          <option value="10">October</option>
          <option value="11">November</option>
          <option value="12">December</option>
        </select>
        <input type="number" id="reportYear" value="2024" min="2020" max="2030">
        <button onclick="generateReport()" class="btn">Generate Report</button>
        <button onclick="exportReport()" class="btn secondary">Export to Excel</button>
      </div>
      <table class="attendance-table" id="reportTable">
        <thead>
          <tr>
            <th>Employee</th>
            <th>Code</th>
            <th>Present</th>
            <th>Absent</th>
            <th>Late</th>
            <th>OT</th>
            <th>Deductions</th>
            <th>Net Salary</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>
  </div>

  <script>
    // CONFIG
    const SETTINGS = {
      shiftIn: "09:00:00",
      shiftOut: "19:00:00",
      workingDays: 26,
      lateDeductionPerMin: 0.75,
      absentDeductionPerDay: 450,
      halfDayDeduction: 225,
      otRatePerHour: 90
    };

    // DATA
    let employees = JSON.parse(localStorage.getItem('employees')) || [];
    let attendance = JSON.parse(localStorage.getItem('attendance')) || [];

    // INIT
    renderEmployees();
    populateEmpSelect();
    document.getElementById('employeeDashboard').style.display = 'none';

    // SECTIONS
    function showSection(sectionId) {
      document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
      document.querySelectorAll('.nav-buttons button').forEach(b => b.classList.remove('active'));
      document.getElementById(sectionId).classList.add('active');
      event.target.classList.add('active');
    }

    // EMPLOYEES
    function addEmployee() {
      const name = document.getElementById('empName').value.trim();
      const code = document.getElementById('empCode').value.trim();
      const salary = parseFloat(document.getElementById('empSalary').value);
      if(!name || !salary) return alert('Please enter name and salary');
      const emp = { id: Date.now(), name, code, salary };
      employees.push(emp);
      saveData();
      renderEmployees();
      populateEmpSelect();
      document.getElementById('empName').value = '';
      document.getElementById('empCode').value = '';
      document.getElementById('empSalary').value = '';
    }

    function renderEmployees() {
      const tbody = document.getElementById('empTable');
      tbody.innerHTML = '';
      employees.forEach(emp => {
        tbody.innerHTML += `
          <tr>
            <td>${emp.id}</td>
            <td>${emp.name}</td>
            <td>${emp.code}</td>
            <td>₹${emp.salary.toFixed(2)}</td>
            <td><button class="btn danger" onclick="deleteEmployee(${emp.id})">Delete</button></td>
          </tr>
        `;
      });
    }

    function deleteEmployee(id) {
      if(!confirm('Delete this employee?')) return;
      employees = employees.filter(e => e.id !== id);
      attendance = attendance.filter(a => a.empId !== id);
      saveData();
      renderEmployees();
      populateEmpSelect();
    }

    function populateEmpSelect() {
      const select = document.getElementById('empSelectDashboard');
      select.innerHTML = '<option value="">Select Employee</option>';
      employees.forEach(emp => {
        select.innerHTML += `<option value="${emp.id}">${emp.name} (${emp.code})</option>`;
      });
    }

    // DASHBOARD
    function loadEmployeeDashboard() {
      const empId = parseInt(document.getElementById('empSelectDashboard').value);
      const month = document.getElementById('monthSelect').value;
      const year = document.getElementById('yearSelect').value;
      if(!empId) return alert('Select an employee');

      const emp = employees.find(e => e.id === empId);
      if(!emp) return alert('Employee not found');

      const monthName = new Date(year, month-1, 1).toLocaleString('default', { month: 'long' });
      document.getElementById('dashboardTitle').textContent = `${emp.name} - ${monthName} ${year}`;

      const empAttendance = attendance.filter(a => 
        a.empId === empId && 
        a.date.startsWith(`${year}-${month}`)
      );

      calculateDashboard(emp, empAttendance);
      renderDailyAttendance(empAttendance);
      document.getElementById('employeeDashboard').style.display = 'block';
    }

    function calculateDashboard(emp, empAttendance) {
      let present = 0, absent = 0, half = 0;
      let totalLate = 0, totalOT = 0;
      let lateDeduction = 0, absenceDeduction = 0, halfDeduction = 0;
      let extraDays = 0;

      const perDay = emp.salary / SETTINGS.workingDays;
      const perHour = perDay / 8;
      const perMin = perHour / 60;

      empAttendance.forEach(rec => {
        if(rec.status === 'present') present++;
        if(rec.status === 'absent') absent++;
        if(rec.status === 'half') half++;

        totalLate += rec.lateMinutes || 0;
        totalOT += rec.otMinutes || 0;

        if(rec.isWeekend) extraDays++;
      });

      lateDeduction = totalLate * SETTINGS.lateDeductionPerMin;
      absenceDeduction = absent * SETTINGS.absentDeductionPerDay;
      halfDeduction = half * SETTINGS.halfDayDeduction;
      const totalDeductions = lateDeduction + absenceDeduction + halfDeduction;
      const otAddition = (totalOT / 60) * SETTINGS.otRatePerHour;
      const netSalary = emp.salary - totalDeductions + otAddition;

      document.getElementById('scheduledInOut').textContent = `${SETTINGS.shiftIn} / ${SETTINGS.shiftOut}`;
      document.getElementById('presentDays').textContent = present;
      document.getElementById('absentDays').textContent = absent;
      document.getElementById('halfDays').textContent = half;
      document.getElementById('totalLate').textContent = formatTime(totalLate * 60);
      document.getElementById('totalOT').textContent = formatTime(totalOT * 60);
      document.getElementById('extraDays').textContent = extraDays;

      document.getElementById('monthlySalary').textContent = `₹${emp.salary.toFixed(2)}`;
      document.getElementById('perDaySalary').textContent = `₹${perDay.toFixed(2)}`;
      document.getElementById('perHourSalary').textContent = `₹${perHour.toFixed(2)}`;
      document.getElementById('perMinSalary').textContent = `₹${perMin.toFixed(2)}`;

      document.getElementById('lateDeduction').textContent = `- ₹${lateDeduction.toFixed(2)}`;
      document.getElementById('absenceDeduction').textContent = `- ₹${absenceDeduction.toFixed(2)}`;
      document.getElementById('halfDayDeduction').textContent = `- ₹${halfDeduction.toFixed(2)}`;
      document.getElementById('extraAddition').textContent = `+ ₹${otAddition.toFixed(2)}`;

      document.getElementById('finalSalary').textContent = `₹${emp.salary.toFixed(2)}`;
      document.getElementById('totalDeductions').textContent = `- ₹${totalDeductions.toFixed(2)}`;
      document.getElementById('netPayable').textContent = `₹${netSalary.toFixed(2)}`;
    }

    function renderDailyAttendance(empAttendance) {
      const tbody = document.getElementById('dailyAttendanceTable');
      tbody.innerHTML = '';
      empAttendance.forEach(rec => {
        const date = new Date(rec.date);
        const dateStr = date.toLocaleDateString('en-US', { weekday: 'short', month: 'short', day: '2-digit' });
        const isWeekend = date.getDay() === 0 || date.getDay() === 6;
        const rowClass = isWeekend ? 'weekend' : rec.status === 'absent' ? 'absent' : '';
        const lateStr = rec.lateMinutes > 0 ? formatTime(rec.lateMinutes * 60) : '00:00';
        const otStr = rec.otMinutes > 0 ? formatTime(rec.otMinutes * 60) : '00:00';
        tbody.innerHTML += `
          <tr class="${rowClass}">
            <td>${dateStr}</td>
            <td>${rec.inTime || '--'}</td>
            <td>${rec.outTime || '--'}</td>
            <td class="${rec.lateMinutes > 0 ? 'late' : ''}">${lateStr}</td>
            <td class="${rec.otMinutes > 0 ? 'ot' : ''}">${otStr}</td>
            <td>${rec.status.charAt(0).toUpperCase() + rec.status.slice(1)}</td>
          </tr>
        `;
      });
    }

    // IMPORT
    function handleImportFile(e) {
      const file = e.target.files[0];
      if(!file) return;
      const reader = new FileReader();
      reader.onload = function(event) {
        const data = event.target.result;
        if(file.name.endsWith('.csv') || file.name.endsWith('.txt')) {
          parseCSV(data);
        } else {
          const workbook = XLSX.read(data, {type: 'binary'});
          const sheetName = workbook.SheetNames[0];
          const sheet = workbook.Sheets[sheetName];
          const json = XLSX.utils.sheet_to_json(sheet);
          showImportPreview(json);
        }
      };
      if(file.name.endsWith('.csv') || file.name.endsWith('.txt')) {
        reader.readAsText(file);
      } else {
        reader.readAsBinaryString(file);
      }
    }

    function parseCSV(data) {
      const lines = data.split('\n');
      const headers = lines[0].split(',').map(h => h.trim());
      const json = lines.slice(1).map(line => {
        const values = line.split(',').map(v => v.trim());
        const obj = {};
        headers.forEach((h, i) => obj[h] = values[i] || '');
        return obj;
      });
      showImportPreview(json);
    }

    function showImportPreview(json) {
      const preview = document.getElementById('importPreview');
      if(json.length === 0) {
        preview.innerHTML = '<p>No data found</p>';
        return;
      }
      let html = '<table class="attendance-table"><thead><tr>';
      Object.keys(json[0]).forEach(key => html += `<th>${key}</th>`);
      html += '</tr></thead><tbody>';
      json.slice(0, 10).forEach(row => {
        html += '<tr>';
        Object.values(row).forEach(val => html += `<td>${val}</td>`);
        html += '</tr>';
      });
      html += '</tbody></table>';
      preview.innerHTML = html;
      window.importData = json;
    }

    function processImport() {
      if(!window.importData) return alert('No data to import');
      let imported = 0;
      window.importData.forEach(row => {
        const empCode = row['Emp Code'] || row['Employee ID'] || row['Code'];
        const date = row['Date'] || row['Punch Date'];
        const inTime = row['In Time'] || row['Check In'];
        const outTime = row['Out Time'] || row['Check Out'];
        if(!empCode || !date) return;
        const emp = employees.find(e => e.code === empCode);
        if(!emp) return;
        const lateMinutes = calculateLateMinutes(inTime, SETTINGS.shiftIn);
        const otMinutes = calculateOTMinutes(outTime, SETTINGS.shiftOut);
        const status = row['Status'] || (lateMinutes > 0 ? 'late' : 'present');
        const isWeekend = new Date(date).getDay() === 0 || new Date(date).getDay() === 6;
        attendance.push({
          empId: emp.id,
          date: parseDate(date),
          inTime,
          outTime,
          status,
          lateMinutes,
          otMinutes,
          isWeekend
        });
        imported++;
      });
      saveData();
      alert(`Imported ${imported} records`);
    }

    // REPORTS
    function generateReport() {
      const month = document.getElementById('reportMonth').value;
      const year = document.getElementById('reportYear').value;
      const tbody = document.getElementById('reportTable').querySelector('tbody');
      tbody.innerHTML = '';
      employees.forEach(emp => {
        const empAttendance = attendance.filter(a => 
          a.empId === emp.id && 
          a.date.startsWith(`${year}-${month}`)
        );
        let present = 0, absent = 0, late = 0, ot = 0;
        let deductions = 0;
        empAttendance.forEach(rec => {
          if(rec.status === 'present') present++;
          if(rec.status === 'absent') absent++;
          if(rec.status === 'late') late++;
          ot += rec.otMinutes || 0;
          deductions += (rec.lateMinutes || 0) * SETTINGS.lateDeductionPerMin;
          deductions += (rec.status === 'absent') ? SETTINGS.absentDeductionPerDay : 0;
          deductions += (rec.status === 'half') ? SETTINGS.halfDayDeduction : 0;
        });
        const otAddition = (ot / 60) * SETTINGS.otRatePerHour;
        const netSalary = emp.salary - deductions + otAddition;
        tbody.innerHTML += `
          <tr>
            <td>${emp.name}</td>
            <td>${emp.code}</td>
            <td>${present}</td>
            <td>${absent}</td>
            <td>${late}</td>
            <td>${formatTime(ot * 60)}</td>
            <td>₹${deductions.toFixed(2)}</td>
            <td>₹${netSalary.toFixed(2)}</td>
          </tr>
        `;
      });
    }

    function exportReport() {
      const table = document.getElementById('reportTable');
      const wb = XLSX.utils.table_to_book(table, {sheet: "Salary Report"});
      const month = document.getElementById('reportMonth').value;
      const year = document.getElementById('reportYear').value;
      XLSX.writeFile(wb, `Salary_Report_${year}_${month}.xlsx`);
    }

    // UTILS
    function saveData() {
      localStorage.setItem('employees', JSON.stringify(employees));
      localStorage.setItem('attendance', JSON.stringify(attendance));
    }

    function calculateLateMinutes(inTime, shiftIn) {
      try {
        const [h1, m1, s1] = inTime.split(':').map(Number);
        const [h2, m2, s2] = shiftIn.split(':').map(Number);
        const inSec = h1 * 3600 + m1 * 60 + s1;
        const shiftSec = h2 * 3600 + m2 * 60 + s2;
        return Math.max(0, (inSec - shiftSec) / 60);
      } catch(e) {
        return 0;
      }
    }

    function calculateOTMinutes(outTime, shiftOut) {
      try {
        const [h1, m1, s1] = outTime.split(':').map(Number);
        const [h2, m2, s2] = shiftOut.split(':').map(Number);
        const outSec = h1 * 3600 + m1 * 60 + s1;
        const shiftSec = h2 * 3600 + m2 * 60 + s2;
        return Math.max(0, (outSec - shiftSec) / 60);
      } catch(e) {
        return 0;
      }
    }

    function formatTime(seconds) {
      const hours = Math.floor(seconds / 3600);
      const minutes = Math.floor((seconds % 3600) / 60);
      return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;
    }

    function parseDate(dateStr) {
      try {
        const date = new Date(dateStr);
        return date.toISOString().split('T')[0];
      } catch(e) {
        return dateStr;
      }
    }
  </script>
</body>
</html>
