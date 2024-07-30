<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Auxiliador de Vendedores de Bala</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        h1, h2, h3 {
            color: #333;
        }
        .product, .note {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        .product button, .note button {
            margin: 0 5px;
        }
        .nav {
            margin-bottom: 20px;
        }
        .nav a {
            margin-right: 10px;
        }
        input[type="number"], input[type="text"], input[type="date"] {
            width: 150px;
        }
        input[type="text"], input[type="date"] {
            width: 200px;
        }
    </style>
</head>
<body>
    <div class="nav">
        <a href="#" onclick="showPage('products')">Produtos</a> | 
        <a href="#" onclick="showPage('sales')">Vendas</a> | 
        <a href="#" onclick="showPage('notes')">Anotações</a> | 
        <a href="#" onclick="showPage('report')">Relatório</a>
    </div>

    <div id="products">
        <h1>Lista de Balas e Doces</h1>
        <div id="productList"></div>

        <h2>Adicionar Produto</h2>
        <form onsubmit="addProduct(); return false;">
            <input type="text" id="newProductName" placeholder="Nome do Produto" required>
            <input type="number" id="newProductPrice" step="0.01" placeholder="Preço" required>
            <button type="submit">Adicionar</button>
        </form>
    </div>

    <div id="sales" style="display: none;">
        <h2>Produtos Vendidos</h2>
        <ul id="salesList"></ul>
        <h3>Total: R$ <span id="totalSales">0.00</span></h3>
    </div>

    <div id="notes" style="display: none;">
        <h2>Anotações de Pagamentos Pendentes</h2>
        <form onsubmit="addNote(); return false;">
            <div>
                <label for="customerName">Nome do Cliente:</label>
                <input type="text" id="customerName" required>
            </div>
            <div>
                <label for="date">Data:</label>
                <input type="date" id="date" required>
            </div>
            <div>
                <label for="products">Produtos:</label>
                <select id="productsSelect" required></select>
                <input type="number" id="productQuantity" placeholder="Quantidade" min="1" value="1" required>
                <button type="button" onclick="addProductToNote()">Adicionar Produto</button>
            </div>
            <ul id="noteProductsList"></ul>
            <button type="submit">Adicionar Anotação</button>
        </form>
        <ul id="notesList"></ul>
    </div>

    <div id="report" style="display: none;">
        <h2>Relatório da Loja</h2>
        <ul id="reportList"></ul>
    </div>

    <script>
        let products = [];
        let sales = {};
        let notes = [];
        let dailySales = {};

        document.addEventListener("DOMContentLoaded", function() {
            loadSavedData();
            populateProductSelect();
            populateProductList();
            updateSalesDisplay();
            updateNotesDisplay();
            updateReport();
        });

        function showPage(page) {
            document.getElementById('products').style.display = page === 'products' ? 'block' : 'none';
            document.getElementById('sales').style.display = page === 'sales' ? 'block' : 'none';
            document.getElementById('notes').style.display = page === 'notes' ? 'block' : 'none';
            document.getElementById('report').style.display = page === 'report' ? 'block' : 'none';
            if (page === 'report') updateReport();
        }

        function increaseQuantity(button) {
            let quantityInput = button.previousElementSibling;
            quantityInput.value = parseInt(quantityInput.value) + 1;
            recordSale(button.parentElement.parentElement);
        }

        function decreaseQuantity(button) {
            let quantityInput = button.nextElementSibling;
            let quantity = parseInt(quantityInput.value);
            if (quantity > 0) {
                quantityInput.value = quantity - 1;
                recordSale(button.parentElement.parentElement, true);
            }
        }

        function manualQuantityChange(input) {
            if (parseInt(input.value) < 0) {
                input.value = 0;
            }
        }

        function recordSale(productElement, isDecrease = false) {
            let name = productElement.getAttribute('data-name');
            let price = parseFloat(productElement.getAttribute('data-price'));
            sales[name] = (sales[name] || 0) + (isDecrease ? -1 : 1);

            let date = new Date().toISOString().split('T')[0];
            dailySales[date] = dailySales[date] || { total: 0, products: {}, payments: { paid: [], pending: [] } };
            dailySales[date].total += (isDecrease ? -price : price);
            dailySales[date].products[name] = (dailySales[date].products[name] || 0) + (isDecrease ? -1 : 1);

            updateSalesDisplay();
            saveData();
        }

        function updateSalesDisplay() {
            let salesList = document.getElementById('salesList');
            salesList.innerHTML = '';
            for (let product in sales) {
                let listItem = document.createElement('li');
                listItem.textContent = `${product} (${sales[product]})`;
                salesList.appendChild(listItem);
            }

            let totalSales = document.getElementById('totalSales');
            let total = Object.keys(dailySales).reduce((sum, date) => sum + dailySales[date].total, 0);
            totalSales.textContent = total.toFixed(2);
        }

        function addProduct() {
            let name = document.getElementById('newProductName').value;
            let price = parseFloat(document.getElementById('newProductPrice').value);
            products.push({name: name, price: price});
            populateProductList();
            populateProductSelect();
            saveData();
        }

        function populateProductList() {
            let productList = document.getElementById('productList');
            productList.innerHTML = '';
            products.forEach(product => {
                let productElement = document.createElement('div');
                productElement.className = 'product';
                productElement.setAttribute('data-name', product.name);
                productElement.setAttribute('data-price', product.price.toFixed(2));
                productElement.innerHTML = `
                    ${product.name} - R$ ${product.price.toFixed(2)}
                    <div>
                        <button onclick="decreaseQuantity(this)">Vendido -</button>
                        <input type="number" value="0" min="0" onchange="manualQuantityChange(this)">
                        <button onclick="increaseQuantity(this)">Adicionado +</button>
                    </div>
                `;
                productList.appendChild(productElement);
            });
        }

        function populateProductSelect() {
            let productSelect = document.getElementById('productsSelect');
            productSelect.innerHTML = '';
            products.forEach(product => {
                let option = document.createElement('option');
                option.value = product.name;
                option.textContent = `${product.name} - R$ ${product.price.toFixed(2)}`;
                productSelect.appendChild(option);
            });
        }

        let noteProducts = [];

        function addProductToNote() {
            let productSelect = document.getElementById('productsSelect');
            let productName = productSelect.value;
            let quantity = parseInt(document.getElementById('productQuantity').value);

            noteProducts.push({ productName, quantity });
            updateNoteProductsList();
        }

        function updateNoteProductsList() {
            let noteProductsList = document.getElementById('noteProductsList');
            noteProductsList.innerHTML = '';
            noteProducts.forEach((item, index) => {
                let listItem = document.createElement('li');
                listItem.textContent = `${item.productName} - ${item.quantity}`;
                listItem.innerHTML += ` <button onclick="removeProductFromNote(${index})">Remover</button>`;
                noteProductsList.appendChild(listItem);
            });
        }

        function removeProductFromNote(index) {
            noteProducts.splice(index, 1);
            updateNoteProductsList();
        }

        function addNote() {
            let customerName = document.getElementById('customerName').value;
            let date = document.getElementById('date').value;
            let note = { customerName, date, products: noteProducts, status: 'pendente' };

            notes.push(note);
            dailySales[date] = dailySales[date] || { total: 0, products: {}, payments: { paid: [], pending: [] } };
            dailySales[date].payments.pending.push(customerName);
            noteProducts = [];
            updateNotesDisplay();
            updateNoteProductsList();
            saveData();
        }

        function updateNotesDisplay() {
            let notesList = document.getElementById('notesList');
            notesList.innerHTML = '';
            notes.forEach((note, index) => {
                let productsList = note.products.map(item => `${item.productName} (${item.quantity})`).join(', ');
                let noteItem = document.createElement('li');
                noteItem.className = 'note';
                noteItem.innerHTML = `
                    ${note.customerName} - ${note.date} - ${productsList} - 
                    <button onclick="togglePaymentStatus(${index})">${note.status}</button>
                `;
                notesList.appendChild(noteItem);
            });
        }

        function togglePaymentStatus(index) {
            let note = notes[index];
            note.status = note.status === 'pendente' ? 'pago' : 'pendente';
            let dailyPaymentList = dailySales[note.date].payments;

            if (note.status === 'pago') {
                dailyPaymentList.pending = dailyPaymentList.pending.filter(name => name !== note.customerName);
                dailyPaymentList.paid.push(note.customerName);
            } else {
                dailyPaymentList.paid = dailyPaymentList.paid.filter(name => name !== note.customerName);
                dailyPaymentList.pending.push(note.customerName);
            }

            updateNotesDisplay();
            updateReport();
            saveData();
        }

        function updateReport() {
            let reportList = document.getElementById('reportList');
            reportList.innerHTML = '';
            for (let date in dailySales) {
                let dateItem = document.createElement('li');
                let dailyTotal = dailySales[date].total.toFixed(2);
                let productsSold = Object.keys(dailySales[date].products).map(product => `${product} (${dailySales[date].products[product]})`).join(', ');
                let paymentsPaid = dailySales[date].payments.paid.join(', ');
                let paymentsPending = dailySales[date].payments.pending.join(', ');

                dateItem.innerHTML = `
                    <h3>${date}</h3>
                    <p>Total: R$ ${dailyTotal}</p>
                    <p>Produtos Vendidos: ${productsSold}</p>
                    <p>Pagamentos Realizados: ${paymentsPaid}</p>
                    <p>Pagamentos Pendentes: ${paymentsPending}</p>
                `;
                reportList.appendChild(dateItem);
            }
        }

        function saveData() {
            localStorage.setItem('products', JSON.stringify(products));
            localStorage.setItem('sales', JSON.stringify(sales));
            localStorage.setItem('notes', JSON.stringify(notes));
            localStorage.setItem('dailySales', JSON.stringify(dailySales));
        }

        function loadSavedData() {
            if (localStorage.getItem('products')) {
                products = JSON.parse(localStorage.getItem('products'));
            }
            if (localStorage.getItem('sales')) {
                sales = JSON.parse(localStorage.getItem('sales'));
            }
            if (localStorage.getItem('notes')) {
                notes = JSON.parse(localStorage.getItem('notes'));
            }
            if (localStorage.getItem('dailySales')) {
                dailySales = JSON.parse(localStorage.getItem('dailySales'));
            }
        }
    </script>
</body>
</html>
