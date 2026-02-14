# GraphQL Schema - Plataforma E-Learning

## Descripción General
Este documento define el schema GraphQL que proporciona una interfaz flexible y eficiente para consultar datos de la plataforma E-Learning, permitiendo a los clientes solicitar exactamente los datos que necesitan.

**GraphQL Endpoint**: `https://api.elearning-platform.com/graphql`

---

## Tipos de Datos (Types)

### User
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  role: UserRole!
  profile: UserProfile
  enrolledCourses: [Enrollment!]!
  createdCourses: [Course!]
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### UserProfile
```graphql
type UserProfile {
  bio: String
  avatar: String
  location: String
  socialLinks: [SocialLink!]
}
```

### UserRole
```graphql
enum UserRole {
  STUDENT
  TEACHER
  ADMIN
}
```

### Course
```graphql
type Course {
  id: ID!
  title: String!
  description: String!
  instructor: User!
  category: CourseCategory!
  level: CourseLevel!
  duration: Int!
  price: Float!
  thumbnail: String
  lessons: [Lesson!]!
  enrollments: [Enrollment!]!
  enrolledStudents: Int!
  rating: Float
  reviews: [Review!]
  published: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### CourseCategory
```graphql
enum CourseCategory {
  PROGRAMMING
  DESIGN
  BUSINESS
  MARKETING
  DATA_SCIENCE
  LANGUAGES
  OTHER
}
```

### CourseLevel
```graphql
enum CourseLevel {
  BEGINNER
  INTERMEDIATE
  ADVANCED
}
```

### Lesson
```graphql
type Lesson {
  id: ID!
  title: String!
  description: String
  course: Course!
  order: Int!
  duration: Int!
  type: LessonType!
  content: LessonContent!
  completed(userId: ID!): Boolean!
}
```

### LessonType
```graphql
enum LessonType {
  VIDEO
  TEXT
  QUIZ
  ASSIGNMENT
}
```

### LessonContent
```graphql
type LessonContent {
  videoUrl: String
  textContent: String
  resources: [Resource!]
  quiz: Quiz
}
```

### Resource
```graphql
type Resource {
  id: ID!
  name: String!
  type: String!
  url: String!
  size: Int
}
```

### Enrollment
```graphql
type Enrollment {
  id: ID!
  user: User!
  course: Course!
  progress: Int!
  completedLessons: [Lesson!]!
  totalLessons: Int!
  status: EnrollmentStatus!
  enrolledAt: DateTime!
  completedAt: DateTime
  lastAccessedAt: DateTime
}
```

### EnrollmentStatus
```graphql
enum EnrollmentStatus {
  ACTIVE
  COMPLETED
  DROPPED
}
```

### Review
```graphql
type Review {
  id: ID!
  user: User!
  course: Course!
  rating: Int!
  comment: String
  createdAt: DateTime!
}
```

### Quiz
```graphql
type Quiz {
  id: ID!
  questions: [Question!]!
  passingScore: Int!
}
```

### Question
```graphql
type Question {
  id: ID!
  text: String!
  options: [String!]!
  correctAnswer: Int!
}
```

### SocialLink
```graphql
type SocialLink {
  platform: String!
  url: String!
}
```

---

## Queries (Consultas)

### Usuarios

#### Obtener todos los usuarios
```graphql
query GetUsers($role: UserRole, $limit: Int, $offset: Int) {
  users(role: $role, limit: $limit, offset: $offset) {
    id
    name
    email
    role
    createdAt
  }
}
```

**Ejemplo de uso:**
```graphql
query {
  users(role: STUDENT, limit: 10) {
    id
    name
    email
    enrolledCourses {
      course {
        title
      }
      progress
    }
  }
}
```

#### Obtener usuario por ID
```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    role
    profile {
      bio
      avatar
    }
    enrolledCourses {
      course {
        title
      }
      progress
    }
  }
}
```

**Ejemplo de uso:**
```graphql
query {
  user(id: "usr_001") {
    name
    email
    enrolledCourses {
      id
      progress
      course {
        title
        instructor {
          name
        }
      }
    }
  }
}
```

---

### Cursos

#### Obtener todos los cursos
```graphql
query GetCourses(
  $category: CourseCategory
  $level: CourseLevel
  $published: Boolean
  $limit: Int
  $offset: Int
) {
  courses(
    category: $category
    level: $level
    published: $published
    limit: $limit
    offset: $offset
  ) {
    id
    title
    description
    instructor {
      name
    }
    category
    level
    price
    rating
  }
}
```

**Ejemplo de uso:**
```graphql
query {
  courses(category: PROGRAMMING, level: INTERMEDIATE, published: true) {
    id
    title
    description
    instructor {
      name
      profile {
        avatar
      }
    }
    lessons {
      id
      title
      duration
    }
    enrolledStudents
    rating
  }
}
```

#### Obtener curso por ID
```graphql
query GetCourse($id: ID!) {
  course(id: $id) {
    id
    title
    description
    instructor {
      id
      name
      profile {
        bio
        avatar
      }
    }
    category
    level
    duration
    price
    lessons {
      id
      title
      description
      duration
      type
      order
    }
    reviews {
      user {
        name
      }
      rating
      comment
      createdAt
    }
    rating
    enrolledStudents
  }
}
```

---

### Inscripciones

#### Obtener inscripciones de un usuario
```graphql
query GetUserEnrollments($userId: ID!) {
  userEnrollments(userId: $userId) {
    id
    course {
      title
      thumbnail
      instructor {
        name
      }
    }
    progress
    status
    enrolledAt
    lastAccessedAt
  }
}
```

**Ejemplo de uso:**
```graphql
query {
  userEnrollments(userId: "usr_001") {
    id
    progress
    status
    course {
      title
      lessons {
        id
        title
        completed(userId: "usr_001")
      }
    }
    completedLessons {
      title
    }
  }
}
```

---

### Búsqueda

#### Búsqueda global
```graphql
query Search($query: String!, $type: SearchType) {
  search(query: $query, type: $type) {
    courses {
      id
      title
      instructor {
        name
      }
    }
    users {
      id
      name
      role
    }
  }
}
```

---

## Mutations (Modificaciones)

### Usuarios

#### Crear usuario
```graphql
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
    role
    createdAt
  }
}
```

**Input Type:**
```graphql
input CreateUserInput {
  name: String!
  email: String!
  password: String!
  role: UserRole!
}
```

**Ejemplo de uso:**
```graphql
mutation {
  createUser(input: {
    name: "Ana López"
    email: "ana.lopez@email.com"
    password: "SecurePass123!"
    role: STUDENT
  }) {
    id
    name
    email
    createdAt
  }
}
```

#### Actualizar perfil de usuario
```graphql
mutation UpdateUserProfile($userId: ID!, $input: UpdateProfileInput!) {
  updateUserProfile(userId: $userId, input: $input) {
    id
    profile {
      bio
      avatar
    }
  }
}
```

---

### Cursos

#### Crear curso
```graphql
mutation CreateCourse($input: CreateCourseInput!) {
  createCourse(input: $input) {
    id
    title
    description
    instructor {
      name
    }
    createdAt
  }
}
```

**Input Type:**
```graphql
input CreateCourseInput {
  title: String!
  description: String!
  category: CourseCategory!
  level: CourseLevel!
  duration: Int!
  price: Float!
  thumbnail: String
}
```

**Ejemplo de uso:**
```graphql
mutation {
  createCourse(input: {
    title: "Introducción a GraphQL"
    description: "Aprende a construir APIs con GraphQL"
    category: PROGRAMMING
    level: BEGINNER
    duration: 20
    price: 49.99
  }) {
    id
    title
    instructor {
      name
    }
  }
}
```

#### Publicar curso
```graphql
mutation PublishCourse($courseId: ID!) {
  publishCourse(courseId: $courseId) {
    id
    title
    published
  }
}
```

---

### Inscripciones

#### Inscribir estudiante
```graphql
mutation EnrollStudent($input: EnrollmentInput!) {
  enrollStudent(input: $input) {
    id
    user {
      name
    }
    course {
      title
    }
    enrolledAt
    status
  }
}
```

**Input Type:**
```graphql
input EnrollmentInput {
  userId: ID!
  courseId: ID!
}
```

**Ejemplo de uso:**
```graphql
mutation {
  enrollStudent(input: {
    userId: "usr_001"
    courseId: "crs_045"
  }) {
    id
    user {
      name
    }
    course {
      title
      instructor {
        name
      }
    }
    progress
    enrolledAt
  }
}
```

#### Marcar lección como completada
```graphql
mutation CompleteLesson($enrollmentId: ID!, $lessonId: ID!) {
  completeLesson(enrollmentId: $enrollmentId, lessonId: $lessonId) {
    id
    progress
    completedLessons {
      id
      title
    }
  }
}
```

---

### Reseñas

#### Crear reseña
```graphql
mutation CreateReview($input: CreateReviewInput!) {
  createReview(input: $input) {
    id
    user {
      name
    }
    course {
      title
    }
    rating
    comment
    createdAt
  }
}
```

**Input Type:**
```graphql
input CreateReviewInput {
  courseId: ID!
  rating: Int!
  comment: String
}
```

---

## Subscriptions (Tiempo Real)

### Progreso del curso
```graphql
subscription OnProgressUpdate($enrollmentId: ID!) {
  progressUpdated(enrollmentId: $enrollmentId) {
    enrollmentId
    progress
    completedLessons {
      id
      title
    }
  }
}
```

### Nuevas inscripciones
```graphql
subscription OnNewEnrollment($courseId: ID!) {
  newEnrollment(courseId: $courseId) {
    id
    user {
      name
    }
    enrolledAt
  }
}
```

---

## Tipos Escalares Personalizados
```graphql
scalar DateTime
scalar JSON
```

---

## Ventajas de GraphQL en este Sistema

1. **Consultas Flexibles**: Los clientes obtienen exactamente los datos que necesitan
2. **Sin Over-fetching**: No se descargan datos innecesarios
3. **Sin Under-fetching**: Una sola consulta puede obtener datos relacionados
4. **Tipado Fuerte**: El schema proporciona validación automática
5. **Introspección**: El schema es auto-documentado
6. **Actualizaciones en Tiempo Real**: Subscriptions para notificaciones

---

## Ejemplo de Consulta Compleja
```graphql
query GetStudentDashboard($userId: ID!) {
  user(id: $userId) {
    name
    profile {
      avatar
    }
    enrolledCourses {
      id
      progress
      status
      course {
        title
        thumbnail
        instructor {
          name
          profile {
            avatar
          }
        }
        lessons {
          id
          title
          duration
          completed(userId: $userId)
        }
      }
      completedLessons {
        id
      }
      lastAccessedAt
    }
  }
}
```

Esta consulta obtiene en **una sola petición**:
- Información del usuario
- Todos sus cursos inscritos
- Progreso de cada curso
- Información del instructor
- Lista de lecciones con estado de completado
- Última fecha de acceso

**En REST esto requeriría múltiples peticiones:**
1. GET /users/{userId}
2. GET /users/{userId}/enrollments
3. GET /courses/{courseId} (por cada curso)
4. GET /courses/{courseId}/lessons (por cada curso)
5. GET /enrollments/{enrollmentId}/progress (por cada inscripción)

---

**Documentación generada**: Febrero 2024  
**Versión de Schema**: v1.0  
**GraphQL Spec**: October 2021
