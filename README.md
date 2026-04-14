<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quản Lý Kho & Giá Cả</title>
    <style>
        :root { --bg-purple: #5c5cfc; --accent: #3498db; --danger: #e74c3c; --success: #27ae60; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; margin: 0; background-color: #f4f6f9; color: #333; }

        /* Header & Toolbar */
        .header { background-color: var(--bg-purple); padding: 25px; color: white; text-align: center; }
        .toolbar { background-color: var(--bg-purple); padding: 0 20px 30px; display: flex; gap: 12px; justify-content: center; }

        .btn-excel { 
            background: white; color: var(--bg-purple); border: 2px solid #000; 
            border-radius: 10px; padding: 10px 20px; font-weight: bold; cursor: pointer; 
            display: flex; align-items: center; gap: 8px; 
        }
        .btn-add-row { 
            background: transparent; color: white; border: 1px solid rgba(255,255,255,0.7); 
            border-radius: 10px; padding: 10px 20px; cursor: pointer;
        }

        /* Form Nhập liệu */
        .input-section {
            max-width: 1000px; margin: -20px auto 20px; background: white; 
            padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 15px;
        }
        .input-group { display: flex; flex-direction: column; gap: 5px; }
        .input-group label { font-size: 11px; font-weight: bold; color: #888; }
        .input-group input { padding: 10px; border: 1px solid #ddd; border-radius: 6px; outline: none; }

        .btn-save { 
            background: var(--bg-purple); color: white; border: none; 
            border-radius: 6px; cursor: pointer; font-weight: bold; height: 40px; align-self: flex-end;
        }

        /* Bảng hiển thị */
        .main-content { max-width: 1100px; margin: auto; padding: 0 20px; }
        .table-card { background: white; border-radius: 10px; overflow: hidden; box-shadow: 0 2px 10px rgba(0,0,0,0.05); }
        table { width: 100%; border-collapse: collapse; }
        th { background: #fafafa; color: #888; padding: 15px; text-align: left; font-size: 13px; border-bottom: 1px solid #eee; }
        td { padding: 15px; border-bottom: 1px solid #f2f2f2; font-size: 14px; }

        .price-text { color: var(--success); font-weight: bold; }
        .file-link { color: var(--accent); text-decoration: none; font-weight: 500; }
        
        /* Modal */
        #modal-confirm { 
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; 
            background: rgba(0,0,0,0.5); justify-content: center; align-items: center; z-index: 1000; 
        }
        .modal-content { background: white; padding: 30px; border-radius: 15px; width: 320px; text-align: center; }
        .modal-btns { margin-top: 20px; display: flex; gap: 10px; justify-content: center; }
        .btn-yes { background: var(--danger); color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; }
        .btn-no { background: #eee; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; }

        #file-input-hidden { display: none; }
    </style>
</head>
<body>

    <div class="header"><h2>QUẢN LÝ KHO VÀ ĐƠN GIÁ</h2></div>

    <div class="toolbar">
        <input type="file" id="file-input-hidden">
        <button class="btn-excel" onclick="document.getElementById('file-input-hidden').click()">
            <span>📁</span> Tải Tệp Excel
        </button>
        <button class="btn-add-row" onclick="document.getElementById('user-name').focus()">+ Thêm Dòng</button>
    </div>

    <div class="input-section">
        <div class="input-group">
            <label>HỌ VÀ TÊN</label>
            <input type="text" id="user-name" placeholder="Nguyễn Văn A">
        </div>
        <div class="input-group">
            <label>SẢN PHẨM</label>
            <input type="text" id="item-name" placeholder="Tên hàng...">
        </div>
        <div class="input-group">
            <label>SỐ LƯỢNG</label>
            <input type="number" id="item-qty" value="1" min="1">
        </div>
        <div class="input-group">
            <label>ĐƠN GIÁ (VNĐ)</label>
            <input type="number" id="item-price" placeholder="500000">
        </div>
        <button class="btn-save" onclick="saveToTable()">XÁC NHẬN</button>
    </div>

    <div class="main-content">
        <div class="table-card">
            <table>
                <thead>
                    <tr>
                        <th>STT</th>
                        <th>Người Thực Hiện</th>
                        <th>Sản Phẩm</th>
                        <th>Số Lượng</th>
                        <th>Đơn Giá</th>
                        <th>Chứng Từ</th>
                        <th style="text-align: center;">Hành Động</th>
                    </tr>
                </thead>
                <tbody id="inventory-data"></tbody>
            </table>
        </div>
    </div>

    <div id="modal-confirm">
        <div class="modal-content">
            <h3 style="color: var(--danger);">Xóa dòng này?</h3>
            <div class="modal-btns">
                <button class="btn-no" onclick="closeModal()">Hủy</button>
                <button class="btn-yes" id="btn-delete-confirm">Đồng ý</button>
            </div>
        </div>
    </div>

<script>
    const tableBody = document.getElementById('inventory-data');
    const modal = document.getElementById('modal-confirm');
    let rowToDelete = null;
    let currentFile = null;

    function updateSTT() {
        const rows = tableBody.querySelectorAll('tr');
        rows.forEach((row, index) => { row.cells[0].innerText = index + 1; });
    }

    // Định dạng tiền tệ VNĐ
    function formatMoney(num) {
        return new Intl.NumberFormat('vi-VN', { style: 'currency', currency: 'VND' }).format(num);
    }

    document.getElementById('file-input-hidden').onchange = (e) => {
        currentFile = e.target.files[0];
        if(currentFile) alert("Đã đính kèm file: " + currentFile.name);
    };

    function saveToTable() {
        const name = document.getElementById('user-name').value.trim();
        const item = document.getElementById('item-name').value.trim();
        const qty = document.getElementById('item-qty').value;
        const price = document.getElementById('item-price').value;

        if (!name || !item || !price) {
            alert("Vui lòng điền đủ Họ tên, Sản phẩm và Giá!");
            return;
        }

        const tr = document.createElement('tr');
        let fileHTML = '<span style="color:#ccc">Trống</span>';
        if (currentFile) {
            const url = URL.createObjectURL(currentFile);
            fileHTML = `<a href="${url}" target="_blank" class="file-link" download="${currentFile.name}">📄 Xem file</a>`;
        }

        tr.innerHTML = `
            <td></td>
            <td><strong>${name}</strong></td>
            <td>${item}</td>
            <td style="text-align:center">${qty}</td>
            <td class="price-text">${formatMoney(price)}</td>
            <td>${fileHTML}</td>
            <td style="text-align: center;">
                <button style="color:var(--accent); background:none; border:none; cursor:pointer;" onclick="editRow(this, ${price})">Sửa</button>
                <button style="color:var(--danger); background:none; border:none; cursor:pointer; margin-left:10px;" onclick="askDelete(this)">Xóa</button>
            </td>
        `;

        tableBody.prepend(tr);
        updateSTT();

        // Reset
        document.getElementById('user-name').value = '';
        document.getElementById('item-name').value = '';
        document.getElementById('item-qty').value = '1';
        document.getElementById('item-price').value = '';
        currentFile = null;
    }

    function editRow(btn, oldPrice) {
        const row = btn.closest('tr');
        const newPrice = prompt("Nhập giá mới (số):", oldPrice);
        if (newPrice) {
            row.querySelector('.price-text').innerText = formatMoney(newPrice);
        }
    }

    function askDelete(btn) {
        rowToDelete = btn.closest('tr');
        modal.style.display = 'flex';
    }

    function closeModal() { modal.style.display = 'none'; rowToDelete = null; }

    document.getElementById('btn-delete-confirm').onclick = () => {
        if (rowToDelete) { rowToDelete.remove(); updateSTT(); closeModal(); }
    };
</script>
</body>
</html>
