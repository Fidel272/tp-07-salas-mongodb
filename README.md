# Trabajo práctico 07

## 1. Punto de partida y nombre de la copia

- **Punto de partida**: Copia funcional del TP-06: `tp-06-salas-modular`, conservando el repositorio individual de GitHub y la estructura previa.
- **Nombre de la copia**: `tp-07-salas-mongodb`.

## 2. Versiones utilizadas y confirmación de Atlas Free
* **Node.js**: `v22.22.0` (con soporte nativo para archivos de entorno mediante `--env-file`).
* **Mongoose**: Versión `9.10.3`.
* **Entorno de Base de Datos**: Se confirma el uso del clúster gratuito en la nube **MongoDB Atlas Free (M0)** (igual al preparado durante las clases), utilizando una base de datos propia denominada `reservas_salas_u7`.

## 3. Pasos desde una descarga nueva: instalar dependencias, preparar el clúster gratuito, usuario e IP permitida,copiar la plantilla de entorno y ejecutar.

1. **Clonar el repositorio y posicionarse en la carpeta raíz**:
```bash
cd tp-07-salas-mongodb
```
Instalar las dependencias:
```bash
npm install
```

2. Configurar el entorno:
```bash
cp .env.example .env
```
Abrir el archivo .env generado y configurar la variable MONGODB_URI utilizando tu clúster de MongoDB Atlas (cluster0.iqnjoef.mongodb.net), especificando la base de datos obligatoria reservas_salas_u7 y tus credenciales de usuario (revisar .env.example)

3. Preparar el clúster en MongoDB Atlas:
- Verificar que se esté utilizando el clúster gratuito en la nube MongoDB Atlas Free (M0).
- Comprobar que el usuario de base de datos configurado cuente con los permisos necesarios de lectura y escritura.
- Configurar el acceso a la red (Network Access) permitiendo la dirección IP correspondiente o habilitando temporalmente el acceso general (0.0.0.0/0) para el entorno de prácticas de la materia.

## 4. Diagnóstico DNS, resolutores y comandos de ejecución

### Diagnóstico DNS y Red
Para evitar bloqueos o fallos en la resolución de nombres SRV/TXT de MongoDB Atlas en redes locales o de aula, se configuran explícitamente resolutores DNS autorizados (`8.8.8.8` y `1.1.1.1`) dentro de `src/config/base-datos.js` mediante `dns.setServers()`.

### Comandos de ejecución mediante npm scripts

- Comprobar conexión con la base de datos:
```bash
npm run comprobardb
```
- Crear una reserva de prueba (escritura):
```bash
npm run crear
```
- Consultar y listar todas las reservas:
```bash
npm run listar
```
- Consultar y listar todas las reservas:
```bash
npm run listar
```

- Verificación de calidad de código (Prettier y ESLint):
```bash
npm run check
```

## 5. Mapa de archivos y explicación de la arquitectura

### Árbol de archivos principales
```
tp-07-salas-mongodb/
├── src/
│   ├── config/
│   │   ├── entorno.js          # Centraliza variables de entorno (PORT, NODE_ENV, MONGODB_URI)
│   │   └── base-datos.js       # Configuración DNS y funciones conectarBaseDatos / desconectarBaseDatos
│   ├── modelos/
│   │   └── reserva.js          # Esquema y modelo Mongoose asociado explícitamente a la colección
│   └── ...                    # Archivos previos de la app web en memoria
├── scripts/
│   ├── comprobar-conexion.js   # Script de diagnóstico de red y conexión
│   ├── crear-reserva.js        # Script CLI para insertar un documento de prueba
│   └── listar-reservas.js      # Script CLI para consultar y mostrar el listado tabular
├── .env.example               # Plantilla pública de variables de entorno
├── .gitignore
├── eslint.config.js
└── package.json
```

