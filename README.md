# Privesc-need-restart-2024-48990
## Explicación del Exploit (CVE-2024-48990 - Pure Python)

Este script explota la vulnerabilidad **CVE-2024-48990** en la utilidad `needrestart` de Linux mediante el secuestro de la ruta de librerías (*PYTHONPATH hijacking*). A diferencia del vector original, este método utiliza exclusivamente Python, lo que permite saltarse el requisito de tener compiladores como `gcc` instalados en la máquina víctima.

### Funcionamiento Paso a Paso

1. **Estructura de Directorios:** Se crea el directorio `/tmp/malicious/importlib`. Esto imita la estructura de la librería estándar de Python.
2. **Payload Malicioso (`importlib/__init__.py`):**
Contiene el código que se ejecutará con privilegios elevados. Cuando se carga, comprueba si el usuario actual es `root` (UID 0). Si es así, copia el binario `/bin/bash` a `/tmp/poc` y le asigna el bit SUID (`chmod 4755`). Esto crea una puerta trasera que permite a cualquier usuario escalar privilegios.
3. **Proceso Cebo (`e.py`):**
Es un script inofensivo que se mantiene en ejecución en un bucle infinito. Cumple dos funciones:
* Mantener un intérprete de Python corriendo en el sistema para que `needrestart` lo intercepte.
* Monitorear constantemente el directorio `/tmp`. En el momento en que detecta que se ha creado el archivo `/tmp/poc`, lo ejecuta automáticamente con la flag `-p` (para no perder los privilegios de root) entregando la shell interactiva.


4. **Secuestro de Variables de Entorno:**
Al ejecutar el cebo con `PYTHONPATH="$PWD" python3 e.py`, alteramos el comportamiento de Python. Obligamos a que cualquier importación busque primero en `/tmp/malicious` antes que en las rutas legítimas del sistema.
5. **Activación y Escalada (`needrestart`):**
Cuando `needrestart` se ejecuta con privilegios de administrador (ya sea manualmente o por una tarea programada del sistema), escanea los procesos activos. Al llegar a nuestro `e.py`, extrae sus variables de entorno e inicia su propio proceso de Python para analizarlo. Al heredar el `PYTHONPATH` malicioso, el proceso root de `needrestart` importa nuestra falsa librería `importlib` y ejecuta el payload, otorgando control total sobre la máquina.
