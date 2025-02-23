<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quản lý Công nhân</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f9f9f9;
        }

        header {
            background-color: #4CAF50;
            color: white;
            padding: 15px 20px;
            text-align: center;
        }

        .container {
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            background: white;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            border-radius: 10px;
        }

        h2 {
            color: #333;
        }

        .form-group {
            margin-bottom: 15px;
        }

        label {
            font-weight: bold;
            display: block;
            margin-bottom: 5px;
        }

        input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
            font-size: 16px;
        }

        .button-group {
            text-align: center;
            margin-top: 20px;
        }

        button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            font-size: 16px;
            margin: 5px;
            cursor: pointer;
        }

        button:hover {
            background-color: #45a049;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }

        table th, table td {
            padding: 10px;
            border: 1px solid #ddd;
            text-align: left;
        }

        table th {
            background-color: #4CAF50;
            color: white;
        }

        .action-btn {
            background-color: #f44336;
            padding: 5px 10px;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        .action-btn:hover {
            background-color: #d32f2f;
        }
    </style>
</head>
<body>
    <header>
        <h1>Quản lý Công nhân</h1>
    </header>

    <div class="container">
        <h2>Thêm Công nhân</h2>
        <div class="form-group">
            <label for="workerName">Tên Công nhân:</label>
            <input type="text" id="workerName" placeholder="Nhập tên công nhân">
        </div>
        <div class="form-group">
            <label for="workerDate">Ngày làm việc:</label>
            <input type="date" id="workerDate">
        </div>
        <div class="form-group">
            <label for="workerTimeIn">Giờ vào:</label>
            <input type="text" id="workerTimeIn" placeholder="Nhập giờ vào (YYYY-MM-DD HH:MM)">
        </div>
        <div class="form-group">
            <label for="workerTimeOut">Giờ ra:</label>
            <input type="text" id="workerTimeOut" placeholder="Nhập giờ ra (YYYY-MM-DD HH:MM)">
        </div>
        <div class="form-group">
            <label for="workerOvertime">Giờ tăng ca:</label>
            <input type="number" id="workerOvertime" placeholder="Nhập giờ tăng ca (số giờ)" step="0.01" min="0">
        </div>
        <div class="button-group">
            <button onclick="addWorker()">Thêm Công nhân</button>
            <button onclick="downloadFile()">Tải Danh Sách</button>
        </div>

        <h2>Danh sách Công nhân</h2>
        <table>
            <thead>
                <tr>
                    <th>STT</th>
                    <th>Tên Công nhân</th>
                    <th>Ngày làm việc</th>
                    <th>Giờ vào</th>
                    <th>Giờ ra</th>
                    <th>Giờ tăng ca</th>
                    <th>Giờ tăng ca thực tế</th>
                    <th>Hành động</th>
                </tr>
            </thead>
            <tbody id="workerTableBody"></tbody>
        </table>
        <h3>Tổng số ngày làm và tổng số giờ tăng ca trong tháng</h3>
        <div id="workerSummary"></div>
    </div>

    <script>
        let workers = JSON.parse(localStorage.getItem("workers")) || [];

        function calculateOvertime(timeIn, timeOut) {
            const workHours = 8; // Giờ làm việc tiêu chuẩn
            const inTime = new Date(timeIn);
            const outTime = new Date(timeOut);
            const diffHours = (outTime - inTime) / (1000 * 60 * 60); // Chuyển từ ms sang giờ
            return diffHours > workHours ? (diffHours - workHours).toFixed(2) : "0";
        }

        function displayWorkers() {
            const tableBody = document.getElementById("workerTableBody");
            tableBody.innerHTML = "";

            workers.sort((a, b) => {
                if (a.date !== b.date) {
                    return new Date(a.date) - new Date(b.date);
                }
                return a.name.localeCompare(b.name);
            });

            workers.forEach((worker, index) => {
                const overtime = calculateOvertime(worker.timeIn, worker.timeOut);
                const row = document.createElement("tr");
                row.innerHTML = `
                    <td>${index + 1}</td>
                    <td>${worker.name}</td>
                    <td>${worker.date}</td>
                    <td>${worker.timeIn}</td>
                    <td>${worker.timeOut}</td>
                    <td>${parseFloat(worker.overtime).toFixed(2)} giờ</td> <!-- Display overtime as decimal -->
                    <td>${parseFloat(overtime).toFixed(2)} giờ</td> <!-- Display calculated overtime as decimal -->
                    <td><button class="action-btn" onclick="deleteWorker(${index})">Xóa</button></td>
                `;
                tableBody.appendChild(row);
            });

            displayWorkerSummary();
        }

        function addWorker() {
            const name = document.getElementById("workerName").value;
            const date = document.getElementById("workerDate").value;
            const timeIn = document.getElementById("workerTimeIn").value;
            const timeOut = document.getElementById("workerTimeOut").value;
            const overtime = document.getElementById("workerOvertime").value;

            if (name && date && timeIn && timeOut && overtime !== "") {
                workers.push({ name, date, timeIn, timeOut, overtime: parseFloat(overtime) });
                localStorage.setItem("workers", JSON.stringify(workers));
                displayWorkers();

                document.getElementById("workerName").value = "";
                document.getElementById("workerDate").value = "";
                document.getElementById("workerTimeIn").value = "";
                document.getElementById("workerTimeOut").value = "";
                document.getElementById("workerOvertime").value = "";
            } else {
                alert("Vui lòng nhập đầy đủ thông tin!");
            }
        }

        function deleteWorker(index) {
            workers.splice(index, 1);
            localStorage.setItem("workers", JSON.stringify(workers));
            displayWorkers();
        }

        function displayWorkerSummary() {
            // Get the worker's total days and overtime in a given month
            const summary = {};

            workers.forEach(worker => {
                const workerKey = `${worker.name}-${worker.date.slice(0, 7)}`; // Use month and year as the key

                if (!summary[workerKey]) {
                    summary[workerKey] = { days: new Set(), overtime: 0 };
                }

                summary[workerKey].days.add(worker.date); // Add unique days
                summary[workerKey].overtime += parseFloat(worker.overtime); // Sum overtime
            });

            let summaryText = "";
            for (const workerKey in summary) {
                const [workerName, month] = workerKey.split("-");
                const { days, overtime } = summary[workerKey];
                const workdays = days.size;

                summaryText += `
                    <p><strong>${workerName} (${month}):</strong> ${workdays} ngày làm việc, ${overtime.toFixed(2)} giờ tăng ca.</p>
                `;
            }

            document.getElementById("workerSummary").innerHTML = summaryText;
        }

        function downloadFile() {
            if (workers.length === 0) {
                alert("Danh sách công nhân trống!");
                return;
            }

            let content = "| STT | Tên Công nhân     | Ngày làm việc | Giờ vào           | Giờ ra           | Giờ tăng ca  | Giờ tăng ca thực tế |\n";
            content += "|-----|-------------------|---------------|-------------------|-------------------|-------------|--------------------|\n";

            workers.forEach((worker, index) => {
                const overtime = calculateOvertime(worker.timeIn, worker.timeOut);
                content += `| ${String(index + 1).padEnd(3)} | ${worker.name.padEnd(17)} | ${worker.date.padEnd(13)} | ${worker.timeIn.padEnd(19)} | ${worker.timeOut.padEnd(19)} | ${parseFloat(worker.overtime).toFixed(2).padEnd(11)} | ${parseFloat(overtime).toFixed(2).padEnd(11)} |\n`;
            });

            const blob = new Blob([content], { type: 'text/plain' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = 'danhsach_congnhan.txt';
            link.click();
        }

        // Hiển thị danh sách công nhân khi trang tải
        displayWorkers();
    </script>
</body>
</html>
