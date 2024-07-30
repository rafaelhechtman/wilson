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
        h1, h2 {
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
        <a href="#" onclick="showPage('notes')">Anotações</a>
    </div>

    <div id="products">
        <h1>Lista de Balas e Doces</h1>
        <div class="product" data-name="Bala de Morango" data-price="0.50">
            Bala de Morango - R$ 0,50
            <div>
                <button onclick="decreaseQuantity(this)">-</button>
                <input type="number" value="0" min="0" onchange="manualQuantityChange(this)">
                <button onclick="increaseQuantity(this)">+</button>
            </div>
        </div>
        <div class="product" data-name="Chiclete" data-price="1.00">
            Chiclete - R$ 1,00
            <div>
                <button onclick="decreaseQuantity(this)">-</button>
                <input type="number" value="0" min="0" onchange="manualQuantityChange(this)">
                <button onclick="increaseQuantity(this)">+</button>
            </div>
        </div>
        <div class="product" data-name="Pirulito" data-price="0.75">
            Pirulito - R$ 0,75
            <div>
                <button onclick="decreaseQuantity(this)">-</button>
                <input type="number" value="0" min="0" onchange="manualQuantityChange(this)">
                <button onclick="increaseQuantity(this)">+</button>
            </div>
        </div>
        <div class="product" data-name="Ruffles" data-price="3.00">
            Ruffles - R$ 3,00
            <div>
                <button onclick="decreaseQuantity(this)">-</button>
                <input type="number" value="0" min="0" onchange="manualQuantityChange(this)">
                <button onclick="increaseQuantity(this)">+</button>
            </div>
        </div>
        <div class="product" data-name="Lays" data-price="2.50">
            Lays - R$ 2,50
            <div>
                <button onclick="decreaseQuantity(this)">-</button>
                <input type="number" value="0" min="0" onchange="manualQuantityChange(this)">
                <button onclick="increaseQuantity(this)">+</button>
            </div>
        </div>
        <!-- Formulário para adicionar mais produtos -->
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
                <select id="productsSelect" required>
                    <!-- Options will be populated by JavaScript -->
                </select>
                <input type="number" id="productQuantity" placeholder="Quantidade" min="1" value="1" required>
            </div>
            <button type="submit">Adicionar Anotação</button>
        </form>
        <ul id="notesList"></ul>
    </div>

    <script>
        let products = [
            {name: "Bala de Morango", price: 0.50},
            {name: "Chiclete", price: 1.00},
            {name: "Pirulito", price: 0.75},
            {name: "Ruffles", price: 3.00},
            {name: "Lays", price: 2.50}
        ];

        let sales = {};
        let notes = [];

        document.addEventListener("DOMContentLoaded", function() {
            loadSavedData();
            populateProductSelect();
        });

        function showPage(page) {
            document.getElementById('products').style.display = page === 'products' ? 'block' : 'none';
            document.getElementById('sales').style.display = page === 'sales' ? 'block' : 'none';
            document.getElementById('notes').style.display = page === 'notes' ? 'block' : 'none';
        }

        function increaseQuantity(button) {
            let quantityInput = button.previousElementSibling;
            quantityInput.value = parseInt(quantityInput.value) + 1;
        }

        function decreaseQuantity(button) {
            let quantityInput = button.nextElementSibling;
            let quantity = parseInt(quantityInput.value);
            if (quantity > 0) {
                quantityInput.value = quantity - 1;
                recordSale(button.parentElement.parentElement);
            }
        }

        function manualQuantityChange(input) {
            if (parseInt(input.value) < 0) {
                input.value = 0;
            }
        }

        function recordSale(productElement) {
            let name = productElement.getAttribute('data-name');
            let price = parseFloat(productElement.getAttribute('data-price'));
            sales[name] = (sales[name] || 0) + 1;
            updateSalesDisplay();

            let totalSales = document.getElementById('totalSales');
            let total = parseFloat(totalSales.textContent);
            total += price;
            totalSales.textContent = total.toFixed(2);
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
            let productList = document.getElementById('products');
            productList.innerHTML = '<h1>Lista de Balas e Doces</h1>';
            products.forEach(product => {
                let productElement = document.createElement('div');
                productElement.className = 'product';
                productElement.setAttribute('data-name', product.name);
                productElement.setAttribute('data-price', product.price.toFixed(2));
                productElement.innerHTML = `
                    ${product.name} - R$ ${product.price.toFixed(2)}
                    <div>
                        <button onclick="decreaseQuantity(this)">-</button>
                        <input type="number" value="0" min="0" onchange="manualQuantityChange(this)">
                        <button onclick="increaseQuantity(this)">+</button>
                    </div>
                `;
                productList.appendChild(productElement);
            });
            let addProductForm = `
                <h2>Adicionar Produto</h2>
                <form onsubmit="addProduct(); return false;">
                    <input type="text" id="newProductName" placeholder="Nome do Produto" required>
                    <input type="number" id="newProductPrice" step="0.01" placeholder="Preço" required>
                    <button type="submit">Adicionar</button>
                </form>
            `;
            productList.innerHTML += addProductForm;
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

        function addNote() {
            let customerName = document.getElementById('customerName').value;
            let date = document.getElementById('date').value;
            let product = document.getElementById('productsSelect').value;
            let quantity = parseInt(document.getElementById('productQuantity').value);
            let note = {customerName, date, product, quantity, status: 'pendente'};

            notes.push(note);
            updateNotesDisplay();
            saveData();
        }

        function updateNotesDisplay() {
            let notesList = document.getElementById('notesList');
            notesList.innerHTML = '';
            notes.forEach((note, index) => {
                let noteItem = document.createElement('li');
                noteItem.className = 'note';
                noteItem.innerHTML = `
                    ${note.customerName} - ${note.date} - ${note.product} (${note.quantity}) - R$ ${(getProductPrice(note.product) * note.quantity).toFixed(2)}
                    <button onclick="togglePaymentStatus(${index})">${note.status}</button>
                `;
                notesList.appendChild(noteItem);
            });
        }

        function togglePaymentStatus(index) {
            notes[index].status = notes[index].status === 'pendente' ? 'pago' : 'pendente';
            updateNotesDisplay();
            saveData();
        }

        function getProductPrice(productName) {
            let product = products.find(p => p.name === productName);
            return product ? product.price : 0;
        }

        function saveData() {
            localStorage.setItem('products', JSON.stringify(products));
            localStorage.setItem('sales', JSON.stringify(sales));
            localStorage.setItem('notes', JSON.stringify(notes));
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
            populateProductList();
            updateSalesDisplay();
            updateNotesDisplay();
        }
    </script>
</body>
</html>
