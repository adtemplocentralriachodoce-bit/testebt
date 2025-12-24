<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Login – Celebração</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- Estilo simples e oficial -->
  <style>
    * {
      box-sizing: border-box;
      font-family: Arial, Helvetica, sans-serif;
    }

    body {
      background: #f2f4f7;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
    }

    .login-box {
      background: #fff;
      padding: 30px;
      width: 100%;
      max-width: 360px;
      border-radius: 8px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    }

    .login-box h1 {
      text-align: center;
      margin-bottom: 20px;
      color: #333;
    }

    label {
      display: block;
      margin-top: 12px;
      font-size: 14px;
      color: #555;
    }

    input {
      width: 100%;
      padding: 10px;
      margin-top: 6px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 15px;
    }

    button {
      width: 100%;
      margin-top: 20px;
      padding: 12px;
      background: #1877f2;
      border: none;
      color: #fff;
      font-size: 16px;
      border-radius: 4px;
      cursor: pointer;
    }

    button:hover {
      background: #145dbf;
    }

    .msg {
      margin-top: 15px;
      text-align: center;
      font-size: 14px;
      color: red;
    }
  </style>
</head>
<body>

  <div class="login-box">
    <h1>Celebração</h1>

    <label>Usuário</label>
    <input type="text" id="user" placeholder="admin" />

    <label>Senha</label>
    <input type="password" id="password" placeholder="******" />

    <button onclick="login()">Entrar</button>

    <div class="msg" id="msg"></div>
  </div>

  <!-- Firebase SDK -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
    import { getFirestore, doc, getDoc, updateDoc } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

    // 🔴 OBRIGATÓRIO: CONFIG DO SEU PROJETO
    const firebaseConfig = {
      apiKey: "COLE_AQUI",
      authDomain: "adlogin.firebaseapp.com",
      projectId: "adlogin",
      storageBucket: "adlogin.appspot.com",
      messagingSenderId: "COLE_AQUI",
      appId: "COLE_AQUI"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    window.login = async function () {
      const user = document.getElementById("user").value;
      const password = document.getElementById("password").value;
      const msg = document.getElementById("msg");

      msg.textContent = "Verificando...";

      try {
        const ref = doc(db, "users", user);
        const snap = await getDoc(ref);

        if (!snap.exists()) {
          msg.textContent = "Usuário não existe";
          return;
        }

        if (snap.data().password !== password) {
          msg.textContent = "Senha incorreta";
          return;
        }

        msg.style.color = "green";
        msg.textContent = "Login realizado com sucesso";

      } catch (e) {
        msg.textContent = "Erro de conexão com o banco";
        console.error(e);
      }
    };
  </script>

</body>
</html>
