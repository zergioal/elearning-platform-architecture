# API REST - Plataforma E-Learning

## Descripción General
Esta API RESTful proporciona los endpoints necesarios para gestionar usuarios, cursos, inscripciones y contenido educativo de la plataforma E-Learning.

**Base URL**: `https://api.elearning-platform.com/v1`

---

## 1. Gestión de Usuarios

### 1.1 Obtener todos los usuarios
```http
GET /users
```

**Parámetros de consulta:**
- `role` (opcional): `student`, `teacher`, `admin`
- `page` (opcional): número de página (default: 1)
- `limit` (opcional): resultados por página (default: 20)

**Respuesta exitosa (200 OK):**
```json
{
  "data": [
    {
      "id": "usr_001",
      "name": "Juan Pérez",
      "email": "juan.perez@email.com",
      "role": "student",
      "enrolledCourses": 3,
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150
  }
}
```

### 1.2 Crear un nuevo usuario
```http
POST /users
```

**Body (JSON):**
```json
{
  "name": "María García",
  "email": "maria.garcia@email.com",
  "password": "SecurePass123!",
  "role": "student"
}
```

**Respuesta exitosa (201 Created):**
```json
{
  "id": "usr_152",
  "name": "María García",
  "email": "maria.garcia@email.com",
  "role": "student",
  "createdAt": "2024-02-13T14:20:00Z"
}
```

### 1.3 Obtener usuario por ID
```http
GET /users/{userId}
```

**Respuesta exitosa (200 OK):**
```json
{
  "id": "usr_001",
  "name": "Juan Pérez",
  "email": "juan.perez@email.com",
  "role": "student",
  "profile": {
    "bio": "Estudiante de desarrollo web",
    "avatar": "https://cdn.example.com/avatars/usr_001.jpg"
  },
  "enrolledCourses": [
    {
      "courseId": "crs_045",
      "courseName": "Desarrollo Fullstack",
      "progress": 65
    }
  ],
  "createdAt": "2024-01-15T10:30:00Z"
}
```

---

## 2. Gestión de Cursos

### 2.1 Obtener todos los cursos
```http
GET /courses
```

**Parámetros de consulta:**
- `category` (opcional): categoría del curso
- `level` (opcional): `beginner`, `intermediate`, `advanced`
- `published` (opcional): `true`, `false`

**Respuesta exitosa (200 OK):**
```json
{
  "data": [
    {
      "id": "crs_045",
      "title": "Desarrollo Fullstack con Node.js",
      "description": "Aprende a crear aplicaciones completas",
      "instructor": {
        "id": "usr_023",
        "name": "Carlos Mendoza"
      },
      "category": "Programming",
      "level": "intermediate",
      "duration": 40,
      "price": 99.99,
      "enrolledStudents": 245,
      "rating": 4.7
    }
  ]
}
```

### 2.2 Crear un nuevo curso
```http
POST /courses
```

**Autenticación requerida**: `Bearer Token` (rol: teacher o admin)

**Body (JSON):**
```json
{
  "title": "Introducción a GraphQL",
  "description": "Aprende a construir APIs con GraphQL",
  "category": "Programming",
  "level": "beginner",
  "duration": 20,
  "price": 49.99
}
```

### 2.3 Actualizar un curso
```http
PUT /courses/{courseId}
```

**Autenticación requerida**: `Bearer Token` (instructor del curso o admin)

### 2.4 Eliminar un curso
```http
DELETE /courses/{courseId}
```

**Autenticación requerida**: `Bearer Token` (admin)

---

## 3. Gestión de Inscripciones

### 3.1 Inscribir estudiante en un curso
```http
POST /enrollments
```

**Body (JSON):**
```json
{
  "userId": "usr_001",
  "courseId": "crs_045"
}
```

**Respuesta exitosa (201 Created):**
```json
{
  "id": "enr_789",
  "userId": "usr_001",
  "courseId": "crs_045",
  "enrolledAt": "2024-02-13T15:00:00Z",
  "progress": 0,
  "status": "active"
}
```

### 3.2 Obtener progreso del estudiante
```http
GET /enrollments/{enrollmentId}/progress
```

**Respuesta exitosa (200 OK):**
```json
{
  "enrollmentId": "enr_789",
  "courseId": "crs_045",
  "userId": "usr_001",
  "progress": 65,
  "completedLessons": 13,
  "totalLessons": 20,
  "lastAccessedAt": "2024-02-12T18:30:00Z"
}
```

### 3.3 Actualizar progreso
```http
PATCH /enrollments/{enrollmentId}/progress
```

**Body (JSON):**
```json
{
  "lessonId": "lsn_456",
  "completed": true
}
```

---

## 4. Gestión de Lecciones

### 4.1 Obtener lecciones de un curso
```http
GET /courses/{courseId}/lessons
```

**Respuesta exitosa (200 OK):**
```json
{
  "data": [
    {
      "id": "lsn_456",
      "title": "Introducción a Node.js",
      "description": "Conceptos básicos de Node.js",
      "order": 1,
      "duration": 15,
      "type": "video",
      "content": {
        "videoUrl": "https://cdn.example.com/videos/lsn_456.mp4",
        "resources": [
          {
            "name": "Slides.pdf",
            "url": "https://cdn.example.com/resources/slides_lsn_456.pdf"
          }
        ]
      }
    }
  ]
}
```

---

## Códigos de Estado HTTP

| Código | Descripción |
|--------|-------------|
| 200 | OK - Solicitud exitosa |
| 201 | Created - Recurso creado exitosamente |
| 400 | Bad Request - Datos inválidos |
| 401 | Unauthorized - No autenticado |
| 403 | Forbidden - Sin permisos |
| 404 | Not Found - Recurso no encontrado |
| 500 | Internal Server Error - Error del servidor |

---

## Autenticación

La API utiliza **JWT (JSON Web Tokens)** para la autenticación.

**Header requerido:**
```http
Authorization: Bearer {token}
```

**Endpoint de autenticación:**
```http
POST /auth/login
```

**Body:**
```json
{
  "email": "usuario@email.com",
  "password": "password123"
}
```

**Respuesta:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "usr_001",
    "name": "Juan Pérez",
    "email": "juan.perez@email.com",
    "role": "student"
  }
}
```

---

## Rate Limiting

- **Límite**: 100 peticiones por minuto por IP
- **Header de respuesta**: `X-RateLimit-Remaining`

---

**Documentación generada**: Febrero 2024  
**Versión de API**: v1.0
