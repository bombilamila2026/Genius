
<!DOCTYPE html>
<html lang="ru">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Мой магазин обуви</title>

    <style>
        * {
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            background: #f2f2f2;
            margin: 0;
            padding: 20px;
        }

        header {
            background: blueviolet;
            color: white;
            padding: 25px;
            text-align: center;
            border-radius: 12px;
            margin-bottom: 25px;
        }

        .admin-panel {
            max-width: 600px;
            margin: 0 auto 30px;
            background: white;
            padding: 25px;
            border: 2px solid blueviolet;
            border-radius: 12px;
        }

        h2 {
            color: blueviolet;
            text-align: center;
        }

        label {
            display: block;
            margin-top: 15px;
            font-weight: bold;
        }

        input, select, textarea {
            width: 100%;
            padding: 12px;
            margin-top: 6px;
            border: 1px solid #aaa;
            border-radius: 6px;
            font-size: 16px;
        }

        input:focus, textarea:focus, select:focus {
            outline: none;
            border: 2px solid blueviolet;
        }

        button {
            padding: 12px;
            border: none;
            border-radius: 6px;
            background: blueviolet;
            color: white;
            font-size: 15px;
            cursor: pointer;
        }

        button:hover {
            background: purple;
        }

        .save-button {
            width: 100%;
            margin-top: 20px;
        }

        .cancel-button {
            width: 100%;
            margin-top: 10px;
            background: gray;
        }

        .catalog {
            max-width: 1200px;
            margin: auto;
        }

        .catalog h2 {
            margin-bottom: 20px;
        }

        .products {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
            gap: 20px;
        }

        .product-card {
            background: white;
            padding: 15px;
            border-radius: 12px;
            box-shadow: 0 2px 8px #00000015;
        }

        .product-card img {
            width: 100%;
            height: 200px;
            object-fit: contain;
            background: #f7f7f7;
            border-radius: 8px;
        }

        .product-card h3 {
            margin-bottom: 8px;
        }

        .price {
            color: blueviolet;
            font-size: 22px;
            font-weight: bold;
        }

        .description {
            color: #555;
            line-height: 1.5;
        }

        .sizes {
            color: #444;
            font-size: 14px;
        }

        .actions {
            display: flex;
            gap: 8px;
            margin-top: 15px;
        }

        .actions button {
            flex: 1;
        }

        .delete-button {
            background: #d32f2f;
        }

        .delete-button:hover {
            background: #a50000;
        }

        .search {
            max-width: 600px;
            margin: 0 auto 25px;
        }

        .empty {
            text-align: center;
            color: #777;
            padding: 30px;
        }

        .hidden {
            display: none;
        }

        @media (max-width: 500px) {
            body {
                padding: 10px;
            }

            .products {
                grid-template-columns: repeat(2, minmax(0, 1fr));
                gap: 10px;
            }

            .product-card {
                padding: 10px;
            }

            .product-card img {
                height: 140px;
            }

            .actions {
                flex-direction: column;
            }
        }
    </style>
</head>

