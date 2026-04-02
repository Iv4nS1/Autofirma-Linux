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

   El archivo completo debería quedar algo así:

```bash
[Desktop Entry]
Encoding=UTF-8
Version=1.0
Type=Application
Terminal=false
Categories=Office;Utilities;Signature;Java;
Exec=/usr/lib/jvm/zulu-11/bin/java -Djdk.tls.maxHandshakeMessageSize=65536 -jar /usr/lib64/autofirma/autofirma.jar %u
Name=Autofirma
Icon=/usr/lib64/autofirma/autofirma.png
GenericName=Herramienta de firma
Comment=Herramienta de firma
MimeType=x-scheme-handler/afirma;
```

## Paso 4: Verificación y Navegadores

### Prueba de ejecución
Intenta lanzar el programa desde la terminal para ver si hay errores:

```bash
# Opción 1 (Directo)
autofirma

# Opción 2 (Ruta completa manual si la 1 falla)
/usr/lib/jvm/zulu-11/bin/java -jar /usr/lib64/autofirma/AutoFirma.jar
```
### Registrar el handler del protocolo `afirma://` 

Crear el `.desktop` con el `MimeType` no es suficiente. Hay que decirle al sistema explícitamente que ese archivo es el gestor del protocolo `afirma://` y actualizar la base de datos de aplicaciones:

```bash
# Actualizar la base de datos de MIME
update-desktop-database ~/.local/share/applications/

# Registrar AutoFirma como handler del protocolo afirma://
xdg-mime default autofirma.desktop x-scheme-handler/afirma
```

Verifica que quedó registrado:

```bash
xdg-mime query default x-scheme-handler/afirma
# Debe devolver: autofirma.desktop
```

Y comprueba que en el archivo `~/.config/mimeapps.list` aparece:

```ini
[Default Applications]
x-scheme-handler/afirma=autofirma.desktop
```
### Integración con Google Chrome
Con los pasos que hemos realizado tendría que funcionar todo perfectamente con google Chrome version 146.

### Integración con Firefox
La versión de Firefox usada es la última en este momento versión 149 en wayland, lo cual nos da algunos problemas extra que tenemos que solventar.

Para firmar en webs, Autofirma debe comunicarse con el navegador mediante un certificado raíz.

1. Abre Firefox y ve a `Ajustes` -> `Privacidad y Seguridad`.
2. Baja hasta **Certificados** y pulsa en **Ver certificados**.
3. En la pestaña **Autoridades**, busca: `Autoridad de certificación del canal seguro de AutoFirma` o `Autofirma ROOT`.

**¿No aparece?**
1. Abre Autofirma.
2. Ve a `Herramientas` -> `Restaurar configuración de AutoFirma`.
3. Reinicia Firefox.

####  Instalar extensión firefox

Las páginas web de firma intenta  inyectar un **iframe** a través de la url de  `afirma://sign?...`. **Firefox bloquea los protocolos personalizados en iframes** — Vivaldi/Chrome los permite. 
No podemos configurarlo directamente, tenemos que instalar lo siguiente:

- Instalamos  **Tampermonkey** desde `about:addons`
- Haz clic en el icono de Tampermonkey → **Crear nuevo script**
- Creamos el siguiente script:
```script
// ==UserScript== 
// @name AutoFirma iframe fix (universal) 
// @match https://*.gob.es/* 
// @match https://*.administracion.es/* 
// @match https://*.redsara.es/* 
// @match https://*.sepe.es/* 
// @match https://*.aeat.es/* 
// @match https://*.seg-social.es/* 
// @match https://*.ayto-santander.es/* 
// @match https://*.santander.es/* 
// @match https://*.educantabria.es/* 
// @match https://*.cantabria.es/* 
// @run-at document-start 
// @grant GM_openInTab 
// ==/UserScript==

(function() {
    'use strict';

    function lanzarAfirma(url) {
        console.log('[AutoFirma Fix] Lanzando:', url);
        
        // Método 1: link con click
        const a = document.createElement('a');
        a.href = url;
        a.rel = 'opener';
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        
        // Método 2: GM_openInTab como fallback
        setTimeout(() => GM_openInTab(url, {active: false}), 300);
    }

    const observer = new MutationObserver(function(mutations) {
        mutations.forEach(function(mutation) {
            mutation.addedNodes.forEach(function(node) {
                if (node.nodeType === 1) {
                    // Comprobar el nodo directamente
                    if (node.tagName === 'IFRAME') {
                        const src = node.getAttribute('src') || node.src || '';
                        if (src.startsWith('afirma://')) {
                            console.log('[AutoFirma Fix] Iframe detectado:', src);
                            node.remove();
                            lanzarAfirma(src);
                        }
                    }
                    // Comprobar iframes dentro del nodo
                    node.querySelectorAll && node.querySelectorAll('iframe[src^="afirma://"]').forEach(function(iframe) {
                        const src = iframe.src;
                        console.log('[AutoFirma Fix] Iframe anidado detectado:', src);
                        iframe.remove();
                        lanzarAfirma(src);
                    });
                }
            });
        });
    });

    // Arrancar observer en cuanto haya body
    function iniciarObserver() {
        const target = document.body || document.documentElement;
        observer.observe(target, { childList: true, subtree: true });
        console.log('[AutoFirma Fix] Observer activo en', target.tagName);
    }

    if (document.body) {
        iniciarObserver();
    } else {
        document.addEventListener('DOMContentLoaded', iniciarObserver);
    }

})();
```

- Guardamos con **ctrl + s**

- Añadir los dominios según necesites en la cabezacera, modificando lo siguiente con las urls que necesites.

```script
// ==UserScript==
// @name         AutoFirma iframe fix (universal)
// @match        https://*.gob.es/*
// @match        https://*.administracion.es/*
// @match        https://*.redsara.es/*
// @match        https://*.sepe.es/*
// @match        https://*.aeat.es/*
// @match        https://*.seg-social.es/*
// @match        https://*.ayto-santander.es/*
// @match        https://*.santander.es/*
// @match        https://*.educantabria.es/*
// @match        https://*.cantabria.es/*
// @run-at       document-start
// @grant        GM_openInTab
// ==/UserScript==
```

Así cualquier persona con Fedora + Firefox + Wayland podrá reproducirlo sin horas de depuración.

## Test Final
Para confirmar que todo funciona, visita la [Página de prueba de la Administración](https://www.sededgsfp.gob.es/es/Paginas/TestAutofirma.aspx).

## Direcciones de interés

- [wiki autofirma](https://github.com/ctt-gob-es/clienteafirma/wiki/)
- [Test de firma más completos](https://valide.redsara.es/firmaMovil/)