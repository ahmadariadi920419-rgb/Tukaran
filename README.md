<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pasar List Pro v3</title>
    <style>
        :root { 
            --primary: #2e7d32; 
            --highlight: #e8f5e9; 
            --danger: #d32f2f;
        }
        body { 
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; 
            background: #f0f2f5; 
            padding: 10px; 
            margin: 0;
        }
        .container { 
            max-width: 500px; 
            margin: auto; 
            background: white; 
            padding: 20px; 
            border-radius: 15px; 
            box-shadow: 0 4px 15px rgba(0,0,0,0.08); 
        }
        h2 { text-align: center; color: var(--primary); margin-bottom: 20px; }
        
        .form-group { 
            display: flex; 
            flex-direction: column; 
            gap: 10px; 
            margin-bottom: 20px; 
            padding-bottom: 20px; 
            border-bottom: 1px solid #eee; 
        }
        input, select, button { 
            padding: 14px; 
            border-radius: 10px; 
            border: 1px solid #ddd; 
            font-size: 16px; 
            outline: none;
        }
        input:focus { border-color: var(--primary); }
        
        button#add-btn { 
            background: var(--primary); 
            color: white; 
            font-weight: bold; 
            border: none;
            cursor: pointer;
        }
        
        .category-title { 
            background: #f8f9fa; 
            padding: 8px 12px; 
            border-radius: 6px; 
            font-size: 13px; 
            font-weight: bold; 
            margin-top: 20px; 
            color: #666;
            border-left: 4px solid var(--primary);
        }
        
        ul { list-style: none; padding: 0; margin: 0; }
        li { 
            display: flex; 
            align-items: center; 
            padding: 12px 5px; 
            border-bottom: 1px solid #f9f9f9; 
            gap: 10px; 
        }
        
        /* Gaya Baris Saat Dicentang */
        li.checked { 
            background-color: var(--highlight); 
            border-radius: 8px;
        }
        
        .checkbox {
            width: 22px;
            height: 22px;
            accent-color: var(--primary);
        }
        
        .item-text { 
            flex-grow: 1; 
            font-size: 16px; 
            color: #333;
        }
        
        .checked .item-text { 
            text-decoration: line-through; 
            color: #999; 
        }
        
        .btn-action { 
            background: none; 
            border: none; 
            font-size: 18px; 
            padding: 8px;
            cursor: pointer;
        }

        .clear-btn { 
            width: 100%; 
            margin-top: 30px; 
            background: #fff; 
            color: var(--danger); 
            border: 1px solid var(--danger); 
            padding: 12px; 
            border-radius: 10px; 
            font-weight: bold;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>🛒 List Belanja Pasar</h2>
    
    <div class="form-group">
        <input type="text" id="itemName" placeholder="Ketik nama barang...">
        <select id="itemCategory">
            <option value="🥦 Sayuran">🥦 Sayuran</option>
            <option value="🥩 Daging/Ikan">🥩 Daging/Ikan</option>
            <option value="🌶️ Bumbu">🌶️ Bumbu</option>
            <option value="🍚 Sembako">🍚 Sembako</option>
            <option value="📦 Lainnya">📦 Lainnya</option>
        </select>
        <button id="add-btn" onclick="addNewItem()">Tambah ke Daftar</button>
    </div>

    <div id="listContainer"></div>
    
    <button class="clear-btn" onclick="clearAll()">🗑️ Hapus Semua Daftar</button>
</div>

<script>
    let shoppingList = JSON.parse(localStorage.getItem('pasarDataV3')) || [];

    function addNewItem() {
        const nameInput = document.getElementById('itemName');
        const catSelect = document.getElementById('itemCategory');
        if (!nameInput.value.trim()) return;

        shoppingList.push({ 
            id: Date.now(), 
            name: nameInput.value, 
            category: catSelect.value, 
            checked: false 
        });
        saveAndRender();
        nameInput.value = "";
        nameInput.focus();
    }

    function toggleItem(id) {
        shoppingList = shoppingList.map(item => 
            item.id === id ? {...item, checked: !item.checked} : item
        );
        saveAndRender();
    }

    function editItem(id) {
        const item = shoppingList.find(i => i.id === id);
        const newName = prompt("Edit nama barang:", item.name);
        if (newName !== null && newName.trim() !== "") {
            item.name = newName;
            saveAndRender();
        }
    }

    function deleteItem(id) {
        if(confirm("Hapus barang ini?")) {
            shoppingList = shoppingList.filter(item => item.id !== id);
            saveAndRender();
        }
    }

    function clearAll() {
        if(confirm("Hapus seluruh daftar belanjaan?")) {
            shoppingList = [];
            saveAndRender();
        }
    }

    function saveAndRender() {
        localStorage.setItem('pasarDataV3', JSON.stringify(shoppingList));
        renderList();
    }

    function renderList() {
        const container = document.getElementById('listContainer');
        container.innerHTML = "";
        
        if (shoppingList.length === 0) {
            container.innerHTML = "<p style='text-align:center; color:#999; margin-top:20px;'>Belum ada daftar belanja.</p>";
            return;
        }

        const categories = [...new Set(shoppingList.map(item => item.category))];

        categories.forEach(cat => {
            const catWrapper = document.createElement('div');
            catWrapper.innerHTML = `<div class="category-title">${cat}</div>`;
            const ul = document.createElement('ul');
            
            shoppingList.filter(item => item.category === cat).forEach(item => {
                const li = document.createElement('li');
                if (item.checked) li.className = "checked";
                li.innerHTML = `
                    <input type="checkbox" class="checkbox" ${item.checked ? 'checked' : ''} onclick="toggleItem(${item.id})">
                    <span class="item-text" onclick="toggleItem(${item.id})">${item.name}</span>
                    <button class="btn-action" onclick="editItem(${item.id})" title="Edit">✏️</button>
                    <button class="btn-action" onclick="deleteItem(${item.id})" title="Hapus">❌</button>
                `;
                ul.appendChild(li);
            });
            catWrapper.appendChild(ul);
            container.appendChild(catWrapper);
        });
    }

    // Jalankan render pertama kali
    renderList();
</script>
</body>
</html>
