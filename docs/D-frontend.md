# D. Creación de frontend

[← C. Creación de backend](C-backend.md) | [Volver al README](../README.md) | [Siguiente: E. Despliegue local →](E-despliegue-local.md)

## 1. Crear proyecto React

```bash
cd ../frontend
npm create vite@latest . -- --template react
npm install
npm install axios react-router-dom firebase
```

<img width="1173" height="569" alt="image" src="https://github.com/user-attachments/assets/f6c0a7ef-e906-4283-b1d5-738db2cfa911" />


## 2. Crear `.env.example`

Archivo `frontend/.env.example`:

```env
VITE_API_URL=http://localhost:8000/api
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_APP_ID=
```

Copiarlo a `frontend/.env` y completar con los datos del proyecto ya existente.

## 3. Configurar Firebase en frontend

Archivo `frontend/src/firebaseConfig.js`:

```javascript
import { initializeApp } from "firebase/app";
import { getAuth, signInWithPopup, GoogleAuthProvider } from "firebase/auth";

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);

export const auth = getAuth(app);
export const provider = new GoogleAuthProvider();
export { signInWithPopup };
```

## 4. Crear cliente HTTP

Archivo `frontend/src/api.js`:

```javascript
import axios from "axios";

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("firebaseToken");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
```

## 5. Crear páginas

### `frontend/src/LoginPage.jsx`

```javascript
import React from "react";
import { useNavigate } from "react-router-dom";
import { auth, provider, signInWithPopup } from "./firebaseConfig";

export default function LoginPage() {
  const navigate = useNavigate();

  const handleLogin = async () => {
    const result = await signInWithPopup(auth, provider);
    const token = await result.user.getIdToken();
    localStorage.setItem("firebaseToken", token);
    navigate("/company");
  };

  return (
    <div>
      <h2>Login</h2>
      <button onClick={handleLogin}>Ingresar con Google</button>
    </div>
  );
}
```

### `frontend/src/CompanyPage.jsx`

```javascript
import React, { useState } from "react";
import api from "./api";

export default function CompanyPage() {
  const [form, setForm] = useState({
    name: "",
    nit: "",
    city: "",
    sector: "",
  });
  const [msg, setMsg] = useState("");

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await api.post("/companies/", form);
      setMsg("Empresa creada");
    } catch {
      setMsg("Error al crear empresa");
    }
  };

  return (
    <div>
      <h2>Empresa</h2>
      {msg && <p>{msg}</p>}
      <form onSubmit={handleSubmit}>
        <input name="name" placeholder="Nombre" onChange={handleChange} />
        <input name="nit" placeholder="NIT" onChange={handleChange} />
        <input name="city" placeholder="Ciudad" onChange={handleChange} />
        <input name="sector" placeholder="Sector" onChange={handleChange} />
        <button type="submit">Guardar</button>
      </form>
    </div>
  );
}
```

### `frontend/src/MaterialsPage.jsx`

```javascript
import React, { useState, useEffect } from "react";
import api from "./api";

export default function MaterialsPage() {
  const [companies, setCompanies] = useState([]);
  const [items, setItems] = useState([]);
  const [form, setForm] = useState({
    company_id: "",
    material_type: "",
    quantity: "",
    unit: "kg",
    location: "",
    status: "available",
  });

  useEffect(() => {
    loadCompanies();
    loadItems();
  }, []);

  const loadCompanies = async () => {
    const res = await api.get("/companies/");
    setCompanies(res.data);
  };

  const loadItems = async () => {
    const res = await api.get("/material-listings/");
    setItems(res.data);
  };

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await api.post("/material-listings/", form);
    loadItems();
  };

  return (
    <div>
      <h2>Materiales</h2>

      <form onSubmit={handleSubmit}>
        <select name="company_id" onChange={handleChange}>
          <option value="">Seleccione empresa</option>
          {companies.map((c) => (
            <option key={c.id} value={c.id}>{c.name}</option>
          ))}
        </select>

        <input name="material_type" placeholder="Tipo de material" onChange={handleChange} />
        <input name="quantity" placeholder="Cantidad" onChange={handleChange} />
        <input name="unit" placeholder="Unidad" onChange={handleChange} />
        <input name="location" placeholder="Ubicación" onChange={handleChange} />

        <button type="submit">Crear publicación</button>
      </form>

      <hr />

      {items.map((item) => (
        <div key={item.id}>
          <strong>{item.material_type}</strong> - {item.quantity} {item.unit} - {item.location}
        </div>
      ))}
    </div>
  );
}
```

## 6. Configurar rutas

Archivo `frontend/src/App.jsx`:

```javascript
import React from "react";
import { BrowserRouter, Routes, Route, Link } from "react-router-dom";
import LoginPage from "./LoginPage";
import CompanyPage from "./CompanyPage";
import MaterialsPage from "./MaterialsPage";

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/login">Login</Link> |{" "}
        <Link to="/company">Empresa</Link> |{" "}
        <Link to="/materials">Materiales</Link>
      </nav>

      <Routes>
        <Route path="/login" element={<LoginPage />} />
        <Route path="/company" element={<CompanyPage />} />
        <Route path="/materials" element={<MaterialsPage />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

## Evidencia Twelve-Factor transversal en esta sección

- **Factor I. Codebase**: frontend en el mismo repositorio.
- **Factor II. Dependencies**: `package.json`.
- **Factor III. Config**: variables `VITE_*`.
- **Factor IV. Backing services**: frontend autenticando con Firebase y consumiendo el backend por configuración.
- **Factor VII. Port binding**: interfaz servida por puerto local.
