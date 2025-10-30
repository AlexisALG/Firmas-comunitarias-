firmas-comunitarias/
├── index.html
├── package.json
├── vite.config.js
├── src/
│   ├── App.jsx
│   ├── main.jsx
│   ├── components/
│   │   ├── ProjectCard.jsx
│   │   ├── SignaturePad.jsx
│   │   └── DonationBox.jsx
│   └── styles.css
└── README.txt
<!doctype html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Firmas Comunitarias</title>
    <link rel="stylesheet" href="/src/styles.css" />
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
{
  "name": "firmas-comunitarias",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "vite": "^5.2.0"
  }
}
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
body {
  font-family: 'Segoe UI', sans-serif;
  background-color: #f6f9fc;
  margin: 0;
  padding: 0;
}

header {
  background-color: #4DA3FF;
  color: white;
  text-align: center;
  padding: 20px 0;
  font-size: 1.5em;
  font-weight: bold;
}

.container {
  max-width: 800px;
  margin: 20px auto;
  padding: 0 15px;
}

button {
  background-color: #4DA3FF;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 6px;
  cursor: pointer;
}
button:hover {
  background-color: #378de0;
}
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './styles.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
import React from 'react'

export default function ProjectCard({ project, onSign }) {
  return (
    <div className="project-card" style={{ background: 'white', padding: '15px', borderRadius: '8px', marginBottom: '15px' }}>
      <h3>{project.name}</h3>
      <p>{project.description}</p>
      <p><strong>Firmas:</strong> {project.signatures.length}</p>
      <button onClick={() => onSign(project.id)}>Firmar</button>
    </div>
  )
}
import React, { useRef } from 'react'

export default function SignaturePad({ onSubmit }) {
  const nameRef = useRef()
  const canvasRef = useRef()

  const draw = (e) => {
    const ctx = canvasRef.current.getContext('2d')
    ctx.lineWidth = 2
    ctx.lineCap = 'round'
    const rect = canvasRef.current.getBoundingClientRect()
    ctx.lineTo(e.clientX - rect.left, e.clientY - rect.top)
    ctx.stroke()
  }

  const clear = () => {
    const ctx = canvasRef.current.getContext('2d')
    ctx.clearRect(0, 0, canvasRef.current.width, canvasRef.current.height)
  }

  const handleSubmit = () => {
    const dataURL = canvasRef.current.toDataURL()
    onSubmit(nameRef.current.value, dataURL)
  }

  return (
    <div style={{ background: 'white', padding: '15px', borderRadius: '8px' }}>
      <input ref={nameRef} placeholder="Tu nombre" style={{ width: '100%', padding: '8px' }} />
      <canvas
        ref={canvasRef}
        width="400"
        height="150"
        style={{ border: '1px solid #ccc', display: 'block', marginTop: '10px' }}
        onMouseDown={(e) => {
          const ctx = canvasRef.current.getContext('2d')
          ctx.beginPath()
          canvasRef.current.addEventListener('mousemove', draw)
        }}
        onMouseUp={() => canvasRef.current.removeEventListener('mousemove', draw)}
      ></canvas>
      <button onClick={clear} style={{ marginTop: '10px', marginRight: '10px' }}>Borrar</button>
      <button onClick={handleSubmit} style={{ marginTop: '10px' }}>Enviar firma</button>
    </div>
  )
}
import React from 'react'

export default function DonationBox() {
  const PAYMENT = {
    paypal: "https://www.paypal.com/donate?hosted_button_id=YOUR_BUTTON_ID",
    mercadopago: "https://mpago.la/REPLACE_ME"
  }

  return (
    <div style={{ textAlign: 'center', marginTop: '40px', background: 'white', padding: '20px', borderRadius: '8px' }}>
      <h3>💙 Apoyá a Firmas Comunitarias</h3>
      <p>Tu ayuda cubre colectivos, papelería y gestiones para mejorar el barrio.</p>
      <a href={PAYMENT.mercadopago} target="_blank" rel="noopener noreferrer">
        <button>Donar con Mercado Pago</button>
      </a>
      <a href={PAYMENT.paypal} target="_blank" rel="noopener noreferrer" style={{ marginLeft: '10px' }}>
        <button>Donar con PayPal</button>
      </a>
    </div>
  )
}
import React, { useState } from 'react'
import ProjectCard from './components/ProjectCard.jsx'
import SignaturePad from './components/SignaturePad.jsx'
import DonationBox from './components/DonationBox.jsx'

const ADMIN_EMAIL = "Gomezalexis0210@hotmail.com"
const ADMIN_PIN = "02041003"

export default function App() {
  const [email, setEmail] = useState("")
  const [pin, setPin] = useState("")
  const [isAdmin, setIsAdmin] = useState(false)
  const [projects, setProjects] = useState([])
  const [signing, setSigning] = useState(null)

  const verifyAdmin = () => {
    if (email === ADMIN_EMAIL && pin === ADMIN_PIN) setIsAdmin(true)
    else alert("Correo o PIN incorrectos.")
  }

  const addProject = () => {
    const name = prompt("Nombre del proyecto:")
    const description = prompt("Descripción:")
    if (name) setProjects([...projects, { id: Date.now(), name, description, signatures: [] }])
  }

  const handleSign = (projectId) => setSigning(projectId)

  const submitSignature = (name, signatureData) => {
    setProjects(projects.map(p => {
      if (p.id === signing) {
        return { ...p, signatures: [...p.signatures, { name, signatureData }] }
      }
      return p
    }))
    setSigning(null)
  }

  return (
    <>
      <header>Firmas Comunitarias</header>
      <div className="container">
        {!isAdmin && (
          <div style={{ marginBottom: '20px', background: 'white', padding: '15px', borderRadius: '8px' }}>
            <h3>Acceso administrador</h3>
            <input type="email" placeholder="Correo" onChange={e => setEmail(e.target.value)} />
            <input type="password" placeholder="PIN" onChange={e => setPin(e.target.value)} />
            <button onClick={verifyAdmin}>Entrar</button>
          </div>
        )}

        {isAdmin && (
          <button onClick={addProject} style={{ marginBottom: '20px' }}>+ Nuevo Proyecto</button>
        )}

        {projects.map(p => (
          <ProjectCard key={p.id} project={p} onSign={handleSign} />
        ))}

        {signing && <SignaturePad onSubmit={submitSignature} />}

        <DonationBox />
      </div>
    </>
  )
}
1. Instalar dependencias:
   npm install

2. Ejecutar localmente:
   npm run dev

3. Subir a GitHub y luego desplegar en Vercel.

4. Reemplazar tus enlaces de donación en:
   src/components/DonationBox.jsx
   
