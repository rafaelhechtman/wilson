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
    </style>
</head>
<body>
    <div class="nav">
        <a href="#" onclick="showProducts()">Produtos</a> | 
        <a href="#" onclick="showSales()">Vendas</a>
    </div>
    <div id="products">
        <h1>Lista de Balas e Doces</h1>
        <div class="product" data-name="Bala de Morango" data-price="0.50">
            Bala de Morango - R$ 0,50
            <div>
                <button onclick="decreaseQuantity(this)">-</button>
                <span>0</span>
                <button onclick="increaseQuantity(this)">+</button>
            </div>
        </div>
        <div class="product" data-name="Chiclete" data-price="1.00">
            Chiclete - R$ 1,00
            <div>
                <button onclick="decreaseQuantity(this)">-</button>
                <span>0</span>
                <button onclick="increaseQuantity(this)">+</button>
            </div>
        </div>
        <div class="product" data-name="Pirulito" data-price="0.75">
            Pirulito - R$ 0,75
            <div>
                <button onclick="decreaseQuantity(this)">-</button>
                <span>0</span>
                <button onclick="increaseQuantity(this)">+</button>
            </div>
        </div>
    </div>
    <div id="sales" style="display: none;">
        <h2>Produtos Vendidos</h2>
        <ul id="salesList"></ul>
        <h3>Total: R$ <span id="totalSales">0.00</span></h3>
    </div>
    <script>
        function showProducts() {
            document.getElementById('products').style.display = 'block';
            document.getElementById('sales').style.display = 'none';
        }

        function showSales() {
            document.getElementById('products').style.display = 'none';
            document.getElementById('sales').style.display = 'block';
        }

        function increaseQuantity(button) {
            let quantitySpan = button.previousElementSibling;
            let quantity = parseInt(quantitySpan.textContent);
            quantitySpan.textContent = quantity + 1;
        }

        function decreaseQuantity(button) {
            let quantitySpan = button.nextElementSibling;
            let quantity = parseInt(quantitySpan.textContent);
            if (quantity > 0) {
                quantitySpan.textContent = quantity - 1;
                recordSale(button.parentElement.parentElement);
            }
        }

        function recordSale(productElement) {
            let name = productElement.getAttribute('data-name');
            let price = parseFloat(productElement.getAttribute('data-price'));
            let salesList = document.getElementById('salesList');
            let saleItem = document.createElement('li');
            saleItem.textContent = `${name} - R$ ${price.toFixed(2)}`;
            salesList.appendChild(saleItem);

            let totalSales = document.getElementById('totalSales');
            let total = parseFloat(totalSales.textContent);
            total += price;
            totalSales.textContent = total.toFixed(2);
        }
    </script>
</body>
</html>