### Esquema, Modelo y Colección
- Esquema (Schema): Define de manera estricta los tipos de datos, validaciones de formato, límites numéricos y opciones permitidas mediante Mongoose.
- Modelo (Model): Interfaz compilada a partir del esquema (mongoose.model("Reserva", esquemaReserva)) que permite interactuar directamente con la base de datos.
- Colección: Se asocia explícitamente a la colección reservas mediante la opción { collection: "reservas" }.

## 6. Campos, reglas mínimas y límites pendientes

| **Campo** | **Tipo** | **Regla mínima / Validación** |
|---|---|---|
| **estudiante** | `String` | Obligatorio, sin espacios en extremos (`trim`) y no vacío. |
| **email** | `String` | Obligatorio, con `trim`, no vacío y presencia del carácter `@` mediante `match`. |
| **sala** | `String` | Obligatorio. Valores permitidos (`enum`): *Sala Norte*, *Sala Sur* o *Sala Multimedia*. |
| **fecha** | `String` | Obligatorio, con `trim` y validación de contenido no vacío. |
| **turno** | `String` | Obligatorio. Valores permitidos (`enum`): *Mañana*, *Tarde* o *Noche*. |
| **personas** | `Number` | Obligatorio, entero entre 1 y 6, validado mediante `min`, `max` y `Number.isInteger`. |

Límites aún no controlados: El alcance actual no considera la validación semántica avanzada de fechas imposibles, superposición de horarios, cupos compartidos ni validación estricta del formato de correo electrónico más allá de la presencia del símbolo @. El identificador _id generado es un ObjectId nativo de MongoDB que solo se convierte a texto (.toString()) al momento de visualizarse. Un _id asignado en memoria a un objeto de instancia previa todavía no demuestra una escritura física confirmada hasta que el método asíncrono de persistencia finaliza con éxito.

## 7. Matriz de pruebas y evidencia de persistencia

### Matriz de Pruebas Realizada

| **Caso de Prueba** | **Resultado Observado** |
|---|---|
| **Conexión válida** | Confirmación en consola y proceso finalizado con éxito. |
| **Fallo DNS simulado** | Detección de código de red/DNS, informe de error y salida con código `1`. |
| **Variable `MONGODB_URI` vacía** | Fallo de configuración controlado, sin mensaje de éxito. |
| **Colección vacía** | Listado ejecutado correctamente con cantidad `0` de registros. |
| **Reserva válida (`npm run crear`)** | Documento guardado exitosamente y retorno del `_id` en texto. |
| **Consulta desde proceso nuevo (`npm run listar`)** | Recuperación exitosa del mismo `_id` creado previamente. |
| **Estudiante vacío o con espacios** | Rechazo por validación del esquema, cantidad de registros sin cambios. |
| **Email sin `@`** | Rechazo por regla `match`, sin alteraciones en la base. |
| **Sala o turno no permitido** | Rechazo por restricción `enum`, sin escritura. |
| **Personas fuera de rango (`0`, `7`) o decimales** | Rechazo por límites `min`/`max` y validación de números enteros. |

### Evidencia de Persistencia entre Procesos
1. Ejecución del listado inicial: cantidad recuperada N.
2. Ejecución del script de creación (`npm run crear`): se genera y confirma el `_id`.
3. Cierre del proceso y ejecución independiente de `npm run listar`.
4. Verificación: la cantidad total aumentó en una unidad y el documento con dicho `_id` exacto se recupera de forma persistente desde MongoDB Atlas.

## 8. Continuidad del sitio HTTP en memoria
El servicio web Express y las rutas operan actualmente sobre el almacenamiento en memoria heredado de las unidades previas. Esto se debe a que el objetivo de este trabajo práctico se centra exclusivamente en el diseño del laboratorio de persistencia por consola, modelos Mongoose y diagnóstico de conectividad.

## 9. Seguridad y Privacidad
- Se confirma que no se publican credenciales de acceso, contraseñas de la base de datos ni datos personales sensibles en el repositorio.
- El archivo `.env` con credenciales reales se encuentra excluido mediante `.gitignore`, versionando únicamente la plantilla pública `.env.example`.

### Matriz de Pruebas Realizada