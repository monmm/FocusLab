## 1. MVC
Para este proyecto, se ha implementado el patrón **Model-View-Controller (MVC)**. 
A diferencia de las arquitecturas modernas como MVVM, el MVC en Android distribuye las responsabilidades de la siguiente manera:

* **Modelo:** Representado por la entidad `Session` y la clase `SessionManager`. Se encarga de la lógica de datos y la persistencia en SQLite.
* **Vista:** Los archivos de diseño XML y componentes visuales como `RecyclerView`.
* **Controlador:** Las clases `MainActivity` y `SessionHistoryActivity`. Actúan como intermediarios, capturando eventos del usuario y coordinando las actualizaciones entre el Modelo y la Vista.

## 2. Gestión de Datos con SQLite
Siguiendo las notas de clase, se utiliza un **Contrato** y un **Helper** para estructurar la base de datos.

### A. Definición del Contrato
Se define la estructura de la tabla de sesiones para garantizar la integridad de la información. Las columnas corresponden a los atributos de la entidad `Session`: `type`, `date`, `startTime`, `duration` y `completed`.

### B. Implementación de `SessionManager`
Esta clase extiende `SQLiteOpenHelper`. Sus funciones principales son:
* **`onCreate`**: Ejecuta el script SQL para crear la tabla si no existe.
* **`onUpgrade`**: Permite la migración de datos si la estructura de la tabla cambia en versiones futuras.
* **`saveSession(Session)`**: Utiliza `ContentValues` para realizar operaciones de inserción (Create) de forma segura.
* **`getAllSessions()`**: Ejecuta una consulta (Read) y devuelve una lista de objetos, transformando los datos del `Cursor` a objetos Java.

## 3. Lógica 
El controlador gestiona cómo y cuándo se guardan o recuperan los datos.

### A. Uso en `MainActivity`
Es el punto donde se origina la información.
* **Inicialización:** El `SessionManager` debe instanciarse en el `onCreate` proporcionando el contexto de la actividad.
* **Captura de Datos:** Al iniciar una sesión, se crea una instancia de `Session`. Los textos se obtienen mediante `getString(R.string...)` para cumplir con la estrategia de recursos y evitar valores fijos (*hardcoded*).
* **Persistencia:** Al finalizar o interrumpir la sesión (por ejemplo, en el método `onStop` o mediante un evento de botón), el controlador solicita al modelo guardar la información.

### B. Gestión del Ciclo de Vida y Orientación
Para simplificar la persistencia en memoria durante la ejecución:
* **Orientación:** Se restringe la actividad a modo *Portrait* en el manifiesto para evitar la destrucción y recreación innecesaria de la interfaz.
* **Estado:** En caso de que la actividad deba recrearse, se utiliza `onSaveInstanceState` para preservar variables temporales del cronómetro que aún no han sido enviadas al modelo SQLite.

## 4. Visualización del Historial
En `SessionHistoryActivity`, el controlador solicita la lista completa al modelo y la vincula a la interfaz.

* **Recuperación Asíncrona:** Siguiendo las recomendaciones de rendimiento, la carga de datos del historial debe realizarse sin bloquear el hilo principal (UI Thread).
* **Empty State:** Si la base de datos no devuelve registros, la vista debe mostrar un estado vacío (por ejemplo, una `ProgressBar` o un mensaje informativo) para retroalimentar al usuario.
* **Adaptador:** El `SessionHistoryAdapter` toma la colección de objetos `Session` y los infla en tarjetas individuales siguiendo los principios de Material Design.

## 5. Requerimientos Técnicos Cumplidos
1. **Persistencia:** Uso de SQLite nativo para asegurar que los datos no se pierdan al cerrar la app.
2. **Arquitectura:** Separación clara entre la interfaz (View), la lógica de control (Activity) y el acceso a datos (Helper/Model).
3. **Recursos:** Uso estricto de identificadores de recursos para soporte multi-idioma.
