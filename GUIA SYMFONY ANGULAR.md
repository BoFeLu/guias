# Guía Completa: Stack Symfony + Angular + Docker

Guía práctica para desarrollar aplicaciones web modernas usando Symfony como API backend, Angular como frontend y Docker para la containerización completa del entorno.

## Tabla de Contenidos

1. [Introducción al Stack](#introducción-al-stack)
2. [Requisitos Previos](#requisitos-previos)
3. [Arquitectura del Sistema](#arquitectura-del-sistema)
4. [Configuración del Backend (Symfony)](#configuración-del-backend-symfony)
5. [Configuración del Frontend (Angular)](#configuración-del-frontend-angular)
6. [Configuración Docker y NGINX](#configuración-docker-y-nginx)
7. [Integración y Comunicación](#integración-y-comunicación)
8. [Flujo de Desarrollo](#flujo-de-desarrollo)
9. [Comandos Útiles](#comandos-útiles)
10. [Troubleshooting](#troubleshooting)

## Introducción al Stack

### ¿Qué es este Stack?

Este stack combina tres tecnologías principales para crear aplicaciones web modernas y escalables:

- **Symfony**: Framework PHP robusto para crear APIs REST eficientes
- **Angular**: Framework de Google para interfaces de usuario dinámicas
- **Docker**: Containerización para entornos consistentes y portables

### Ventajas de esta Arquitectura

**Separación de responsabilidades**: Frontend y backend completamente independientes
**Escalabilidad**: Cada componente puede escalarse por separado
**Desarrollo paralelo**: Equipos frontend y backend pueden trabajar simultáneamente
**Flexibilidad**: Fácil integración con servicios externos y APIs de terceros

## Requisitos Previos

### Software Necesario

```bash
# Verificar versiones instaladas
node --version    # Necesario: Node.js 16+
npm --version     # Viene con Node.js
php --version     # Necesario: PHP 8.1+
composer --version # Gestor de paquetes PHP
docker --version  # Para containerización
docker compose --version
```

### Instalación de Dependencias

```bash
# Node.js (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# PHP y extensiones necesarias
sudo apt install php8.2 php8.2-cli php8.2-common php8.2-mysql \
php8.2-xml php8.2-curl php8.2-mbstring php8.2-zip

# Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Symfony CLI
curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | sudo -E bash
sudo apt install symfony-cli

# Angular CLI
npm install -g @angular/cli

# Docker (Ubuntu/Debian)
sudo apt install docker.io docker-compose
sudo usermod -aG docker $USER # Reiniciar terminal después
```

## Arquitectura del Sistema

### Estructura de Directorios

```
proyecto-stack/
├── backend/                 # API Symfony
│   ├── config/             # Configuraciones Symfony
│   ├── src/                # Código fuente PHP
│   │   ├── Controller/     # Controladores API
│   │   ├── Entity/         # Entidades de base de datos
│   │   └── Repository/     # Repositorios de datos
│   ├── templates/          # Plantillas Twig (si se usan)
│   ├── public/             # Punto de entrada web
│   ├── var/                # Cache y logs
│   ├── vendor/             # Dependencias PHP
│   ├── composer.json       # Dependencias PHP
│   └── .env                # Variables de entorno
├── frontend/               # Aplicación Angular
│   ├── src/
│   │   ├── app/
│   │   │   ├── components/ # Componentes reutilizables
│   │   │   ├── services/   # Servicios para API
│   │   │   ├── models/     # Interfaces TypeScript
│   │   │   └── app.module.ts
│   │   ├── assets/         # Recursos estáticos
│   │   └── index.html      # Página principal
│   ├── angular.json        # Configuración Angular
│   └── package.json        # Dependencias Node.js
├── docker/                 # Configuración contenedores
│   ├── nginx/
│   │   └── default.conf    # Configuración servidor web
│   ├── php/
│   │   └── Dockerfile      # Imagen PHP personalizada
│   └── docker-compose.yml  # Orquestación de servicios
├── docs/                   # Documentación
└── README.md               # Instrucciones principales
```

### Flujo de Comunicación

```
USUARIO → NAVEGADOR → NGINX → DESTINO
    ↓
    ├── Puerto 80 → PHP-FPM → SYMFONY (API Backend)
    └── Puerto 4200 → ANGULAR (Frontend)
    ↓
    MYSQL (Base de datos) ← SYMFONY
```

## Configuración del Backend (Symfony)

### Conceptos Clave de Symfony

**MVC (Modelo-Vista-Controlador)**: Separación clara de responsabilidades
- **Modelo**: Entidades y lógica de base de datos (Doctrine ORM)
- **Vista**: Respuestas JSON para APIs (sin templates HTML)
- **Controlador**: Lógica de negocio y endpoints API

**Componentes Principales**:
- **Routing**: Mapeo de URLs a métodos de controlador
- **Doctrine**: ORM para gestión de base de datos
- **Serializer**: Conversión de objetos PHP a JSON
- **Validator**: Validación de datos de entrada

### Inicialización del Proyecto Symfony

```bash
# Crear directorio principal
mkdir proyecto-stack
cd proyecto-stack

# Crear proyecto Symfony para API
mkdir backend
cd backend
symfony new . --version="6.4" --webapp

# Instalar paquetes específicos para API
composer require api cors
composer require doctrine/orm
composer require symfony/validator

# Configurar base de datos en .env
echo 'DATABASE_URL="mysql://root:root@mysql:3306/app_database"' >> .env
```

### Estructura Básica de un Controlador API

```php
<?php
// src/Controller/TaskController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Annotation\Route;

#[Route('/api/tasks', name: 'api_task_')]
class TaskController extends AbstractController
{
    #[Route('', name: 'list', methods: ['GET'])]
    public function list(): JsonResponse
    {
        // Lógica para obtener tareas
        $tasks = [
            ['id' => 1, 'title' => 'Tarea 1', 'completed' => false],
            ['id' => 2, 'title' => 'Tarea 2', 'completed' => true]
        ];
        
        return $this->json($tasks);
    }
    
    #[Route('', name: 'create', methods: ['POST'])]
    public function create(Request $request): JsonResponse
    {
        $data = json_decode($request->getContent(), true);
        
        // Lógica para crear tarea
        // Validación y persistencia en base de datos
        
        return $this->json(['message' => 'Tarea creada'], 201);
    }
}
```

### Configuración CORS para Angular

```yaml
# config/packages/cors.yaml
nelmio_cors:
    defaults:
        origin_regex: true
        allow_origin: ['http://localhost:4200']
        allow_methods: ['GET', 'OPTIONS', 'POST', 'PUT', 'PATCH', 'DELETE']
        allow_headers: ['Content-Type', 'Authorization']
        expose_headers: ['Link']
        max_age: 3600
    paths:
        '^/api/':
            allow_origin: ['http://localhost:4200']
```

## Configuración del Frontend (Angular)

### Conceptos Clave de Angular

**Componentes**: Elementos reutilizables de la interfaz de usuario
**Servicios**: Lógica de negocio y comunicación con APIs
**Módulos**: Organización y agrupación de funcionalidad
**TypeScript**: JavaScript tipado para mayor robustez

### Inicialización del Proyecto Angular

```bash
# Desde la raíz del proyecto
cd ..
mkdir frontend
cd frontend

# Crear proyecto Angular
ng new . --routing --style=scss --skip-install
npm install

# Instalar dependencias adicionales
npm install bootstrap @angular/material @angular/cdk
```

### Configuración del HttpClient

```typescript
// src/app/app.module.ts
import { HttpClientModule } from '@angular/common/http';
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

@NgModule({
  imports: [
    BrowserModule,
    HttpClientModule, // Necesario para comunicación con API
    AppRoutingModule
  ],
  // ... resto de configuración
})
export class AppModule { }
```

### Servicio para Comunicación con API

```typescript
// src/app/services/task.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

export interface Task {
  id?: number;
  title: string;
  description?: string;
  completed: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class TaskService {
  private readonly apiUrl = 'http://localhost/api/tasks';

  constructor(private http: HttpClient) {}

  getTasks(): Observable<Task[]> {
    return this.http.get<Task[]>(this.apiUrl)
      .pipe(catchError(this.handleError));
  }

  createTask(task: Task): Observable<Task> {
    return this.http.post<Task>(this.apiUrl, task)
      .pipe(catchError(this.handleError));
  }

  updateTask(id: number, task: Task): Observable<Task> {
    return this.http.put<Task>(`${this.apiUrl}/${id}`, task)
      .pipe(catchError(this.handleError));
  }

  deleteTask(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`)
      .pipe(catchError(this.handleError));
  }

  private handleError(error: HttpErrorResponse): Observable<never> {
    console.error('Error en API:', error);
    return throwError(() => error);
  }
}
```

### Componente de Ejemplo

```typescript
// src/app/components/task-list/task-list.component.ts
import { Component, OnInit } from '@angular/core';
import { TaskService, Task } from '../../services/task.service';

@Component({
  selector: 'app-task-list',
  template: `
    <div class="task-container">
      <h2>Lista de Tareas</h2>
      
      <div class="task-item" *ngFor="let task of tasks">
        <h3>{{ task.title }}</h3>
        <p>{{ task.description }}</p>
        <button (click)="toggleComplete(task)" 
                [class.completed]="task.completed">
          {{ task.completed ? 'Completada' : 'Pendiente' }}
        </button>
        <button (click)="deleteTask(task.id!)" class="delete-btn">
          Eliminar
        </button>
      </div>
      
      <button (click)="loadTasks()" class="refresh-btn">
        Actualizar Lista
      </button>
    </div>
  `,
  styles: [`
    .task-container { max-width: 600px; margin: 2rem auto; }
    .task-item { border: 1px solid #ddd; padding: 1rem; margin: 1rem 0; }
    .completed { background-color: #d4edda; }
    .delete-btn { background-color: #dc3545; color: white; }
    .refresh-btn { background-color: #007bff; color: white; }
  `]
})
export class TaskListComponent implements OnInit {
  tasks: Task[] = [];
  loading = false;

  constructor(private taskService: TaskService) {}

  ngOnInit(): void {
    this.loadTasks();
  }

  loadTasks(): void {
    this.loading = true;
    this.taskService.getTasks().subscribe({
      next: (tasks) => {
        this.tasks = tasks;
        this.loading = false;
      },
      error: (error) => {
        console.error('Error cargando tareas:', error);
        this.loading = false;
      }
    });
  }

  toggleComplete(task: Task): void {
    const updatedTask = { ...task, completed: !task.completed };
    this.taskService.updateTask(task.id!, updatedTask).subscribe({
      next: () => this.loadTasks(),
      error: (error) => console.error('Error actualizando tarea:', error)
    });
  }

  deleteTask(id: number): void {
    if (confirm('¿Estás seguro de eliminar esta tarea?')) {
      this.taskService.deleteTask(id).subscribe({
        next: () => this.loadTasks(),
        error: (error) => console.error('Error eliminando tarea:', error)
      });
    }
  }
}
```

## Configuración Docker y NGINX

### Dockerfile para PHP/Symfony

```dockerfile
# docker/php/Dockerfile
FROM php:8.2-fpm-alpine

# Instalar extensiones necesarias
RUN apk add --no-cache \
    zip unzip curl git \
    && docker-php-ext-install pdo pdo_mysql

# Instalar Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Configurar directorio de trabajo
WORKDIR /var/www/symfony

# Copiar archivos del proyecto
COPY ./backend /var/www/symfony

# Instalar dependencias PHP
RUN composer install --no-dev --optimize-autoloader

# Configurar permisos
RUN chown -R www-data:www-data /var/www/symfony/var
```

### Configuración NGINX

```nginx
# docker/nginx/default.conf

# Configuración para API Symfony
server {
    listen 80;
    server_name localhost;
    root /var/www/symfony/public;
    index index.php;

    # Configuración para endpoints API
    location /api/ {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    # Procesamiento de archivos PHP
    location ~ \.php$ {
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        
        # Headers para CORS
        add_header Access-Control-Allow-Origin "http://localhost:4200" always;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
        add_header Access-Control-Allow-Headers "Origin, Content-Type, Accept, Authorization" always;
    }

    # Manejo de preflight requests
    location ~ ^/api.*$ {
        if ($request_method = OPTIONS) {
            add_header Access-Control-Allow-Origin "http://localhost:4200";
            add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
            add_header Access-Control-Allow-Headers "Origin, Content-Type, Accept, Authorization";
            add_header Content-Length 0;
            add_header Content-Type text/plain;
            return 200;
        }
    }
}

# Configuración para Angular Frontend
server {
    listen 4200;
    server_name localhost;
    root /var/www/angular;
    index index.html;

    # Configuración para SPA (Single Page Application)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Caché para assets estáticos
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### Docker Compose

```yaml
# docker/docker-compose.yml
version: '3.8'

services:
  # Servidor web NGINX
  nginx:
    image: nginx:1.25-alpine
    container_name: stack_nginx
    ports:
      - "80:80"      # API Symfony
      - "4200:4200"  # Frontend Angular
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ../backend/public:/var/www/symfony/public
      - ../frontend/dist:/var/www/angular
    depends_on:
      - php
      - angular
    networks:
      - app-network

  # Contenedor PHP para Symfony
  php:
    build:
      context: .
      dockerfile: php/Dockerfile
    container_name: stack_php
    volumes:
      - ../backend:/var/www/symfony
    environment:
      - APP_ENV=dev
      - DATABASE_URL=mysql://root:rootpassword@mysql:3306/app_database
    depends_on:
      - mysql
    networks:
      - app-network

  # Base de datos MySQL
  mysql:
    image: mysql:8.0
    container_name: stack_mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: app_database
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppassword
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app-network

  # Contenedor para Angular (desarrollo)
  angular:
    image: node:18-alpine
    container_name: stack_angular
    working_dir: /app
    volumes:
      - ../frontend:/app
    command: sh -c "npm install && ng serve --host 0.0.0.0 --port 4200"
    ports:
      - "4201:4200"  # Puerto alternativo para desarrollo directo
    networks:
      - app-network

volumes:
  mysql_data:

networks:
  app-network:
    driver: bridge
```

## Integración y Comunicación

### Flujo de Datos Completo

1. **Usuario interactúa** con Angular (frontend)
2. **Angular hace petición HTTP** a Symfony API
3. **NGINX recibe la petición** y la enruta a PHP-FPM
4. **Symfony procesa la petición**, consulta MySQL si es necesario
5. **Symfony devuelve respuesta JSON** a Angular
6. **Angular actualiza la interfaz** con los datos recibidos

### Ejemplo Práctico Completo

**Crear una nueva tarea desde Angular:**

```typescript
// En el componente Angular
createTask(): void {
  const newTask: Task = {
    title: 'Nueva tarea',
    description: 'Descripción de la tarea',
    completed: false
  };

  this.taskService.createTask(newTask).subscribe({
    next: (createdTask) => {
      console.log('Tarea creada:', createdTask);
      this.loadTasks(); // Recargar lista
    },
    error: (error) => console.error('Error:', error)
  });
}
```

**Procesar la petición en Symfony:**

```php
// En TaskController.php
#[Route('', name: 'create', methods: ['POST'])]
public function create(Request $request, EntityManagerInterface $em): JsonResponse
{
    $data = json_decode($request->getContent(), true);
    
    // Crear nueva entidad
    $task = new Task();
    $task->setTitle($data['title']);
    $task->setDescription($data['description'] ?? '');
    $task->setCompleted($data['completed'] ?? false);
    $task->setCreatedAt(new \DateTime());
    
    // Persistir en base de datos
    $em->persist($task);
    $em->flush();
    
    return $this->json($task, 201);
}
```

## Flujo de Desarrollo

### Configuración Inicial

```bash
# 1. Crear estructura de proyecto
mkdir proyecto-stack && cd proyecto-stack
mkdir -p {backend,frontend,docker/{nginx,php},docs}

# 2. Configurar backend
cd backend
symfony new . --webapp
composer require api cors doctrine/orm

# 3. Configurar frontend  
cd ../frontend
ng new . --routing --style=scss
npm install @angular/material bootstrap

# 4. Configurar Docker
cd ../docker
# Crear archivos docker-compose.yml y configuraciones NGINX

# 5. Levantar entorno completo
docker-compose up -d
```

### Desarrollo Diario

```bash
# Iniciar entorno de desarrollo
cd docker
docker-compose up -d

# Verificar que todos los servicios están funcionando
docker-compose ps

# Ver logs en tiempo real
docker-compose logs -f

# Acceder a contenedor PHP para comandos Symfony
docker-compose exec php bash
# Dentro del contenedor:
php bin/console doctrine:database:create
php bin/console make:entity Task
php bin/console doctrine:migrations:migrate

# Desarrollo frontend con hot-reload
cd ../frontend
ng serve --host 0.0.0.0 --port 4200

# Parar entorno cuando termines
cd ../docker
docker-compose down
```

### Estructura de Trabajo por Componentes

**Backend Developer:**
- Trabaja en `backend/src/`
- Crea controladores, entidades y servicios
- Usa contenedor PHP para comandos Symfony
- Prueba endpoints con herramientas como Postman

**Frontend Developer:**
- Trabaja en `frontend/src/`
- Desarrolla componentes y servicios Angular
- Consume API através de servicios HTTP
- Prueba interfaz en http://localhost:4200

## Comandos Útiles

### Symfony (Backend)

```bash
# Acceder al contenedor PHP
docker-compose exec php bash

# Comandos dentro del contenedor
php bin/console list                           # Ver todos los comandos disponibles
php bin/console make:entity Task               # Crear entidad
php bin/console make:controller ApiTaskController # Crear controlador
php bin/console doctrine:database:create       # Crear base de datos
php bin/console doctrine:migrations:migrate    # Ejecutar migraciones
php bin/console doctrine:fixtures:load         # Cargar datos de prueba
php bin/console cache:clear                    # Limpiar caché
```

### Angular (Frontend)

```bash
# Comandos Angular CLI
ng generate component task-list         # Crear componente
ng generate service services/task       # Crear servicio
ng generate module tasks --routing      # Crear módulo con routing
ng build --prod                        # Build de producción
ng serve --host 0.0.0.0                # Servidor de desarrollo
ng test                                 # Ejecutar tests unitarios
ng lint                                 # Verificar calidad de código
```

### Docker y Desarrollo

```bash
# Gestión de contenedores
docker-compose up -d                    # Levantar en background
docker-compose down                     # Parar y eliminar contenedores
docker-compose restart nginx            # Reiniciar servicio específico
docker-compose logs -f php              # Ver logs de servicio específico
docker-compose exec mysql mysql -u root -p # Acceder a MySQL

# Limpieza y mantenimiento
docker-compose down --volumes           # Eliminar volúmenes también
docker system prune                     # Limpiar imágenes no utilizadas
docker-compose build --no-cache         # Rebuild sin caché
```

### Base de Datos

```bash
# Dentro del contenedor MySQL
docker-compose exec mysql bash
mysql -u root -p

# Comandos SQL útiles
SHOW DATABASES;
USE app_database;
SHOW TABLES;
DESCRIBE task;

# Backup de base de datos
docker-compose exec mysql mysqldump -u root -p app_database > backup.sql

# Restaurar backup
docker-compose exec -T mysql mysql -u root -p app_database < backup.sql
```

## Troubleshooting

### Problemas Comunes y Soluciones

#### Error de CORS en Angular

**Síntoma**: Angular no puede conectar con la API
**Solución**:
```bash
# Verificar configuración CORS en Symfony
# Verificar headers en NGINX
# Comprobar que Angular hace peticiones a la URL correcta
```

#### Error de conexión a base de datos

**Síntoma**: Symfony no puede conectar con MySQL
**Solución**:
```bash
# Verificar que el contenedor MySQL está ejecutándose
docker-compose ps

# Verificar variables de entorno en .env
# Comprobar que MySQL está aceptando conexiones
docker-compose exec mysql mysql -u root -p -e "SHOW DATABASES;"
```

#### Angular no recarga automáticamente

**Síntoma**: Cambios en código no se reflejan
**Solución**:
```bash
# Usar ng serve con configuración de host
ng serve --host 0.0.0.0 --poll=2000

# Verificar que no hay errores de compilación
ng build
```

#### Contenedores no se levantan

**Síntoma**: docker-compose up falla
**Solución**:
```bash
# Verificar logs detallados
docker-compose up --no-daemon

# Verificar puertos ocupados
sudo netstat -tlnp | grep :80

# Reconstruir imágenes
docker-compose build --no-cache
```

### Logs y Debugging

```bash
# Ver logs de Symfony
docker-compose exec php tail -f var/log/dev.log

# Ver logs de NGINX
docker-compose logs nginx

# Ver logs de Angular en desarrollo
# Los errores aparecen en la consola del navegador

# Monitorear todas las peticiones HTTP
# Usar herramientas de developer tools del navegador
```

### Verificación de Configuración

```bash
# Verificar configuración Symfony
docker-compose exec php php bin/console debug:config
docker-compose exec php php bin/console debug:router

# Verificar configuración Angular
ng config

# Test de conectividad API
curl -X GET http://localhost/api/tasks
curl -X POST http://localhost/api/tasks -d '{"title":"Test"}' -H "Content-Type: application/json"
```

---

Esta guía proporciona una base sólida para desarrollar aplicaciones web modernas usando el stack Symfony + Angular + Docker. La clave del éxito está en la configuración correcta inicial y el entendimiento del flujo de comunicación entre los diferentes componentes.

Para proyectos en producción, considera aspectos adicionales como seguridad, optimización de rendimiento, CI/CD, y monitoreo de aplicaciones.