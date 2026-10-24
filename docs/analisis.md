# F1 · Análisis — TareaFácil con MySQL (gestor de tareas)

## 1. Caso de estudio
Aplicación web de una sola página donde un usuario inicia sesión, crea tareas,
las marca como completadas y las elimina. Los datos viven en una base de datos
MySQL (XAMPP). El navegador y PHP se comunican con JSON.

## 2. Reglas de negocio
| Id | Regla |
|----|-------|
| R1 | Solo un usuario con sesión iniciada puede ver o cambiar tareas. |
| R2 | El título de una tarea tiene entre 3 y 100 caracteres. |
| R3 | Cada usuario puede tener máximo 50 tareas. |
| R4 | Cada usuario solo ve y modifica sus propias tareas. |
| R5 | Una tarea nace pendiente; se puede marcar completada/pendiente y eliminar. |

## 3. Flujo de datos
Usuario → Navegador (HTML + JS) → `fetch` con JSON → PHP (`api/`) → consulta SQL → MySQL → PHP arma la respuesta JSON → JS dibuja la lista.

## 4. Integraciones
- XAMPP: Apache (sirve la página y ejecuta PHP) y MySQL (guarda los datos).
- phpMyAdmin para crear y revisar la base de datos.
- Bootstrap 5 por CDN.
- `fetch` (AJAX) entre el navegador y la API PHP.
- Git para versionar cada fase.

## 5. Modelo de datos (MySQL)
Base de datos: `tareafacil`

**usuarios**
| Columna | Tipo | Detalle |
|---------|------|---------|
| id | INT, clave primaria | autoincremental |
| usuario | VARCHAR(30) | único (no se repite) |
| nombre | VARCHAR(60) | |
| clave_hash | VARCHAR(255) | clave cifrada, nunca la clave real |

**tareas**
| Columna | Tipo | Detalle |
|---------|------|---------|
| id | INT, clave primaria | autoincremental |
| usuario_id | INT | clave foránea → usuarios.id |
| titulo | VARCHAR(100) | |
| completada | TINYINT(1) | 0 = pendiente, 1 = completada |
| creada | DATETIME | fecha de creación automática |

Relación: **un usuario tiene muchas tareas** (1 a N).

## 6. Contrato de la API
| Método | URL | Cuerpo | Respuestas |
|--------|-----|--------|-----------|
| GET | api/sesion.php | — | 200 `{autenticado, usuario}` |
| POST | api/login.php | `{usuario, clave}` | 200 · 401 |
| POST | api/logout.php | — | 200 |
| GET | api/tareas.php | — | 200 `{tareas}` · 401 |
| POST | api/tareas.php | `{titulo}` | 201 `{tarea}` · 401 · 422 |
| PUT | api/tareas.php?id=N | `{completada}` | 200 `{tarea}` · 401 · 404 |
| DELETE | api/tareas.php?id=N | — | 200 `{ok}` · 401 · 404 |

## 7. Pantallas y navegación (análisis del Frontend)
- **Login:** usuario, clave, botón Entrar.
- **Tareas:** campo de texto + botón Agregar, contador, lista (casilla, título, botón Eliminar), botón Cerrar sesión.
- Flujo: Login → (datos correctos) → Tareas → (Cerrar sesión) → Login.
- Si la sesión vence mientras se usa la app → Login con un aviso.

**Estados que la interfaz debe mostrar**
| Situación | Qué ve el usuario |
|-----------|-------------------|
| Lista vacía | Mensaje «Aún no tienes tareas» |
| Error (clave mala, título inválido, servidor caído) | Alerta roja con el motivo |
| Tarea completada | Título tachado y casilla marcada |

**Cada elemento de la interfaz usa un endpoint**
| Elemento | Endpoint |
|----------|----------|
| Cargar la página | GET api/sesion.php |
| Botón Entrar | POST api/login.php |
| Mostrar la lista | GET api/tareas.php |
| Botón Agregar | POST api/tareas.php |
| Casilla de completada | PUT api/tareas.php?id=N |
| Botón Eliminar | DELETE api/tareas.php?id=N |
| Botón Cerrar sesión | POST api/logout.php |

**Requisitos no funcionales:** se ve bien en celular, se puede usar solo con teclado y los textos son claros.

## 8. Casos de prueba (se ejecutan en F4)
| Caso | Acción | Resultado esperado |
|------|--------|--------------------|
| C1 | GET sesion.php sin login | 200, `autenticado:false` |
| C2 | GET tareas.php sin login | 401 |
| C3 | Login con clave incorrecta | 401 |
| C4 | Login correcto | 200 |
| C5 | Crear tarea con título de 2 caracteres | 422 |
| C6 | Crear tarea con título de 101 caracteres | 422 |
| C7 | Crear tarea válida | 201, `completada:false` |
| C8 | Listar tareas | 200, contiene la tarea |
| C9 | Marcar completada | 200, `completada:true` |
| C10 | Eliminar tarea | 200 |
| C11 | Eliminar la misma tarea otra vez | 404 |
| C12 | Crear la tarea número 51 | 422 |
| C13 | Cerrar sesión | 200 |
| C14 | GET tareas.php después de salir | 401 |
