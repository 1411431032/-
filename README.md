<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>我的投資報酬率計算器</title>
    <style>
        body { font-family: sans-serif; max-width: 600px; margin: 40px auto; padding: 20px; background: #f4f7f6; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h2 { color: #2c3e50; }
        table { width: 100%; border-collapse: collapse; margin-bottom: 20px; }
        th, td { border-bottom: 1px solid #ddd; padding: 10px; text-align: left; }
        input { padding: 8px; border: 1px solid #ccc; border-radius: 4px; width: 90%; }
        button { padding: 10px 15px; cursor: pointer; border: none; border-radius: 4px; transition: 0.3s; }
        .add-btn { background: #27ae60; color: white; }
        .calc-btn { background: #2980b9; color: white; width: 100%; font-size: 1.1em; }
        .remove-btn { background: #e74c3c; color: white; }
        #result { margin-top: 20px; font-size: 1.2em; font-weight: bold; color: #2c3e50; text-align: center; }
        .hint { font-size: 0.85em; color: #7f8c8d; margin-bottom: 15px; }
    </style>
</head>
<body>

<div class="card">
    <h2>XIRR 投資報酬率計算器</h2>
    <p class="hint">※ 投入資金請填「負數」(如 -10000)，目前市值或領回請填「正數」。</p>
    
    <table id="cashflowTable">
        <thead>
            <tr>
                <th>日期</th>
                <th>金額 (正/負)</th>
                <th>操作</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><input type="date" class="date-input" value="2024-01-01"></td>
                <td><input type="number" class="amount-input" value="-100000"></td>
                <td></td>
            </tr>
            <tr>
                <td><input type="date" class="date-input" value="2025-01-30"></td>
                <td><input type="number" class="amount-input" value="120000"></td>
                <td></td>
            </tr>
        </tbody>
    </table>

    <button class="add-btn" onclick="addRow()">+ 新增一筆</button>
    <hr>
    <button class="calc-btn" onclick="calculateXIRR()">開始計算年化報酬率</button>

    <div id="result"></div>
</div>

<script>
    function addRow() {
        const table = document.getElementById('cashflowTable').getElementsByTagName('tbody')[0];
        const newRow = table.insertRow();
        newRow.innerHTML = `
            <td><input type="date" class="date-input"></td>
            <td><input type="number" class="amount-input"></td>
            <td><button class="remove-btn" onclick="this.parentElement.parentElement.remove()">移除</button></td>
        `;
    }

    // XIRR 核心邏輯 (牛頓法)
    function calculateXIRR() {
        const dates = document.querySelectorAll('.date-input');
        const amounts = document.querySelectorAll('.amount-input');
        let flows = [];

        for (let i = 0; i < dates.length; i++) {
            if (dates[i].value && amounts[i].value) {
                flows.push({
                    date: new Date(dates[i].value),
                    amount: parseFloat(amounts[i].value)
                });
            }
        }

        if (flows.length < 2) {
            alert("請至少輸入兩筆資料（一正一負）。");
            return;
        }

        const firstDate = flows[0].date;
        const data = flows.map(f => ({
            amount: f.amount,
            t: (f.date - firstDate) / (1000 * 60 * 60 * 24 * 365) // 換算成天數佔一年的比例
        }));

        let rate = 0.1; // 初始猜測值
        for (let i = 0; i < 100; i++) {
            let npv = 0;
            let derivative = 0;
            for (let f of data) {
                let factor = Math.pow(1 + rate, f.t);
                npv += f.amount / factor;
                derivative -= f.t * f.amount / Math.pow(1 + rate, f.t + 1);
            }
            
            let newRate = rate - npv / derivative;
            if (Math.abs(newRate - rate) < 0.0001) {
                document.getElementById('result').innerText = `年化報酬率 (XIRR): ${(newRate * 100).toFixed(2)}%`;
                return;
            }
            rate = newRate;
        }
        document.getElementById('result').innerText = "計算失敗，請檢查數字是否正確。";
    }
</script>

</body>
</html>
