# Guía de Instalación y Configuración de Autofirma en Linux

Esta guía detalla paso a paso cómo configurar Autofirma correctamente en distribuciones Linux (específicamente centrado en **Fedora/Red Hat**, pero adaptable a otras). 

El proceso consta de 3 fases críticas: Instalación de dependencias (Java 11), Instalación del software y Configuración del sistema.

## Requisitos Previos

> [!WARNING]
> **Importante:** Autofirma presenta incompatibilidades con versiones recientes de Java. Es **obligatorio** utilizar **Java 11** para que funcione correctamente.

## Paso 1: Instalación de Java 11

Autofirma requiere una versión específica de Java. Recomendamos usar **Azul Zulu OpenJDK 11**.

1. Descarga el archivo `.rpm` de Java 11 desde el [sitio oficial de Azul](https://www.azul.com/downloads/?version=java-11-lts&os=linux&package=jdk) o usa este [enlace directo](https://cdn.azul.com/zulu/bin/zulu11.82.19-ca-jdk11.0.28-linux.x86_64.rpm) (versión probada).
2. Instala el paquete descargado:

```bash
sudo dnf install -y /ruta/donde/descargaste/zulu11*.rpm
```

3. Verifica la instalación:
```bash
java -version
```

## Paso 2: Instalación de Autofirma

1. Descarga la versión para Linux desde el [Portal de Administración Electrónica](https://firmaelectronica.gob.es/ciudadanos/descargas).
2. Descomprime el archivo `.zip`.
3. Instala el archivo `.rpm` resultante:

```bash
sudo dnf install -y /ruta/donde/esta/autofirma.rpm
```
4. *Opcional:* Puedes borrar los archivos `.rpm` una vez instalados para ahorrar espacio.

## Paso 3: Configuración del lanzador (.desktop)

Este es el paso más crítico donde suelen ocurrir los errores. Debemos asegurar que el acceso directo apunte al Java 11 que acabamos de instalar y no al del sistema por defecto.

1. **Localizar el archivo original:**
   Normalmente se instala en `/usr/share/applications/autofirma.desktop`.

2. **Crear una copia local para tu usuario:**
   Es mejor no editar el archivo del sistema directamente.

   ```bash
   mkdir -p ~/.local/share/applications 
   cp /usr/share/applications/autofirma.desktop ~/.local/share/applications/
   ```

   > [!IMPORTANT]
   > El archivo debe llamarse obligatoriamente `autofirma.desktop` y NO `afirma.desktop`.

3. **Editar el archivo .desktop:**
   Abre el archivo `~/.local/share/applications/autofirma.desktop` con tu editor favorito y busca la línea que empieza por `Exec=`. Debes reemplazarla por lo siguiente:

   ```ini
   Exec=/usr/lib/jvm/zulu-11/bin/java -Djdk.tls.maxHandshakeMessageSize=65536 -jar /usr/lib64/autofirma/autofirma.jar %u
   ```

   *Nota: Verifica que la ruta `/usr/lib/jvm/zulu-11/bin/java` existe. Si usas otra distribución (como Ubuntu), la ruta del jar podría ser `/usr/lib/autofirma/` en lugar de `/usr/lib64/`.*

## Paso 4: Verificación y Navegadores

### Prueba de ejecución
Intenta lanzar el programa desde la terminal para ver si hay errores:

```bash
# Opción 1 (Directo)
autofirma

# Opción 2 (Ruta completa manual si la 1 falla)
/usr/lib/jvm/zulu-11/bin/java -jar /usr/lib64/autofirma/AutoFirma.jar
```

### Integración con Firefox

Para firmar en webs, Autofirma debe comunicarse con el navegador mediante un certificado raíz.

1. Abre Firefox y ve a `Ajustes` -> `Privacidad y Seguridad`.
2. Baja hasta **Certificados** y pulsa en **Ver certificados**.
3. En la pestaña **Autoridades**, busca: `Autoridad de certificación del canal seguro de AutoFirma` o `Autofirma ROOT`.

**¿No aparece?**
1. Abre Autofirma.
2. Ve a `Herramientas` -> `Restaurar configuración de AutoFirma`.
3. Reinicia Firefox.

## Test Final
Para confirmar que todo funciona, visita la [Página de prueba de la Administración](https://www.sededgsfp.gob.es/es/Paginas/TestAutofirma.aspx).
