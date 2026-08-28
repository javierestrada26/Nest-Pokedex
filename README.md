<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" /></a>
</p>

# Nest Pokédex API

API RESTful completa para una Pokédex construida con **NestJS**, **TypeScript**, **MongoDB / Mongoose**, **Docker** y consumo de **PokéAPI**.

---

## 📋 Tabla de Contenidos
- [Variables de Entorno](#-variables-de-entorno)
- [Ejecución en Desarrollo](#-ejecución-en-desarrollo)
- [Ejecución en Producción con Docker](#-ejecución-en-producción-con-docker)
- [Persistencia de Datos y Análisis de Imagen](#-persistencia-de-datos-y-análisis-de-imagen)
- [Apuntes Técnicos y Arquitectura del Proyecto](#-apuntes-técnicos-y-arquitectura-del-proyecto)

---

## ⚙️ Variables de Entorno

El proyecto utiliza `@nestjs/config` con validación mediante **Joi**. 

1. Copia el archivo de plantilla `.env.template` y renombralo a `.env`:
   ```bash
   cp .env.template .env
   ```

2. Configuración por defecto:
   ```env
   MONGODB=mongodb://localhost:27017/nest-pokemon
   PORT=3000
   ```

- **Configuración de Variables (`src/config/app.config.ts`):** Mapea variables a un objeto tipado (`EnvConfiguration`).
- **Esquema de Validación (`src/config/joi.validation.ts`):** Garantiza mediante `Joi.object` que la variable `MONGODB` sea requerida y `PORT` tenga un valor por defecto numérico.

---

## 🚀 Ejecución en Desarrollo

1. **Instalar dependencias:**
   ```bash
   npm install
   ```

2. **Levantar base de datos en Docker (Desarrollo):**
   ```bash
   docker-compose up -d
   ```

3. **Iniciar la aplicación en modo watch:**
   ```bash
   npm run start:dev
   ```

4. **Sembrar la base de datos (Seed):**
   Realiza una petición GET a:
   `http://localhost:3000/api/v2/seed`

5. **Endpoint Base de Pokémon:**
   `http://localhost:3000/api/v2/pokemon`

---

## 🐳 Ejecución en Producción con Docker

El proyecto cuenta con una compilación optimizada **Multi-stage Build** en el `Dockerfile`:

### Etapas de Construcción (`Dockerfile`):
1. **`deps`**: Instala dependencias completas del proyecto.
2. **`builder`**: Compila el código TypeScript a JavaScript (`dist/`).
3. **`prod-deps`**: Instala únicamente las dependencias de producción (`npm ci --only=production`).
4. **`runner`**: Imagen final ultra liviana basada en `node:20-alpine`, copiando `dist/`, `public/` y `node_modules` de producción.

### Despliegue con Docker Compose de Producción:

1. **Construir y levantar contenedores:**
   ```bash
   docker-compose -f docker-compose.prod.yaml up -d --build
   ```

2. **Detener contenedores:**
   ```bash
   docker-compose -f docker-compose.prod.yaml down
   ```

---

## 💾 Persistencia de Datos y Análisis de Imagen

### 1. Persistencia de la Base de Datos
En `docker-compose.prod.yaml`, se define un volumen nombrado de Docker (`mongo-db-data`):
```yaml
volumes:
  - mongo-db-data:/data/db
```
Esto garantiza que toda la información insertada en MongoDB persista de manera segura en el sistema local, incluso si el contenedor se detiene o destruye.

### 2. Análisis de la Imagen Docker
Para inspeccionar la imagen de producción creada:

- **Verificar imágenes locales y tamaño:**
  ```bash
  docker images
  ```
- **Inspeccionar capas e historia de la imagen:**
  ```bash
  docker history pokedex-prod
  ```
- **Inspeccionar detalles de configuración:**
  ```bash
  docker inspect pokedex-prod
  ```

---

## 📌 Apuntes Técnicos y Arquitectura del Proyecto

### 1. Módulo Pokémon (`PokemonModule`)
- **Entidad y Esquema (`pokemon.entity.ts`):** 
  - Extendida de `Document` de Mongoose con decoradores `@Schema()` y `@Prop()`.
  - Propiedades `name` y `no` configuradas con `{ unique: true, index: true }` para asegurar unicidad y rapidez en búsquedas.
- **DTOs y Validaciones (`create-pokemon.dto.ts`, `update-pokemon.dto.ts`):**
  - Uso de `class-validator` (`@IsString`, `@MinLength`, `@IsInt`, `@IsPositive`, `@Min(1)`).
- **Lógica de Servicio (`pokemon.service.ts`):**
  - **Búsqueda Polimórfica (`findOne`):** Permite buscar un Pokémon por número (`no`), por `MongoID` (usando `isValidObjectId`) o por `name`.
  - **Normalización:** Convierte automáticamente los nombres a minúsculas (`toLocaleLowerCase()`).
  - **Manejo Centralizado de Excepciones (`handleException`):** Intercepta el código de error `11000` (registro duplicado en MongoDB) y retorna una `BadRequestException` clara.
  - **Eliminación Optimizada:** Ejecuta `this.pokemonModel.deleteOne({ _id: id })` y valida `deletedCount` para lanzar excepción si el registro no existía.

### 2. Módulo Común (`CommonModule`)
- **Paginación (`PaginationDto`):**
  - Define `limit` y `offset` opcionales.
  - Habilitado con `enableImplicitConversion: true` en la configuración global de `ValidationPipe` en `main.ts`.
- **Custom Pipe (`ParseMongoIdPipe`):**
  - Pipe personalizado (`PipeTransform`) para validar parámetros de ruta. Verifica con `isValidObjectId` de Mongoose si el parámetro es un ID válido de Mongo antes de llegar al controlador.

### 3. Módulo Seed (`SeedModule`)
- **Poblado Automático de Datos:**
  - Consume la PokéAPI pública (`https://pokeapi.co/api/v2/pokemon?limit=650`) mediante **Axios**.
  - Limpia la base de datos con `deleteMany({})` e inserta masivamente los Pokémon con una sola operación `insertMany()`, garantizando alta eficiencia.