<body>

    <header>
        <h1>Мой магазин обуви</h1>
        <p>Каталог товаров</p>
    </header>

    <!-- ФОРМА ДОБАВЛЕНИЯ ТОВАРА -->

    <div class="admin-panel">

        <h2 id="formTitle">Добавить товар</h2>

        <form id="productForm">

            <label>Название товара:</label>
            <input type="text" id="productName"
                   placeholder="Например: Кроссовки Nike"
                   required>

            <label>Категория:</label>
            <select id="category" required>
                <option value="">Выберите категорию</option>
                <option>Мужская обувь</option>
                <option>Женская обувь</option>
                <option>Детская обувь</option>
                <option>Кроссовки</option>
                <option>Ботинки</option>
                <option>Туфли</option>
                <option>Одежда</option>
                <option>Другое</option>
            </select>

            <label>Цена (грн):</label>
            <input type="number" id="productPrice"
                   min="0" step="0.01"
                   placeholder="1899"
                   required>

            <label>Размеры:</label>
            <input type="text" id="productSizes"
                   placeholder="Например: 40, 41, 42, 43">

            <label>Описание товара:</label>
            <textarea id="productDescription"
                      rows="4"
                      placeholder="Опишите товар..."></textarea>

            <label>Фотография товара:</label>
            <input type="file" id="productImage"
                   accept="image/*">

            <label>Или ссылка на фотографию:</label>
            <input type="url" id="imageUrl"
                   placeholder="https://example.com/photo.jpg">

            <button type="submit" class="save-button"
                    id="saveButton">
                Добавить товар
            </button>

            <button type="button"
                    class="cancel-button hidden"
                    id="cancelButton">
                Отменить редактирование
            </button>

        </form>

    </div>

    <!-- ПОИСК ТОВАРОВ -->

    <div class="search">
        <input type="search" id="searchInput"
               placeholder="Поиск товара по названию...">
    </div>

    <!-- КАТАЛОГ -->

    <main class="catalog">

        <h2>Наши товары</h2>

        <div class="products" id="products">

        </div>

        <p class="empty hidden" id="emptyMessage">
            Товаров пока нет. Добавьте первый товар!
        </p>

    </main>

    <script>

        // БАЗА ДАННЫХ ТОВАРОВ В БРАУЗЕРЕ

        let db;

        const request = indexedDB.open("ShoeStore", 1);

        request.onupgradeneeded = function(event) {
            db = event.target.result;

            if (!db.objectStoreNames.contains("products")) {
                db.createObjectStore("products", {
                    keyPath: "id",
                    autoIncrement: true
                });
            }
        };

        request.onsuccess = function(event) {
            db = event.target.result;
            loadProducts();
        };

        request.onerror = function() {
            alert("Ошибка открытия базы данных!");
        };


        // СОХРАНЕНИЕ ТОВАРА

        const form = document.getElementById("productForm");

        let editingId = null;

        form.addEventListener("submit", async function(event) {

            event.preventDefault();

            const name = document.getElementById("productName").value.trim();
            const category = document.getElementById("category").value;
            const price = Number(document.getElementById("productPrice").value);
            const sizes = document.getElementById("productSizes").value;
            const description = document.getElementById("productDescription").value;

            if (!name || !category || price < 0 || !Number.isFinite(price)) {
                alert("Проверьте заполнение полей!");
                return;
            }

            let image = document.getElementById("imageUrl").value.trim();

            const file = document.getElementById("productImage").files[0];

            if (file) {

                if (!file.type.startsWith("image/")) {
                    alert("Выберите файл изображения!");
                    return;
                }

                if (file.size > 3 * 1024 * 1024) {
                    alert("Размер фотографии не должен превышать 3 МБ!");
                    return;
                }

                image = await new Promise((resolve, reject) => {

                    const reader = new FileReader();

                    reader.onload = () => resolve(reader.result);
                    reader.onerror = reject;

                    reader.readAsDataURL(file);

                });
            }

            if (editingId !== null && !image) {
                const oldProduct = await getProduct(editingId);
                image = oldProduct?.image || "";
            }

            const product = {
                name,
                category,
                price,
                sizes,
                description,
                image
            };

            if (editingId !== null) {

                product.id = editingId;

                const transaction = db.transaction(["products"], "readwrite");

                transaction.objectStore("products").put(product);

                transaction.oncomplete = function() {
                    resetForm();
                    loadProducts();
                    alert("Товар успешно изменён!");
                };

            } else {

                const transaction = db.transaction(["products"], "readwrite");

                transaction.objectStore("products").add(product);

                transaction.oncomplete = function() {
                    resetForm();
                    loadProducts();
                    alert("Товар успешно добавлен!");
                };

            }

        });


        // ПОЛУЧЕНИЕ ОДНОГО ТОВАРА

        function getProduct(id) {

            return new Promise((resolve, reject) => {

                const transaction = db.transaction(["products"], "readonly");

                const request = transaction.objectStore("products").get(id);

                request.onsuccess = () => resolve(request.result);
                request.onerror = () => reject(request.error);

            });

        }


        // ЗАГРУЗКА ВСЕХ ТОВАРОВ

        function loadProducts() {

            const transaction = db.transaction(["products"], "readonly");

            const request = transaction.objectStore("products").getAll();

            request.onsuccess = function() {

                const products = request.result;

                displayProducts(products);

            };

        }


        // ОТОБРАЖЕНИЕ КАРТОЧЕК

        function displayProducts(products) {

            const container = document.getElementById("products");

            container.innerHTML = "";

            document.getElementById("emptyMessage").classList.toggle(
                "hidden", products.length > 0
            );

            products.forEach(product => {

                const card = document.createElement("div");

                card.className = "product-card";

                const img = document.createElement("img");

                img.src = product.image || "https://placehold.co/400x300?text=No+Photo";
                img.alt = product.name;

                img.onerror = function() {
                    this.src = "https://placehold.co/400x300?text=No+Photo";
                };

                const title = document.createElement("h3");
                title.textContent = product.name;

                const category = document.createElement("p");
                category.textContent = product.category;

                const price = document.createElement("p");
                price.className = "price";
                price.textContent = product.price + " грн";

                const sizes = document.createElement("p");
                sizes.className = "sizes";
                sizes.textContent = "Размеры: " + (product.sizes || "Не указаны");

                const description = document.createElement("p");
                description.className = "description";
                description.textContent = product.description;

                const actions = document.createElement("div");
                actions.className = "actions";

                const editButton = document.createElement("button");
                editButton.textContent = "Изменить";
                editButton.onclick = () => editProduct(product.id);

                const deleteButton = document.createElement("button");
                deleteButton.textContent = "Удалить";
                deleteButton.className = "delete-button";
                deleteButton.onclick = () => deleteProduct(product.id);

                actions.append(editButton, deleteButton);

                card.append(
                    img,
                    title,
                    category,
                    price,
                    sizes,
                    description,
                    actions
                );

                container.appendChild(card);

            });

            filterProducts();

        }


        // РЕДАКТИРОВАНИЕ ТОВАРА

        async function editProduct(id) {

            const product = await getProduct(id);

            if (!product) return;

            editingId = id;

            document.getElementById("productName").value = product.name;
            document.getElementById("category").value = product.category;
            document.getElementById("productPrice").value = product.price;
            document.getElementById("productSizes").value = product.sizes;
            document.getElementById("productDescription").value = product.description;

            document.getElementById("imageUrl").value =
                product.image && product.image.startsWith("http")
                    ? product.image : "";

            document.getElementById("formTitle").textContent = "Редактировать товар";

            document.getElementById("saveButton").textContent = "Сохранить изменения";

            document.getElementById("cancelButton").classList.remove("hidden");

            document.getElementById("productForm").scrollIntoView({
                behavior: "smooth"
            });

        }


        // УДАЛЕНИЕ ТОВАРА

        function deleteProduct(id) {

            if (!confirm("Вы действительно хотите удалить этот товар?")) {
                return;
            }

            const transaction = db.transaction(["products"], "readwrite");

            transaction.objectStore("products").delete(id);

            transaction.oncomplete = function() {
                loadProducts();
            };

        }


        // ОЧИСТКА ФОРМЫ

        function resetForm() {

            form.reset();

            editingId = null;

            document.getElementById("formTitle").textContent = "Добавить товар";

            document.getElementById("saveButton").textContent = "Добавить товар";

            document.getElementById("cancelButton").classList.add("hidden");

        }

        document.getElementById("cancelButton").onclick = resetForm;


        // ПОИСК ТОВАРОВ

        document.getElementById("searchInput").addEventListener(
            "input", filterProducts
        );

        function filterProducts() {

            const search = document.getElementById("searchInput")
                .value.toLowerCase();

            const cards = document.querySelectorAll(".product-card");

            let visible = 0;

            cards.forEach(card => {

                const matches = card.textContent.toLowerCase().includes(search);

                card.style.display = matches ? "" : "none";

                if (matches) visible++;

            });

            document.getElementById("emptyMessage").classList.toggle(
                "hidden", visible > 0
            );

        }

    </script>

</body>
</html>

