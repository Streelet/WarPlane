# War Plane

![imagen.png](https://raw.githubusercontent.com/tu_usuario/tu_repositorio/main/imagen.png)

## Versión

1.0

## Fecha de Última Actualización

20 de mayo de 2025

## Desarrolladores

* Erick Pineda
* Ramiro Chacón
* Mario Palma
* Eduardo Cruz

---

## 🚀 Introducción

Este proyecto es un juego de disparos espaciales (Space Shooter) desarrollado en Unity, donde el jugador controla una nave para sobrevivir a oleadas de enemigos, acumular puntuación y gestionar su salud. El diseño busca la modularidad y la claridad del código para facilitar su entendimiento y futuras expansiones.

## 🏗️ Arquitectura General del Sistema

El juego sigue un diseño modular y basado en componentes, encapsulando cada funcionalidad principal en scripts independientes. La comunicación entre módulos se realiza principalmente mediante referencias directas en el Inspector de Unity y el uso del patrón Singleton para sistemas globales.

Los componentes principales del sistema incluyen:

* `GameManager` (Singleton): Control central de la partida.
* `ScoreManager` (Singleton - Implícito): Gestión de la puntuación.
* `Health` (Componente reutilizable): Sistema de puntos de vida para entidades.
* `PlayerHealth` (Componente específico): Interfaz de usuario para la salud del jugador.
* `ShootController` (Componente de proyectil): Define el comportamiento de las balas.
* `MainMenuManager` (Componente de escena): Controla la navegación del menú principal.
* `Spawner` (Sistema de Generación de Enemigos): Gestiona la aparición de adversarios.
* Sistema de UI (Canvas): Presentación visual al usuario.

## 🧩 Módulos y Componentes Clave

A continuación, se describen brevemente los scripts fundamentales, sus responsabilidades y los campos configurables principales.

### `GameManager.cs` (Gestión de Juego)

Controla el ciclo de vida de la partida, la pausa, el Game Over y las transiciones de escena. Es un Singleton.

* **Funcionalidades Principales**: Inicialización del juego, activación de la secuencia de Game Over (pausar, mostrar panel, limpiar escena, desvanecer música), y carga del menú principal.
* **Campos Clave**: `gameOverPanel`, `finalScoreText`, `mainMenuSceneName`, `soundtrackObject`, `musicFadeOutDuration`, `scoreTextUI`, `playerHealthBarImage`.

### `ScoreManager.cs` (Gestión de Puntuación - Implícito)

Administra la puntuación total del jugador. Se accede como un Singleton.

* **Funcionalidades Principales**: Sumar puntos, obtener la puntuación actual.
* **Interacción**: Los enemigos notifican a este manager al ser destruidos.

### `Health.cs` (Sistema de Salud Universal)

Componente reutilizable para cualquier entidad con puntos de vida (jugador, enemigos).

* **Funcionalidades Principales**: Gestionar salud actual y máxima, procesar daño y curación, y manejar la lógica de "muerte" (activar efectos, añadir puntuación para enemigos, notificar a `GameManager` si es el jugador).
* **Campos Clave**: `maxHealth`, `scoreValue`, `deathEffectPrefab`.

### `PlayerHealth.cs` (Control de Salud del Jugador)

Específico para el jugador, se encarga de la integración de la salud con la UI. Requiere `Health.cs`.

* **Funcionalidades Principales**: Monitorizar y actualizar visualmente la barra de vida del jugador en la interfaz de usuario.
* **Campos Clave**: `healthBarImage`.

### `ShootController.cs` (Control de Disparos)

Se adjunta a los prefabs de los proyectiles del jugador.

* **Funcionalidades Principales**: Definir movimiento, duración, daño y lógica de colisión de las balas (activar efectos, aplicar daño a enemigos, reproducir sonidos).
* **Campos Clave**: `direction`, `speed`, `lifespan`, `bulletDamage`, `hitEffectPrefab`, `hitEnemySound`.

### `MainMenuManager.cs` (Gestión del Menú Principal)

Script exclusivo de la escena `MainMenu`.

* **Funcionalidades Principales**: Cargar la escena de juego (`StartGame()`) y cerrar la aplicación (`QuitGame()`).
* **Campos Clave**: `gameSceneName`.

### `Spawner.cs` (Sistema de Generación de Enemigos - Asumido)

Encargado de la aparición dinámica de enemigos en la escena de juego.

* **Funcionalidades Principales**: Generar enemigos periódicamente, controlar tipo de enemigo a generar y posiciones de aparición.
* **Interacción**: El `GameManager` los destruye al finalizar la partida.
* **Campos Clave (Ejemplos)**: `enemyPrefab`, `spawnInterval`, `spawnPoints`, `maxEnemies`.

### Sistema de Interfaz de Usuario (UI)

Basado en un `Canvas` principal y sus elementos.

* **`Canvas`**: Contenedor raíz de la UI.
    * **`Canvas Scaler`**: Esencial para la adaptabilidad de la UI a diferentes resoluciones. Configurado típicamente con `UI Scale Mode` en `Scale With Screen Size`, una `Reference Resolution` (ej. 1920x1080) y `Screen Match Mode` en `Match Width Or Height` (Match 0.5).
* **Elementos Específicos**: `GameOverPanel`, `ScoreTextUI`, `PlayerHealthBarImage`.

## ⚙️ Flujos de Operación Clave

### Inicio del Juego y Transición de Escena

1.  La aplicación inicia en la `MainMenu` (primera escena en Build Settings).
2.  Los botones del menú activan métodos en `MainMenuManager`.
3.  `MainMenuManager.StartGame()` carga la escena principal del juego (`GameScene`).
4.  En `GameScene`, `GameManager.Awake()` inicializa el juego.

### Manejo de Daño y Muerte del Jugador

1.  El jugador recibe daño (`Health.TakeDamage()`).
2.  `Health.cs` reduce la salud; `PlayerHealth.cs` actualiza la barra de vida en la UI.
3.  Si la salud del jugador llega a cero, `Health.cs` invoca `GameManager.GameOver()`.
4.  `Health.cs` del jugador luego gestiona su propia destrucción y efecto de muerte.

### Manejo de Daño y Muerte de Enemigos

1.  Una bala del jugador impacta a un enemigo (`ShootController.OnTriggerEnter2D`).
2.  `ShootController` inflige daño a `Health.cs` del enemigo, reproduce efectos y sonido.
3.  Si la salud del enemigo llega a cero, `Health.cs` del enemigo invoca su método `Die()`.
4.  `Health.Die()` del enemigo añade puntuación, muestra efecto de muerte y destruye el GameObject del enemigo.

### Proceso de "Game Over"

1.  Activado por `GameManager.GameOver()` (llamado por `Health.cs` del jugador).
2.  El juego se pausa (`Time.timeScale = 0f`).
3.  Se oculta la UI de juego y se activa el `GameOverPanel` con la puntuación final.
4.  La música de fondo se desvanece y los `Spawners` y enemigos restantes son destruidos.

## 🛠️ Configuración del Proyecto (Unity Editor)

Para que el proyecto funcione correctamente, asegúrate de las siguientes configuraciones en el Editor de Unity:

* **Etiquetado (Tags)**:
    * El GameObject del jugador debe tener la etiqueta **"Player"**.
    * Los GameObjects de los enemigos deben tener la etiqueta **"Enemy"**.
    * Los GameObjects que actúen como generadores de enemigos deben llevar la etiqueta **"Spawner"**.
* **Prefabs**:
    * Todos los objetos instanciables (nave del jugador, balas, enemigos, efectos) deben ser **prefabs** con sus scripts adjuntos.
    * Asigna todos los campos `[SerializeField]` en el Inspector (arrastrando GameObjects, prefabs, o asignando valores).
* **Build Settings (`File > Build Settings...`)**:
    * Incluye todas las escenas usadas (`MainMenu`, `GameScene`, etc.) en la lista "Scenes In Build".
    * La escena `MainMenu` debe ser la **primera escena** (índice 0).
    * La escena principal del juego debe seguirla (ej. índice 1).
* **Configuración del Canvas Scaler**:
    * En el `Canvas` de tus escenas, configura el `Canvas Scaler` con `UI Scale Mode` en `Scale With Screen Size`, establece una `Reference Resolution` (ej. 1920x1080), y `Screen Match Mode` en `Match Width Or Height` con el slider `Match` en `0.5` para asegurar una UI adaptable.

## ▶️ Cómo Jugar

1.  Clona este repositorio.
2.  Abre el proyecto en Unity (versión recomendada: [Especifica la versión de Unity, ej. 2022.3.10f1]).
3.  Asegúrate de que todas las dependencias (TextMeshPro) estén importadas si te lo solicita Unity.
4.  Navega a la escena `Assets/Scenes/MainMenu.unity`.
5.  Presiona el botón de "Play" en el editor de Unity o compila el juego.
6.  En el menú principal, haz clic en "Jugar" para iniciar la partida.
7.  Controla tu nave para esquivar y destruir enemigos, acumulando la mayor puntuación posible.
8.  Si tu nave es destruida, la partida finalizará y verás tu puntuación.

## 📜 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo `LICENSE` para más detalles.

---
