<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" /></a>
</p>

# Nest Pokedex API

RESTful API para una Pokédex desarrollada con NestJS, TypeScript y MongoDB (Mongoose).

---

## 🚀 Ejecutar en desarrollo

1. **Clonar repositorio e instalar dependencias:**
   ```bash
   npm install
   ```

2. **Nest CLI (opcional pero recomendado):**
   ```bash
   npm i -g @nestjs/cli
   ```

3. **Levantar base de datos en Docker:**
   ```bash
   docker-compose up -d
   ```

4. **Iniciar en modo desarrollo:**
   ```bash
   npm run start:dev
   ```

5. **Endpoint base:**
   `http://localhost:3000/api/v2/pokemon`

---

## 📌 Apuntes del Proyecto y Aspectos Clave

### 1. Configuración Global de la Aplicación (`main.ts` y `app.module.ts`)
- **Prefijo Global:** Configurado a `api/v2` en `main.ts`.
- **ValidationPipe Global:** Activo con `whitelist: true` y `forbidNonWhitelisted: true` para evitar propiedades no solicitadas en los DTOs.
- **Servidor Estático:** Configurado mediante `ServeStaticModule` mapeando la carpeta `./public`.

### 2. Base de Datos y Entidad Mongoose (`pokemon.entity.ts`)
- **Esquema Mongoose:** Definido con decoradores `@Schema()` y `@Prop()`, extendiendo de `Document`.
- **Índices y Unicidad:** Campos `name` y `no` configurados con `{ unique: true, index: true }` para asegurar registros únicos y optimizar consultas.

### 3. Lógica de Negocio y Servicios (`pokemon.service.ts`)
- **Búsqueda por Múltiples Criterios (`findOne`):**
  - Soporta búsqueda polimórfica por número (`no`), por `MongoID` (mediante `isValidObjectId`) y por `name`.
  - Normaliza nombres de Pokémon a minúsculas (`toLocaleLowerCase()`).
- **Manejo Centralizado de Excepciones (`handleException`):**
  - Captura el código de error de MongoDB `11000` (registro duplicado) y retorna una `BadRequestException` descriptiva.
  - Lanza `InternalServerErrorException` para errores no contemplados.
- **Eliminación Optimizada (`remove`):**
  - Utiliza `this.pokemonModel.deleteOne({ _id: id })` evaluando `deletedCount` para verificar si el registro existía.

### 4. Custom Pipes Reutilizables (`common/pipes/parse-mongo-id`)
- **`ParseMongoIdPipe`:** Custom Pipe que implementa `PipeTransform` para validar en los `@Param` si un string es un `MongoID` válido antes de procesar la solicitud, lanzando `BadRequestException` en caso contrario.

### 5. Entorno local con Docker (`docker-compose.yaml`)
- Servicio de MongoDB (`mongo:7`) expuesto en el puerto `27017` con persistencia mediante el volumen `mongo-data`.


