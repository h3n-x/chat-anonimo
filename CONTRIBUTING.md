# 🤝 Guía de Contribución

¡Gracias por tu interés en contribuir a **Chat Anónimo**! Esta guía te ayudará a entender cómo puedes participar en el desarrollo del proyecto.

## 📋 Tabla de Contenidos

- [🚀 Comenzando](#-comenzando)
- [🛠️ Configuración del Entorno](#️-configuración-del-entorno)
- [📝 Tipos de Contribución](#-tipos-de-contribución)
- [🔄 Proceso de Contribución](#-proceso-de-contribución)
- [📊 Estándares de Código](#-estándares-de-código)
- [🧪 Testing](#-testing)
- [📖 Documentación](#-documentación)
- [🐛 Reporte de Bugs](#-reporte-de-bugs)
- [💡 Solicitud de Features](#-solicitud-de-features)

---

## 🚀 Comenzando

### 📋 **Requisitos Previos**
- **Git** instalado
- **Node.js 18+** para el frontend
- **Python 3.11+** para el backend
- **Editor de código** (VS Code recomendado)

### 🍴 **Fork del Repositorio**
```bash
# Dependiendo de lo que quieras contribuir:

# Para Backend:
git clone https://github.com/TU-USUARIO/chat-backend.git
cd chat-backend
git remote add upstream https://github.com/h3n-x/chat-backend.git

# Para Frontend:
git clone https://github.com/TU-USUARIO/chat-frontend.git
cd chat-frontend
git remote add upstream https://github.com/h3n-x/chat-frontend.git
```

---

## 🛠️ Configuración del Entorno

### 🐍 **Backend Setup**
```bash
git clone https://github.com/h3n-x/chat-backend.git
cd chat-backend

# Crear entorno virtual
python -m venv venv
source venv/bin/activate  # Linux/Mac
# o
venv\Scripts\activate     # Windows

# Instalar dependencias
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Dependencias de desarrollo

# Ejecutar tests
python -m pytest tests/
```

### 🎨 **Frontend Setup**
```bash
git clone https://github.com/h3n-x/chat-frontend.git
cd chat-frontend

# Instalar dependencias
npm install

# Ejecutar en desarrollo
npm run dev

# Ejecutar tests
npm run test

# Linting
npm run lint
```

---

## 📝 Tipos de Contribución

### 🐛 **Bug Fixes**
- Corrección de errores en el código
- Mejoras de rendimiento
- Parches de seguridad

### ✨ **Nuevas Características**
- Funcionalidades adicionales
- Mejoras de UI/UX
- Integraciones nuevas

### 📖 **Documentación**
- Mejoras en README
- Documentación de API
- Tutoriales y guías

### 🧪 **Testing**
- Tests unitarios
- Tests de integración
- Tests end-to-end

### 🎨 **Diseño**
- Mejoras visuales
- Accesibilidad
- Responsive design

---

## 🔄 Proceso de Contribución

### 1. **📍 Crear Issue (Opcional pero Recomendado)**
```markdown
Antes de empezar a trabajar en algo grande, crea un issue para:
- Discutir la propuesta
- Evitar trabajo duplicado
- Obtener feedback temprano
```

### 2. **🌿 Crear Rama de Feature**
```bash
# Sincronizar con upstream
git fetch upstream
git checkout main
git merge upstream/main

# Crear nueva rama
git checkout -b feature/nombre-descriptivo
# o
git checkout -b bugfix/descripcion-del-bug
```

### 3. **💻 Desarrollar**
```bash
# Hacer cambios
# Seguir estándares de código
# Agregar tests si es necesario
# Actualizar documentación
```

### 4. **🧪 Testing**
```bash
# Backend
cd backend
python -m pytest tests/

# Frontend
cd frontend
npm run test
npm run lint
npm run type-check
```

### 5. **📝 Commit y Push**
```bash
# Commits descriptivos
git add .
git commit -m "feat: agregar funcionalidad de notificaciones push"

# Push a tu fork
git push origin feature/nombre-descriptivo
```

### 6. **🔄 Pull Request**
- Ir a GitHub y crear Pull Request
- Usar la plantilla de PR
- Describir los cambios claramente
- Referenciar issues relacionados

---

## 📊 Estándares de Código

### 🐍 **Python (Backend)**
```python
# Seguir PEP 8
# Usar type hints
def encrypt_message(message: str, room_id: Optional[str] = None) -> Dict[str, Any]:
    """
    Cifra un mensaje usando AES-256-GCM.
    
    Args:
        message: El mensaje a cifrar
        room_id: ID de la sala (opcional)
        
    Returns:
        Diccionario con datos cifrados
    """
    pass

# Usar docstrings para funciones públicas
# Nombrar variables descriptivamente
# Mantener funciones pequeñas (< 50 líneas)
```

### 🎨 **TypeScript (Frontend)**
```typescript
// Usar interfaces para tipos
interface ChatMessage {
  id: string
  message: string
  timestamp: string
  userId: string
}

// Componentes con tipos explícitos
export function ChatMessage({ message, timestamp }: ChatMessageProps) {
  return (
    <div className="message">
      <span>{message}</span>
      <time>{timestamp}</time>
    </div>
  )
}

// Usar nombres descriptivos
// Preferir funciones arrow para componentes
// Mantener componentes pequeños y enfocados
```

### 📝 **Commits**
```bash
# Formato: tipo(scope): descripción

# Tipos válidos:
feat:     # Nueva funcionalidad
fix:      # Corrección de bug
docs:     # Cambios en documentación
style:    # Cambios de formato (sin cambios de lógica)
refactor: # Refactoring de código
test:     # Agregar o modificar tests
chore:    # Tareas de mantenimiento

# Ejemplos:
feat(chat): agregar cifrado end-to-end para mensajes
fix(upload): corregir límite de tamaño de archivos
docs(readme): actualizar instrucciones de instalación
```

---

## 🧪 Testing

### 🐍 **Backend Tests**
```python
# tests/test_crypto.py
def test_message_encryption():
    """Test que verifica el cifrado de mensajes."""
    crypto = ChatCrypto()
    message = "Mensaje de prueba"
    
    encrypted = crypto.encrypt_message(message)
    decrypted = crypto.decrypt_message(encrypted)
    
    assert decrypted == message
    assert encrypted['algorithm'] == 'AES-256-GCM'

# Ejecutar tests
python -m pytest tests/ -v
```

### 🎨 **Frontend Tests**
```typescript
// __tests__/chat-message.test.tsx
import { render, screen } from '@testing-library/react'
import { ChatMessage } from '../components/chat-message'

describe('ChatMessage', () => {
  it('should render message correctly', () => {
    const props = {
      message: 'Test message',
      timestamp: '2025-01-01T00:00:00Z',
      userId: 'user123'
    }
    
    render(<ChatMessage {...props} />)
    
    expect(screen.getByText('Test message')).toBeInTheDocument()
  })
})
```

### 📊 **Coverage**
```bash
# Backend coverage
python -m pytest --cov=. tests/

# Frontend coverage
npm run test -- --coverage
```

---

## 📖 Documentación

### 📝 **README Updates**
- Mantener documentación actualizada
- Incluir ejemplos de código
- Actualizar screenshots si es necesario

### 🔗 **API Documentation**
```python
# Documentar endpoints con docstrings
@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    """
    Sube un archivo al chat.
    
    Args:
        file: Archivo a subir (máximo 15MB)
        
    Returns:
        Información del archivo subido
        
    Raises:
        HTTPException: Si el archivo es muy grande o tipo no válido
    """
    pass
```

### 💬 **Comentarios en Código**
```typescript
// Explicar lógica compleja
function generateRoomKey(): string {
  // Generar clave AES-256 usando Web Crypto API
  // para máxima seguridad y compatibilidad
  const key = crypto.getRandomValues(new Uint8Array(32))
  return arrayBufferToBase64(key)
}
```

---

## 🐛 Reporte de Bugs

### 📝 **Template de Bug Report**
```markdown
**Descripción del Bug**
Descripción clara y concisa del problema.

**Pasos para Reproducir**
1. Ir a '...'
2. Hacer clic en '...'
3. Scrollear hacia '...'
4. Ver error

**Comportamiento Esperado**
Descripción de lo que esperabas que pasara.

**Screenshots**
Si aplica, agregar screenshots para explicar el problema.

**Información del Sistema:**
- OS: [e.g. Windows 11, macOS 14, Ubuntu 22.04]
- Browser: [e.g. Chrome 121, Firefox 122]
- Versión: [e.g. v1.2.3]

**Contexto Adicional**
Cualquier otra información relevante sobre el problema.
```

### 🔍 **Investigación Previa**
Antes de reportar:
- [ ] Buscar en issues existentes
- [ ] Reproducir en incógnito/privado
- [ ] Probar en diferentes browsers
- [ ] Verificar la última versión

---

## 💡 Solicitud de Features

### 📝 **Template de Feature Request**
```markdown
**¿Tu solicitud está relacionada con un problema?**
Descripción clara del problema. Ej: "Siempre me molesta cuando [...]"

**Describe la solución que te gustaría**
Descripción clara de lo que quieres que pase.

**Describe alternativas que has considerado**
Cualquier solución alternativa o características que hayas considerado.

**Contexto Adicional**
Screenshots, mockups, o cualquier contexto adicional sobre la solicitud.
```

### 🎯 **Criterios de Aceptación**
- Alineado con los objetivos del proyecto
- Técnicamente factible
- No compromete la seguridad
- Mejora la experiencia del usuario

---

## 🔐 Consideraciones de Seguridad

### 🛡️ **Reporte de Vulnerabilidades**
```markdown
Para reportar vulnerabilidades de seguridad:
1. NO crear un issue público
2. Enviar email a: security@[proyecto-email]
3. Incluir descripción detallada
4. Proporcionar pasos para reproducir
5. Tiempo de respuesta esperado: 48 horas
```

### 🔒 **Guidelines de Seguridad**
- Nunca hardcodear credenciales
- Validar todas las entradas
- Usar HTTPS en producción
- Mantener dependencias actualizadas

---

## 🏆 Reconocimientos

### 🌟 **Contributors**
Los contributors aparecerán automáticamente en:
- README del proyecto
- Página de releases
- Hall of Fame (si aplica)

### 🎖️ **Tipos de Contribución**
- 💻 Código
- 📖 Documentación
- 🐛 Bug Reports
- 💡 Ideas
- 🎨 Diseño
- 🧪 Testing

---

## 📞 Contacto y Ayuda

### 💬 **Canales de Comunicación**
- **GitHub Issues**: Para bugs y features
- **GitHub Discussions**: Para preguntas generales
- **Email**: Para asuntos privados/seguridad

### 🤝 **Mentorship**
Si eres nuevo en open source, ¡estamos aquí para ayudar!
- Pregunta en Discussions
- Busca issues etiquetados como `good-first-issue`
- No dudes en pedir ayuda

---

## 📜 Código de Conducta

### 🤝 **Nuestro Compromiso**
- Ambiente acogedor e inclusivo
- Respeto por diferentes puntos de vista
- Feedback constructivo
- Enfoque en lo mejor para la comunidad

### 🚫 **Comportamiento Inaceptable**
- Lenguaje o imágenes sexualizadas
- Ataques personales o políticos
- Acoso público o privado
- Publicar información privada sin permiso

---

<div align="center">

**¡Gracias por contribuir al Chat Anónimo!** 🎉

Tu participación hace que este proyecto sea mejor para todos.

[⬆️ Volver al inicio](#-guía-de-contribución)

</div>
