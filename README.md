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
        .product {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        .product button {
            margin: 0 5px;
        }
        .nav {
            margin-bottom: 20px;
        }
        .nav a {
            margin-right: 10px;
        }
        input[type="number"] {
            width: 50px;
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
        <!-- Adicione mais produtos conforme necessário -->
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
                <input type="text" id="products" required>
            </div>
            <div>
                <label for="amount">Valor:</label>
                <input type="number" id="amount" step="0.01" required>
            </div>
            <button type="submit">Adicionar Anotação</button>
        </form>
        <ul id="notesList"></ul>
    </div>

    <script>
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
            let salesList = document.getElementById('salesList');
            let existingItem = salesList.querySelector(`li[data-name="${name}"]`);
            if (existingItem) {
                let countSpan = existingItem.querySelector('.count');
                countSpan.textContent = parseInt(countSpan.textContent) + 1;
            } else {
                let saleItem = document.createElement('li');
                saleItem.setAttribute('data-name', name);
                saleItem.innerHTML = `${name} (<span class="count">1</span>)`;
                salesList.appendChild(saleItem);
            }

            let totalSales = document.getElementById('totalSales');
            let total = parseFloat(totalSales.textContent);
            total += price;
            totalSales.textContent = total.toFixed(2);
        }

        function addNote() {
            let customerName = document.getElementById('customerName').value;
            let date = document.getElementById('date').value;
            let products = document.getElementById('products').value;
            let amount = parseFloat(document.getElementById('amount').value).toFixed(2);

            let notesList = document.getElementById('notesList');
            let noteItem = document.createElement('li');
            noteItem.textContent = `${customerName} - ${date} - ${products} - R$ ${amount}`;
            notesList.appendChild(noteItem);

            document.getElementById('customerName').value = '';
            document.getElementById('date').value = '';
            document.getElementById('products').value = '';
            document.getElementById('amount').value = '';
        }
    </script>
</body>
</html>
