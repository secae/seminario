## ⚙️ Modos de Funcionamiento del Sistema de Certificados

El portal opera bajo una arquitectura estática en **GitHub Pages** y resuelve la visualización o generación de diplomas mediante dos flujos lógicos diferenciados (*Caso 1* y *Caso 2*). El sistema detecta automáticamente qué modo utilizar realizando una inspección previa de los archivos disponibles en la ruta seleccionada.

---

### 🗂️ Gestión de Visibilidad de Seminarios (Omitir u Ocultar)
El listado dinámico del portal se alimenta de las carpetas de la raíz del repositorio. Si deseas archivar o marcar un **seminario como obsoleto** para que no aparezca en el buscador del portal:
*   **Mecanismo:** Debes renombrar la carpeta anteponiendo un guion medio (`-`) al inicio de su nombre (por ejemplo: `-2023` o `-Seminario_Viejo`).
*   **Comportamiento:** El cargador inicial filtrará y omitirá automáticamente de la interfaz cualquier directorio que comience con este prefijo, garantizando que solo los seminarios vigentes sean accesibles para consulta pública[cite: 1].

---

### 🔹 Caso 1: Búsqueda por Diploma PDF Directo
Este modo se activa cuando el seminario ya cuenta con los documentos individuales previamente emitidos y cargados en formato PDF.

*   **Detección:** Al seleccionar el seminario, el sistema verifica la ausencia de archivos de configuración masiva y asume el flujo de documentos individuales.
*   **Mecanismo:** Cuando el usuario ingresa su número de cédula, el portal realiza una petición HTTP `HEAD` apuntando directamente a la ruta:
    `https://sucpafac.github.io/seminario/[Nombre_Seminario]/[Número_Cédula].pdf`
*   **Resolución:**
    *   Si el archivo existe (`200 OK`), se carga de forma nativa en el visor embebido (`<iframe>`) de la página, permitiendo al usuario utilizar las herramientas del navegador para **visualizar, descargar o imprimir** el documento original.
    *   Si el archivo no se encuentra, el sistema activa la interfaz de error.

---

### 🔹 Caso 2: Generación Dinámica (Imagen Base + Lista CSV)
Este modo se activa de forma automática si el sistema detecta la presencia del archivo centralizado de datos `nombres.csv` dentro de la carpeta del seminario. Está diseñado para mitigar el consumo de almacenamiento renderizando los diplomas en tiempo real en el lado del cliente (navegador).

*   **Detección y Consola:** El sistema valida la existencia de `nombres.csv`. Al confirmarse, imprime en la consola del navegador el mensaje de diagnóstico detallando el modo activo y los parámetros de altura recuperados de la cabecera.
*   **Estructura del CSV (`nombres.csv`):**
    El archivo debe estar codificado en texto plano, separado por punto y coma (`;`) bajo la siguiente estructura adaptada a la configuración regional:
    ```csv
    desfase;[Valor_Numérico_Píxeles]
    [Cédula_1];[Nombre_Completo_1]
    [Cédula_2];[Nombre_Completo_2]
    ```
    *La fila que contiene la palabra clave `desfase` es una línea de control opcional. Puede ubicarse en cualquier parte del archivo; el sistema la detectará dinámicamente y aplicará ese desplazamiento vertical (positivo para bajar, negativo para subir) a todos los diplomas de ese seminario para evitar pisar elementos del diseño base.*
*   **Mecanismo de Renderizado:**
    1.  **Validación:** El portal busca la cédula ingresada dentro del mapa del CSV (omitiendo encabezados o filas de control).
    2.  **Procesamiento Gráfico:** Si se encuentra el registro, el sistema descarga en segundo plano la plantilla base llamada obligatoriamente `diploma.png`.
    3.  **Inyección de Texto:** Utilizando el elemento `<canvas>` de HTML5, se monta la imagen y se escribe el nombre del alumno (en mayúscula sostenida) y su cédula de identidad, aplicando el desfase vertical indicado y un interlineado compacto optimizado de `0.8` para máxima armonía estética.
    4.  **Exportación:** El lienzo resultante se convierte en una imagen comprimida y se encapsula instantáneamente dentro de un documento PDF limpio mediante la librería `jsPDF`. El usuario visualiza el resultado final en pantalla y puede exportarlo como un archivo PDF legítimo sin que se observe ningún otro elemento de la interfaz web.
