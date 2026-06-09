
### **Gestión de Hilos y Concurrencia: Simulación de Cafetería**

El sistema implementa un modelo de **alta concurrencia** donde miles de hilos representan entidades activas que interactúan en un entorno compartido. La gestión eficiente de estos hilos es crucial para evitar condiciones de carrera y asegurar la consistencia del estado global.

#### **1. Tipología de Actores (Hilos Independientes)**

El sistema modela tres tipos de actores, cada uno extendiendo la clase `Thread` de Java y ejecutando ciclos de vida independientes:

* **Clientes (8.000 hilos):** Simulan el comportamiento de los consumidores. Su ciclo de vida es secuencial: llegada (Parque) -> acceso (Cafetería) -> selección (Mostrador) -> pago (Caja) -> consumición (Área de Consumición).


* **Cocineros (500 hilos):** Actúan como productores que generan unidades de café y rosquillas. Su flujo intercala periodos de descanso (`SalaDescanso`), producción en `Cocina` y transporte a `Despensa`.


* **Vendedores (500 hilos):** Funcionan como intermediarios que trasladan productos desde la `Despensa` hasta el `Mostrador`, asegurando el stock disponible para los clientes.



#### **2. Estrategia de Sincronización**

Para coordinar a más de 9.000 hilos compitiendo por recursos limitados, se han aplicado las siguientes técnicas de sincronización avanzada:

* **Monitores (Exclusión Mutua y Coordinación):**
* Se utiliza el mecanismo `synchronized` en métodos críticos de las clases de zona (ej: `Mostrador`, `Caja`, `Despensa`). Esto garantiza que solo un hilo modifique el estado interno de una zona en cada instante.


* **Coordinación:** Se implementan bucles `while` con `wait()` para bloquear hilos cuando las condiciones de negocio no se cumplen (ej: `stock == 0` o aforo máximo alcanzado). Una vez que otro hilo modifica el recurso, se emplea `notifyAll()` para despertar a los hilos en espera y permitirles reevaluar su condición.




* **Control Global de Ejecución (Pausa/Reanudación):**
* Se utiliza un `ReentrantLock` con una `Condition` asociada (`PAUSE_CONDITION`) en la clase `Servidor`. Todos los hilos ejecutan un método `checkPausa()` antes de realizar acciones significativas. Si la simulación está en modo pausa, los hilos quedan bloqueados en `PAUSE_CONDITION.await()`, liberando la CPU hasta que el servidor emita un `signalAll()`.




* **Escritura Atómica en Log:**
* La clase `Logger` utiliza un método `synchronized static`. Esto bloquea el acceso a nivel de clase, permitiendo que miles de hilos escriban eventos cronológicos en `evolucion_cafeteria.txt` sin corromper el fichero ni mezclar mensajes.





#### **3. Optimización de Rendimiento**

Para maximizar el paralelismo, el diseño sigue una regla fundamental: **las operaciones que simulan el paso del tiempo (como `Thread.sleep()` representando espera, comer o trabajar) se ejecutan fuera de las secciones sincronizadas**. De este modo, los hilos solo bloquean los recursos compartidos durante el tiempo estrictamente necesario para actualizar contadores, evitando cuellos de botella que degradarían la escalabilidad del sistema.
