<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>NFT Маркетплейс</title>
    <style>
        * { box-sizing: border-box; font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif; }
        body { background: #f5f7fb; margin: 0; padding: 20px; }
        .container { max-width: 500px; margin: 0 auto; background: white; border-radius: 32px; padding: 20px; box-shadow: 0 8px 20px rgba(0,0,0,0.05); }
        .balance-card { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; border-radius: 24px; padding: 20px; margin: 20px 0; text-align: center; }
        .balance-value { font-size: 2.5rem; font-weight: bold; }
        .nft-card { background: #f8f9ff; border-radius: 20px; padding: 16px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; }
        button { background: #1e293b; border: none; color: white; padding: 8px 18px; border-radius: 40px; font-weight: 600; cursor: pointer; }
        input { width: 100%; padding: 12px; margin: 8px 0; border-radius: 40px; border: 1px solid #ccc; font-size: 16px; }
        .request-item { background: #fff8e7; border-left: 4px solid #f59e0b; padding: 12px; margin-bottom: 8px; border-radius: 16px; display: flex; justify-content: space-between; align-items: center; }
        .error { color: red; margin-top: 8px; }
        h3 { margin-top: 24px; }
        .footer { font-size: 12px; color: #888; text-align: center; margin-top: 32px; }
    </style>
</head>
<body>
<div id="loginPage" class="container">
    <h2>🔐 Вход</h2>
    <input type="text" id="username" placeholder="Имя пользователя">
    <input type="password" id="password" placeholder="Пароль">
    <button onclick="login()">Войти</button>
    <div id="errorMsg" class="error"></div>
    <div class="footer">Данные из Google Sheets</div>
</div>

<div id="appPage" class="container" style="display:none;">
    <div class="balance-card">
        <div class="balance-value" id="balance">0</div>
        <div>⭐ Твои токены</div>
    </div>
    <h3>📦 Купить NFT</h3>
    <div id="nftList"></div>
    <h3>📬 Запросы на покупку</h3>
    <div id="requestsList"></div>
    <button onclick="logout()" style="margin-top: 20px; background: #64748b;">Выйти</button>
    <div class="footer">Покупатель отправляет запрос → владелец подтверждает → NFT переходит</div>
</div>

<script>
    const API_URL = 'https://script.google.com/macros/s/AKfycbwVUDCR5JbgAYUI3mqEP8Pe0g6ixcOH0IcCnZUlrY6G85KVS3o4fqPwtdvBc0LhMHXZ7g/exec';
    
    let currentUser = null;
    
    async function login() {
        const username = document.getElementById('username').value.trim();
        const password = document.getElementById('password').value;
        if (!username || !password) {
            document.getElementById('errorMsg').innerText = 'Введите имя и пароль';
            return;
        }
        const url = `${API_URL}?action=login&username=${encodeURIComponent(username)}&password=${encodeURIComponent(password)}`;
        try {
            const res = await fetch(url);
            const data = await res.json();
            if (data.success) {
                currentUser = data;
                document.getElementById('loginPage').style.display = 'none';
                document.getElementById('appPage').style.display = 'block';
                document.getElementById('balance').innerText = data.tokens;
                loadNfts();
                loadRequests();
            } else {
                document.getElementById('errorMsg').innerText = data.error;
            }
        } catch (err) {
            document.getElementById('errorMsg').innerText = 'Ошибка: ' + err.message;
        }
    }
    
    async function loadNfts() {
        try {
            const res = await fetch(`${API_URL}?action=getNfts`);
            const data = await res.json();
            if (data.success) {
                const available = data.nfts.filter(nft => nft.ownerId != currentUser.userId);
                const container = document.getElementById('nftList');
                if (available.length === 0) {
                    container.innerHTML = '<div style="text-align:center; padding:16px;">Нет доступных NFT</div>';
                    return;
                }
                container.innerHTML = available.map(nft => `
                    <div class="nft-card">
                        <div><strong>${nft.name}</strong><br><span style="font-size:0.8rem;">${nft.price} токенов</span></div>
                        <button onclick="buy(${nft.id}, ${nft.ownerId})">Купить</button>
                    </div>
                `).join('');
            }
        } catch (err) {
            console.error(err);
        }
    }
    
    async function buy(nftId, ownerId) {
        if (!confirm(`Отправить запрос на покупку NFT #${nftId}?`)) return;
        const url = `${API_URL}?action=createRequest&fromUserId=${currentUser.userId}&nftId=${nftId}&toUserId=${ownerId}`;
        const res = await fetch(url);
        const data = await res.json();
        if (data.success) {
            alert('✅ Запрос отправлен владельцу.');
        } else {
            alert('❌ Ошибка: ' + data.error);
        }
    }
    
    async function loadRequests() {
        try {
            const res = await fetch(`${API_URL}?action=getRequests&userId=${currentUser.userId}`);
            const data = await res.json();
            const container = document.getElementById('requestsList');
            if (data.success && data.requests && data.requests.length > 0) {
                container.innerHTML = data.requests.map(req => `
                    <div class="request-item">
                        <span>Пользователь ${req.fromUserId} хочет купить NFT #${req.nftId}</span>
                        <button onclick="approve(${req.id})" style="background:#10b981; padding:6px 12px;">✅ Принять</button>
                    </div>
                `).join('');
            } else {
                container.innerHTML = '<div style="text-align:center; padding:16px;">Нет входящих запросов</div>';
            }
        } catch (err) {
            console.error(err);
        }
    }
    
    async function approve(requestId) {
        const res = await fetch(`${API_URL}?action=approveRequest&requestId=${requestId}`);
        const data = await res.json();
        if (data.success) {
            alert('✅ Сделка подтверждена!');
            loadNfts();
            loadRequests();
            const userRes = await fetch(`${API_URL}?action=login&username=${currentUser.username}&password=${currentUser.password}`);
            const userData = await userRes.json();
            if (userData.success) {
                currentUser = userData;
                document.getElementById('balance').innerText = currentUser.tokens;
            }
        } else {
            alert('❌ Ошибка: ' + data.error);
        }
    }
    
    function logout() {
        currentUser = null;
        document.getElementById('loginPage').style.display = 'block';
        document.getElementById('appPage').style.display = 'none';
        document.getElementById('username').value = '';
        document.getElementById('password').value = '';
        document.getElementById('errorMsg').innerText = '';
    }
</script>
</body>
</html>
