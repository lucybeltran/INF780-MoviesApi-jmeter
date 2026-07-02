# Pruebas de Rendimiento (INF780) - Movies API

Este directorio contiene todos los archivos `.jmx` (planes de pruebas de JMeter), `.jtl` (archivos de resultados) y los reportes HTML autogenerados para la materia de INF780.

## Estructura del Directorio
```
INF780-MoviesApi-jmeter/
├── README.md              # Este archivo con instrucciones
├── ids.csv                # Listado de UUIDs extraídos de la BD para las pruebas
├── smoke.jmx              # Plan de prueba de Humo
├── carga.jmx              # Plan de prueba de Carga
├── estres.jmx             # Plan de prueba de Estrés (equivalente a 100 usuarios)
├── estres_100.jmx         # Plan de prueba de Estrés con 100 usuarios
├── estres_200.jmx         # Plan de prueba de Estrés con 200 usuarios
├── estres_400.jmx         # Plan de prueba de Estrés con 400 usuarios
├── picos.jmx              # Plan de prueba de Picos (Spike Test)
├── smoke.jtl              # Resultados de Humo
├── carga.jtl              # Resultados de Carga
├── estres_100.jtl         # Resultados de Estrés 100
├── estres_200.jtl         # Resultados de Estrés 200
├── estres_400.jtl         # Resultados de Estrés 400
├── picos.jtl              # Resultados de Picos
├── reporte-smoke/         # Reporte HTML autogenerado de Humo
├── reporte-carga/         # Reporte HTML autogenerado de Carga
├── reporte-estres-100/    # Reporte HTML autogenerado de Estrés 100
├── reporte-estres-200/    # Reporte HTML autogenerado de Estrés 200
├── reporte-estres-400/    # Reporte HTML autogenerado de Estrés 400
├── reporte-picos/         # Reporte HTML autogenerado de Picos
└── informe/
    └── informe.md         # Informe detallado de rendimiento (Markdown/PDF)
```

---

## 1. Cómo Levantar la API REST (Backend)

La API REST está ubicada en el directorio raíz del repositorio. Sigue estos pasos para levantarla:

1. **Instalar dependencias:**
   ```bash
   npm install
   ```
2. **Configurar el archivo `.env`:**
   Crea un archivo `.env` en la raíz con tus credenciales de PostgreSQL (ejemplo):
   ```env
   DB_HOST=localhost
   DB_PORT=5432
   DB_USERNAME=movies_user
   DB_PASSWORD=123456
   DB_NAME=movies_api
   PORT=3000
   ```
3. **Iniciar la base de datos y levantar la API en desarrollo:**
   ```bash
   npm run start:dev
   ```
   La API quedará expuesta en `http://localhost:3000`.

---

## 2. Cómo Ejecutar los Planes de Pruebas (.jmx) en Modo No-GUI

Ubícate dentro del directorio `INF780-MoviesApi-jmeter/` en tu terminal antes de ejecutar los comandos:

```bash
cd INF780-MoviesApi-jmeter
```

### 1. Prueba de Humo (Smoke Test)
* **Hilos:** 1, **Ramp-up:** 1s, **Loops:** 5.
```bash
jmeter -n -t smoke.jmx -l smoke.jtl -e -o reporte-smoke
```

### 2. Prueba de Carga (Load Test)
* **Hilos:** 50, **Ramp-up:** 30s, **Loops:** 10.
```bash
jmeter -n -t carga.jmx -l carga.jtl -e -o reporte-carga
```

### 3. Pruebas de Estrés (Stress Tests)
Las pruebas de estrés evalúan el rendimiento bajo concurrencias crecientes manteniendo Loops (10) y Ramp-Up (30s):

* **100 Usuarios:**
  ```bash
  jmeter -n -t estres_100.jmx -l estres_100.jtl -e -o reporte-estres-100
  ```
* **200 Usuarios:**
  ```bash
  jmeter -n -t estres_200.jmx -l estres_200.jtl -e -o reporte-estres-200
  ```
* **400 Usuarios:**
  ```bash
  jmeter -n -t estres_400.jmx -l estres_400.jtl -e -o reporte-estres-400
  ```

### 4. Prueba de Picos (Spike Test)
* **Hilos:** 200, **Ramp-up:** 5s, **Loops:** 10, con Duration Assertion de 800 ms y Response Assertion:
  ```bash
  jmeter -n -t picos.jmx -l picos.jtl -e -o reporte-picos
  ```

---

## Optimización de Memoria (RAM)
Para prevenir excepciones `OutOfMemoryError` en JMeter producidas al descargar payloads JSON de 9.5MB (`GET /movies`) bajo alta concurrencia, todos los archivos JMX de carga, estrés y picos cuentan con un postprocesador JSR223 (Groovy) que ejecuta `prev.setResponseData(new byte[0]);`. Esto descarta los datos de la respuesta de la RAM inmediatamente después de validarlos con las aserciones, permitiendo corridas fluidas con cientos de hilos concurrentes.
