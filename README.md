# nhaplieuorder
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nhập Liệu Sản Phẩm Theo Cửa Hàng</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.5/xlsx.full.min.js"></script>
</head>
<body class="bg-gray-100 p-6">
    <div class="container mx-auto bg-white shadow-lg rounded-lg p-8">
        <h1 class="text-2xl font-bold mb-6 text-center">Nhập Liệu Sản Phẩm</h1>
        
        <form id="productForm" class="space-y-4">
            <div class="grid grid-cols-2 gap-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700">Địa Chỉ Cửa Hàng</label>
                    <select id="storeAddress" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50" readonly>
                        <option value="ChiNhanhA_DiaChi">Chi Nhánh A - 123 Đường ABC, Quận 1, TP.HCM</option>
                    </select>
                </div>
                
                <div>
                    <label class="block text-sm font-medium text-gray-700">Ngày Nhập Liệu</label>
                    <input type="date" id="entryDate" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50" required>
                </div>
            </div>

            <div id="productEntries" class="space-y-4">
                <div class="product-entry grid grid-cols-4 gap-4 items-center">
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Tên Sản Phẩm</label>
                        <select class="product-select mt-1 block w-full rounded-md border-gray-300 shadow-sm">
                            <option value="">Chọn Sản Phẩm</option>
                            <option value="SanPham1">Sản Phẩm 1</option>
                            <option value="SanPham2">Sản Phẩm 2</option>
                            <option value="SanPham3">Sản Phẩm 3</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Số Lượng</label>
                        <input type="number" class="quantity-input mt-1 block w-full rounded-md border-gray-300 shadow-sm" min="0">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Đơn Giá</label>
                        <input type="number" class="price-input mt-1 block w-full rounded-md border-gray-300 shadow-sm" min="0">
                    </div>
                    <div class="self-end">
                        <button type="button" class="remove-entry bg-red-500 text-white px-3 py-2 rounded hover:bg-red-600">Xóa</button>
                    </div>
                </div>
            </div>

            <div class="flex justify-between items-center mt-4">
                <button type="button" id="addProductEntry" class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600">
                    Thêm Sản Phẩm
                </button>
                <button type="submit" class="bg-blue-500 text-white px-6 py-2 rounded hover:bg-blue-600">
                    Lưu Dữ Liệu
                </button>
            </div>
        </form>

        <div id="dataTable" class="mt-8">
            <h2 class="text-xl font-semibold mb-4">Danh Sách Sản Phẩm Đã Nhập</h2>
            <table class="w-full border-collapse border border-gray-300">
                <thead>
                    <tr class="bg-gray-200">
                        <th class="border p-2">STT</th>
                        <th class="border p-2">Tên Sản Phẩm</th>
                        <th class="border p-2">Số Lượng</th>
                        <th class="border p-2">Đơn Giá</th>
                        <th class="border p-2">Thành Tiền</th>
                    </tr>
                </thead>
                <tbody id="productTableBody"></tbody>
            </table>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const productEntries = document.getElementById('productEntries');
            const addProductEntryBtn = document.getElementById('addProductEntry');
            const productForm = document.getElementById('productForm');
            const productTableBody = document.getElementById('productTableBody');

            // Thêm dòng sản phẩm mới
            addProductEntryBtn.addEventListener('click', function() {
                const newEntry = document.querySelector('.product-entry').cloneNode(true);
                newEntry.querySelectorAll('input').forEach(input => input.value = '');
                newEntry.querySelector('.product-select').selectedIndex = 0;
                productEntries.appendChild(newEntry);

                // Thêm sự kiện xóa cho dòng mới
                newEntry.querySelector('.remove-entry').addEventListener('click', function() {
                    if (productEntries.children.length > 1) {
                        productEntries.removeChild(newEntry);
                    }
                });
            });

            // Xử lý xóa dòng sản phẩm
            document.querySelector('.remove-entry').addEventListener('click', function() {
                if (productEntries.children.length > 1) {
                    productEntries.removeChild(this.closest('.product-entry'));
                }
            });

            // Lưu dữ liệu
            productForm.addEventListener('submit', function(e) {
                e.preventDefault();
                const storeAddress = document.getElementById('storeAddress').value;
                const entryDate = document.getElementById('entryDate').value;
                const products = [];
                let total = 0;

                productTableBody.innerHTML = ''; // Xóa bảng cũ

                document.querySelectorAll('.product-entry').forEach((entry, index) => {
                    const productSelect = entry.querySelector('.product-select');
                    const quantityInput = entry.querySelector('.quantity-input');
                    const priceInput = entry.querySelector('.price-input');

                    if (productSelect.value && quantityInput.value && priceInput.value) {
                        const productName = productSelect.options[productSelect.selectedIndex].text;
                        const quantity = parseInt(quantityInput.value);
                        const price = parseFloat(priceInput.value);
                        const lineTotal = quantity * price;
                        total += lineTotal;

                        products.push({
                            productName: productName,
                            quantity: quantity,
                            price: price,
                            lineTotal: lineTotal
                        });

                        // Thêm vào bảng
                        const row = productTableBody.insertRow();
                        row.innerHTML = `
                            <td class="border p-2">${index + 1}</td>
                            <td class="border p-2">${productName}</td>
                            <td class="border p-2">${quantity}</td>
                            <td class="border p-2">${price.toLocaleString()} VND</td>
                            <td class="border p-2">${lineTotal.toLocaleString()} VND</td>
                        `;
                    }
                });

                console.log('Dữ liệu:', {
                    storeAddress,
                    entryDate,
                    products,
                    total
                });

                // Ở đây bạn sẽ thêm logic xuất dữ liệu sang Google Sheets
                alert('Dữ liệu đã được lưu. Chức năng xuất Google Sheets sẽ được cập nhật.');
            });
        });
    </script>
</body>
</html>
