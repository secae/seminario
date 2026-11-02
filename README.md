## ⚙️ Funcionamiento del Sistema de Certificados

El portal opera bajo una arquitectura estática en **GitHub Pages**. Para cada consulta aplica una **única secuencia de resolución** con dos pasos en orden de prioridad: primero busca un **PDF ya emitido** y, solo si no existe, **genera el diploma** desde el listado `nombres.csv`. Ya no hay que elegir ni detectar un "modo" por seminario: ambos recursos pueden convivir en la misma carpeta.

---

### 🔍 Secuencia de Resolución

Al ingresar la identificación y pulsar **Buscar Certificado**:

1.  **Paso 1 – PDF directo:** el portal busca `[Identificación].pdf` dentro de la carpeta del seminario.
2.  **Paso 2 – Generación desde CSV:** si el PDF no existe, el portal consulta `nombres.csv` y, si encuentra a la persona, genera el diploma sobre `diploma.png`.
3.  **Sin resultado:** si no se halla ni el PDF ni el registro en el CSV, se muestra la interfaz de error.

> **Ventaja:** en la misma carpeta se pueden subir diplomas individuales de **profesores, instructores u otros tipos** (nombrados con su identificación), y el resto de participantes se sigue generando automáticamente desde el CSV.

---

### 🪪 Campo de Identificación (Cédula o Pasaporte)

*   Acepta **números y letras**, por lo que sirve para cédulas y pasaportes (ej.: `1234567890`, `AB123456`).
*   Mientras se escribe se eliminan automáticamente espacios y símbolos; solo se conservan letras y números (máximo 30 caracteres).
*   **Mayúsculas y minúsculas:** GitHub Pages distingue entre ellas en los nombres de archivo. Por eso, para el PDF directo el portal prueba tres variantes: tal como se escribió, en MAYÚSCULAS y en minúsculas. Se recomienda nombrar los PDF con letras en **mayúscula** (ej.: `AB123456.pdf`).
*   En la búsqueda dentro del CSV la comparación **no distingue mayúsculas ni minúsculas**, ni guiones o espacios.

---

### 🗂️ Gestión de Visibilidad de Seminarios (Omitir u Ocultar)
El listado dinámico del portal se alimenta de las carpetas de la raíz del repositorio. Si deseas archivar o marcar un **seminario como obsoleto** para que no aparezca en el buscador del portal:
*   **Mecanismo:** renombra la carpeta anteponiendo un guion medio (`-`) al inicio de su nombre (por ejemplo: `-2023` o `-Seminario_Viejo`).
*   **Comportamiento:** el cargador inicial omite automáticamente cualquier directorio que comience con este prefijo, de modo que solo los seminarios vigentes sean accesibles para consulta pública.

---

### 🔹 Paso 1: PDF Directo (`[Identificación].pdf`)
Se usa cuando el documento individual ya fue emitido y cargado en la carpeta del seminario.

*   **Mecanismo:** el portal realiza una petición HTTP `HEAD` a la ruta:
    `https://secae.github.io/seminario/[Nombre_Seminario]/[Identificación].pdf`
*   **Resolución:**
    *   Si el archivo existe (`200 OK`), se carga en el visor embebido (`<iframe>`) y el usuario puede **visualizar, descargar o imprimir** el documento original.
    *   Si no existe, el sistema continúa con el Paso 2.

---

### 🔹 Paso 2: Generación Dinámica (Imagen Base + Lista CSV)
Se ejecuta cuando no se encontró el PDF directo y la carpeta contiene `nombres.csv` y `diploma.png`. Renderiza los diplomas en tiempo real en el navegador, evitando almacenar un PDF por persona.

*   **Estructura del CSV (`nombres.csv`):**
    texto plano separado por punto y coma (`;`):
    ```csv
    desfase;[Valor_Numérico_Píxeles]
    [Identificación_1];[Nombre_Completo_1]
    [Identificación_2];[Nombre_Completo_2]
    ```
    *La fila con la palabra clave `desfase` es opcional y puede ubicarse en cualquier parte del archivo. Define el desplazamiento vertical del texto (positivo para bajar, negativo para subir) para todos los diplomas del seminario y evita pisar elementos del diseño base.*
    *La primera columna puede contener cédulas o pasaportes (letras y números).*
*   **Mecanismo de Renderizado:**
    1.  **Validación:** se busca la identificación ingresada en el CSV (omitiendo encabezados y filas de control).
    2.  **Procesamiento gráfico:** si se encuentra el registro, se descarga en segundo plano la plantilla obligatoria `diploma.png`.
    3.  **Inyección de texto:** con `<canvas>` se escribe el nombre en mayúscula sostenida y, debajo, la identificación, aplicando el desfase vertical y un interlineado de `0.8`. La identificación se rotula **`C.C.`** si es solo numérica o **`ID`** si contiene letras (pasaportes).
    4.  **Exportación:** el lienzo se convierte en imagen comprimida y se encapsula en un PDF con `jsPDF`. El usuario lo visualiza en pantalla y puede descargarlo como PDF sin elementos de la interfaz web.
*   **Consola del navegador:** al generar, se imprime un mensaje de diagnóstico con el seminario y el desfase aplicado.

---

### 📋 Resumen rápido

| Situación en la carpeta del seminario | Resultado |
|---|---|
| Existe `[Identificación].pdf` | Se muestra ese PDF (tiene prioridad) |
| No existe el PDF, pero la persona está en `nombres.csv` | Se genera el diploma con `diploma.png` |
| No existe el PDF ni el registro en el CSV | Mensaje de error |
