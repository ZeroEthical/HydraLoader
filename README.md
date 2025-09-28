# 🛡️ HydraLoader: Advanced PowerShell Execution Framework

**HydraLoader** es un framework de ejecución de payloads basado en PowerShell, altamente sofisticado y resiliente, diseñado para operaciones avanzadas de pentesting y red teaming. Emplea un enfoque de múltiples capas para la evasión, la persistencia y la ejecución en memoria, con el objetivo de operar sin ser detectado en entornos modernos y altamente monitorizados.

---

## ✨ Características Principales

HydraLoader está construido con un enfoque en el sigilo, la resiliencia y la adaptabilidad.

### 🧠 Evasión y Anti-Análisis
- **Bypass de AMSI en Memoria**: Parchea dinámicamente la Interfaz de Escaneo Antimalware (AMSI) en tiempo de ejecución para neutralizar la detección de amenazas basadas en scripts.
- **Chequeos de Entorno Completos**: Detecta y evade activamente los entornos de análisis comprobando:
    - **Depuradores**: Usa llamadas nativas a la API `IsDebuggerPresent()`.
    - **Sandboxes**: Verifica la RAM del sistema, el número de núcleos de la CPU y el tiempo de actividad.
    - **Herramientas de Análisis**: Busca procesos comunes de virtualización y análisis (ej. Wireshark, Process Monitor, herramientas de VMware/VirtualBox).
- **Ofuscación Profunda**: Todo el script está fuertemente ofuscado, con cadenas críticas (funciones de API, DLLs) codificadas en Base64 y una estructura de código compactada para disuadir el análisis estático.

### 🐍 El Motor de Persistencia "Hidra"
HydraLoader emplea un mecanismo de persistencia de doble cabeza y auto-reparación para asegurar el acceso a largo plazo y la resiliencia contra los intentos de eliminación.
- **Método 1: Tarea Programada (Privilegios Elevados)**: Crea una tarea programada disfrazada de un proceso legítimo del sistema (`Microsoft Compatibility Appraiser`) que se ejecuta con privilegios de `SYSTEM` al iniciar sesión.
- **Método 2: Suscripción a Eventos WMI (Sigilo Máximo)**: Establece una suscripción a un evento WMI permanente que se activa a intervalos. Este método es extremadamente difícil de detectar, ya que reside en el repositorio de WMI, fuera de las ubicaciones de auto-arranque habituales.
- **Capacidad de Auto-reparación**: En cada ejecución, el framework comprueba si ambos mecanismos de persistencia están activos. Si uno ha sido descubierto y eliminado, el otro lo recrea automáticamente, asegurando que la "Hidra" sobreviva.

### 🚀 Ejecución del Payload
- **Operación "Fileless" en Memoria**: El payload se descarga directamente en un búfer de memoria, se decodifica y se ejecuta sin tocar nunca el disco, minimizando la huella forense.
- **Framework de Descifrado AES-256**: Incluye una función para descifrar payloads usando AES-256 (CBC). Esto permite que el payload se almacene y transmita en un estado cifrado, haciéndolo inútil para las herramientas de inspección de red. *(Nota: Requiere un payload pre-cifrado)*.
- **Resolución Dinámica de APIs**: Resuelve todas las funciones de la API de Windows necesarias dinámicamente en tiempo de ejecución, evitando tablas de importación estáticas sospechosas.

---

## ⚙️ Arquitectura y Flujo

El flujo de ejecución está diseñado para obtener el máximo sigilo y eficiencia:

1.  **Inicialización**: El bypass de AMSI se ejecuta al instante.
2.  **Chequeos de Evasión**: El script realiza todas las comprobaciones anti-análisis y anti-sandbox. Si alguna falla, termina silenciosamente.
3.  **Chequeo y Reparación de Persistencia**: El motor Hidra verifica que tanto la Tarea Programada como la Suscripción WMI estén en su lugar. Si no, las crea.
4.  **Entrega del Payload**: El framework descarga el payload desde la URL configurada directamente a la memoria.
5.  **Descifrado y Preparación**: El payload se decodifica de Base64. Si AES está habilitado, se descifra.
6.  **Ejecución**: El payload final se inyecta en la memoria y se ejecuta mediante llamadas sigilosas a la API de Windows.

---

## 🔧 Configuración y Despliegue

### 1. Configuración del Payload
- **URL**: Modifica la URL codificada en Base64 en el hashtable `$cfg` (`$cfg.o`).
- **Formato del Payload**: El payload debe estar en formato de shellcode puro, que luego se codifica en Base64.

### 2. Cifrado AES (Opcional)
Para usar la capacidad de descifrado AES:
1.  **Cifra tu payload**: Usa tu herramienta preferida para cifrar tu shellcode con AES-256 (modo CBC, padding PKCS7).
2.  **Configura la Clave y el IV**:
    - Codifica en Base64 tu clave de cifrado de 32 bytes y ponla en `$cfg.r`.
    - Codifica en Base64 tu IV de 16 bytes y ponlo en `$cfg.s`.
3.  **Codifica en Base64 el payload cifrado** y alójalo en tu URL.
4.  **Activa el Descifrado**: Descomenta las tres líneas en la sección de manejo del payload del script para activar la rutina de descifrado.

### 3. Despliegue
El script es autónomo. Puede ser ejecutado a través de cualquier método de invocación de PowerShell estándar. En su primera ejecución en una máquina objetivo, establecerá automáticamente sus mecanismos de persistencia.

---

## ⚠️ Descargo de Responsabilidad

Esta herramienta está destinada únicamente a operaciones autorizadas de red teaming, investigación de seguridad y fines educativos. El uso no autorizado de este framework contra cualquier sistema es ilegal. Los desarrolladores no asumen ninguna responsabilidad y no son responsables de ningún mal uso o daño causado por este programa.