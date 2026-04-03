# Guía de Instalación y Configuración de AutoFirma en Linux

Esta guía detalla paso a paso cómo configurar AutoFirma correctamente en distribuciones Linux. Está específicamente centrada en **Fedora/Red Hat**, pero la lógica es fácilmente adaptable a otras distribuciones. 

El proceso consta de 3 fases críticas: Instalación de dependencias (Java 11), Instalación del software y Configuración del sistema.

## Requisitos Previos e Incompatibilidades

**Importante:** AutoFirma presenta incompatibilidades conocidas con versiones recientes de Java. Es **obligatorio** utilizar **Java 11** para que la comunicación de los sockets funcione correctamente. 

## Paso 1: Instalación de Java 11

AutoFirma requiere una versión específica de Java. Recomendamos usar **Azul Zulu OpenJDK 11** por su estabilidad.

1. Descarga el archivo `.rpm` de Java 11 desde el [sitio oficial de Azul](https://www.azul.com/downloads/?version=java-11-lts&os=linux&package=jdk).
2. Alternativamente, usa este [enlace directo](https://cdn.azul.com/zulu/bin/zulu11.82.19-ca-jdk11.0.28-linux.x86_64.rpm) a la versión probada.
3. Instala el paquete descargado en tu terminal ejecutando: `sudo dnf install -y /ruta/donde/descargaste/zulu11*.rpm`.
4. Verifica que la instalación es correcta ejecutando: `java -version`.

## Paso 2: Instalación de AutoFirma

1. Descarga la versión más reciente para Linux desde el [Portal de Administración Electrónica](https://firmaelectronica.gob.es/ciudadanos/descargas).
2. Descomprime el archivo `.zip` que acabas de descargar.
3. Instala el archivo `.rpm` resultante ejecutando: `sudo dnf install -y /ruta/donde/esta/autofirma.rpm`.
4. Opcionalmente, puedes borrar los archivos `.rpm` e instaladores una vez finalizado el proceso para ahorrar espacio en disco.

## Paso 3: Configuración del lanzador (.desktop)

Este es el paso más crítico donde suelen ocurrir los errores de ejecución. Debemos asegurar que el acceso directo apunte al Java 11 que acabamos de instalar y no a otra versión de Java por defecto.

1. Localiza el archivo original del sistema, que normalmente se instala en `/usr/share/applications/autofirma.desktop`.
2. Crea el directorio de aplicaciones local de tu usuario ejecutando: `mkdir -p ~/.local/share/applications`.
3. Copia el lanzador a tu entorno local para no editar el sistema: `cp /usr/share/applications/autofirma.desktop ~/.local/share/applications/`.
4. **Atención:** El archivo copiado debe llamarse obligatoriamente `autofirma.desktop` y NO `afirma.desktop`.
5. Abre el archivo `~/.local/share/applications/autofirma.desktop` con tu editor favorito y reemplaza la línea que empieza por `Exec=` por: `Exec=/usr/lib/jvm/zulu-11/bin/java -Djdk.tls.maxHandshakeMessageSize=65536 -jar /usr/lib64/autofirma/autofirma.jar %u`.

*Nota para otras distribuciones: Verifica que la ruta `/usr/lib/jvm/zulu-11/bin/java` existe. Si usas distribuciones basadas en Debian/Ubuntu, la ruta final del jar podría ser `/usr/lib/autofirma/` en lugar de `/usr/lib64/`.*

## Paso 4: Verificación del Sistema y Gestor de Protocolos

Intenta lanzar el programa desde la terminal para confirmar que la interfaz gráfica carga sin errores de Java:
`autofirma`.
Si el comando falla, intenta con la ruta completa:
`/usr/lib/jvm/zulu-11/bin/java -jar /usr/lib64/autofirma/AutoFirma.jar`.

Crear el `.desktop` modificado no es suficiente para la web. Hay que decirle al sistema explícitamente que ese archivo es el gestor del protocolo `afirma://`. 

1. Actualiza la base de datos de aplicaciones de tu usuario: `update-desktop-database ~/.local/share/applications/`.
2. Registra AutoFirma como el gestor por defecto: `xdg-mime default autofirma.desktop x-scheme-handler/afirma`.
3. Comprueba el registro lanzando la consulta: `xdg-mime query default x-scheme-handler/afirma` (Debe devolver: `autofirma.desktop`).

## Paso 5: Integración con Navegadores

### Google Chrome
Con los pasos anteriores realizados, AutoFirma tendría que funcionar perfectamente con Google Chrome (versión 146 o superior probada).

### Firefox y Entornos Wayland
La versión de Firefox evaluada es la versión 149 ejecutándose nativamente bajo Wayland. Esto genera problemas adicionales de seguridad y bloqueo de protocolos que debemos solventar. Puedes confirmar si usas Wayland ejecutando en terminal `echo $XDG_SESSION_TYPE`.

Para firmar en webs, Firefox necesita confiar en el certificado de AutoFirma.
1. Abre Firefox y navega a `Ajustes` -> `Privacidad y Seguridad`.
2. Baja hasta la sección **Certificados** y pulsa en **Ver certificados**.
3. En la pestaña **Autoridades**, busca: `Autoridad de certificación del canal seguro de AutoFirma` o `Autofirma ROOT`.

Si el certificado no aparece en Firefox:
1. Abre la aplicación de AutoFirma en tu escritorio.
2. Ve al menú `Herramientas` -> `Restaurar configuración de AutoFirma`.
3. Cierra y reinicia Firefox completamente.

### Solución Definitiva para Iframes en Firefox (Extensión Tampermonkey)

Las páginas de la administración intentan inyectar un **iframe** oculto a través de la URL de `afirma://sign?...`. A diferencia de Chrome, **Firefox bloquea estrictamente los protocolos personalizados dentro de iframes** por motivos de seguridad.

Para solucionarlo, usaremos un script de usuario:
1. Instala la extensión **Tampermonkey** en Firefox desde el gestor de complementos (`about:addons`).
2. Haz clic en el icono de la extensión Tampermonkey y selecciona **Crear nuevo script**.
3. Copia y pega el código que aparece más abajo y guárdalo pulsando **Ctrl + S**.
4. Si necesitas acceder a otras sedes electrónicas, simplemente añade más líneas `@match` en la cabecera del script.

```javascript
// ==UserScript== 
// @name AutoFirma iframe fix (universal) 
// @match https://*.gob.es/* // @match https://*.administracion.es/* // @match https://*.redsara.es/* // @match https://*.sepe.es/* // @match https://*.aeat.es/* // @match https://*.seg-social.es/* // @match https://*.ayto-santander.es/* // @match https://*.santander.es/* // @match https://*.educantabria.es/* // @match https://*.cantabria.es/* // @run-at document-start 
// @grant GM_openInTab 
// ==/UserScript==

(function() {
    'use strict';

    function lanzarAfirma(url) {
        console.log('[AutoFirma Fix] Lanzando:', url);
        
        const a = document.createElement('a');
        a.href = url;
        a.rel = 'opener';
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        
        setTimeout(() => GM_openInTab(url, {active: false}), 300);
    }

    const observer = new MutationObserver(function(mutations) {
        mutations.forEach(function(mutation) {
            mutation.addedNodes.forEach(function(node) {
                if (node.nodeType === 1) {
                    if (node.tagName === 'IFRAME') {
                        const src = node.getAttribute('src') || node.src || '';
                        if (src.startsWith('afirma://')) {
                            console.log('[AutoFirma Fix] Iframe detectado:', src);
                            node.remove();
                            lanzarAfirma(src);
                        }
                    }
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
'''

## Test Final
Para confirmar que todo funciona, visita la [Página de prueba de la Administración](https://www.sededgsfp.gob.es/es/Paginas/TestAutofirma.aspx).

## Direcciones de interés

- [wiki autofirma](https://github.com/ctt-gob-es/clienteafirma/wiki/)
- [Test de firma más completos](https://valide.redsara.es/firmaMovil/)