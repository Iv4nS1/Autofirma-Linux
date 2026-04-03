# Configuración de firma personalizada

Al firmar digitalmente un documento (como un contrato o presupuesto), AutoFirma permite incrustar una estampa visual en las páginas del PDF. Para lograr un acabado realista, necesitamos unos preparativos previos.

## 1. Archivos Necesarios y Preparación

Es obligatorio contar con una imagen de tu firma manuscrita en formato `.png` que contenga el fondo transparente. El archivo debe llamarse preferiblemente `firma.png`.

Nota: en el repositorio tienes el archivo `firma.svg` que puedes modificar cambiando la firma manuscrita por la tuya y añadiendo tu nombre y apellidos. NO modifiques la altura ni la anchura del archivo. Exportalo a `.png` cuando hayas terminado de editarlo.

**Cómo crear tu firma transparente:**
1. Escribe tu firma real en un papel blanco completamente liso usando un bolígrafo de tinta negra o azul.
2. Escanea el papel o tómale una fotografía bien iluminada (sin sombras).
3. Utiliza herramientas gratuitas en línea (como remove.bg) o editores de imagen (como GIMP o INKSCAPE) para eliminar el fondo blanco y dejar solo el trazo.
4. Exporta el resultado como un archivo `.png` con canal alfa (transparencia).


## Pasos a seguir

- Abrimos el programa autofirmar
- Seleccionamos el archivo a firmar
- Seleccionamos **hacer la firma visible dentro del pdf**
- Pulsamos **firmar** y **aceptar**
- Elegimos la página donde queremos que se vea la firma y hacemos un rectángulo en el lugar
- Propiedades parte superior (x e y lo dejamos como está) **Anchura: 32** y **Altura: 15** pulsamos siguiente
- Imagen: **seleccionamos la imagen firma.png**
- Fuente: **Courier**, tamaño: **6 pt**, **no rotar**, **negrita** y color **Rosa**
- Recordar configuración: **activado**