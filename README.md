# Codeanywhere
```markdown
# Configuración y Ejecución del Escritorio en la VM de Codeanywhere

Este documento explica, paso a paso, cómo configurar y arrancar un entorno gráfico (LXDE) en tu VM de Codeanywhere utilizando TigerVNC, así como cómo acceder a él (vía un cliente VNC o mediante noVNC en el navegador). Además, se incluyen los pasos para solucionar errores comunes y qué comandos ejecutar cada vez que reinicies la VM.

---

## Requisitos Previos

- **VM en Codeanywhere:** Basada en Debian: https://app.codeanywhere.com/
- **Acceso a la terminal de la VM.**
- **Conexión a internet** para actualizar e instalar paquetes.
- **Persistencia de la VM:** Recuerda que los contenedores en Codeanywhere pueden ser efímeros; asegúrate de revisar la documentación de la plataforma para mantener tus cambios si es necesario.

---

## Paso 1: Actualizar el Sistema e Instalar Paquetes Necesarios

Actualiza el sistema e instala LXDE (un entorno gráfico ligero) y TigerVNC (servidor VNC):

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install lxde tigervnc-standalone-server -y
```

---

## Paso 2: Configurar el Entorno Gráfico para VNC

### 2.1 Crear el Archivo `.Xresources`

Este archivo se carga al iniciar X; créalo si no existe para evitar errores.

```bash
touch ~/.Xresources
```

### 2.2 Configurar el Archivo de Inicio de VNC (`~/.vnc/xstartup`)

Este archivo define los comandos que se ejecutan al arrancar la sesión VNC.

1. Abre el archivo en un editor (por ejemplo, nano):

   ```bash
   nano ~/.vnc/xstartup
   ```

2. Reemplaza su contenido con lo siguiente:

   ```sh
   #!/bin/sh
   [ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
   export XKL_XMODMAP_DISABLE=1
   lxsession -s LXDE -e LXDE
   ```

   - **Explicación:**
     - `#!/bin/sh` indica que se ejecutará con el intérprete sh.
     - La línea `[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources` carga el archivo de recursos X si existe.
     - `export XKL_XMODMAP_DISABLE=1` ayuda a prevenir problemas de mapeo de teclado.
     - `lxsession -s LXDE -e LXDE` inicia una sesión LXDE.

3. Guarda el archivo:
   - Presiona **Ctrl+O** y luego **Enter** para guardar.
   - Sal del editor con **Ctrl+X**.

4. Haz que el archivo sea ejecutable:

   ```bash
   chmod +x ~/.vnc/xstartup
   ```

---

## Paso 3: Iniciar el Servidor VNC

Cada vez que reinicies la VM, deberás iniciar manualmente el servidor VNC.

1. **(Opcional) Si hay alguna sesión anterior, mátala:**

   ```bash
   vncserver -kill :1
   ```

2. **Inicia el servidor VNC en el display `:1`:**

   ```bash
   vncserver :1 -geometry 1280x720 -depth 24
   ```

   - Esto arranca la sesión VNC en el display `:1`, que corresponde al puerto **5901** (5900 + 1).
   - Si todo está configurado correctamente, verás un mensaje indicando que el servidor se inició.
   - Para verificar la sesión activa, usa:

     ```bash
     vncserver -list
     ```

   Deberías ver algo similar a:

   ```
   TigerVNC server sessions:
   :1    vscode (pid XXXX)
   ```

   *Nota:* Si en arranque previo aparecía el error “Session startup … exited too early”, se solucionó configurando correctamente el archivo `xstartup` y asegurándote de que tenga permisos ejecutables.

---

## Paso 4: Acceder al Escritorio

Tienes dos opciones para conectarte a tu entorno gráfico:

### Opción A: Usar un Cliente VNC

1. **Instala un cliente VNC** en tu equipo local, por ejemplo, [RealVNC Viewer](https://www.realvnc.com/en/connect/download/viewer/).
2. **Obtén la IP de tu VM:**  
   Ejecuta en la terminal:

   ```bash
   hostname -I
   ```

3. **Conéctate al servidor VNC:**  
   En el cliente, usa la dirección `IP:5901` (por ejemplo, `192.168.0.10:5901`).
4. **Ingresa la contraseña:**  
   Usa la contraseña que configuraste la primera vez que iniciaste el VNC.

### Opción B: Usar noVNC para Acceso vía Navegador

1. **Clona el repositorio noVNC** (si no lo tienes aún):

   ```bash
   git clone https://github.com/novnc/noVNC.git
   cd noVNC
   ```

2. **Inicia el proxy web de noVNC:**  
   En versiones recientes, se utiliza el comando `novnc_proxy`:

   ```bash
   ./utils/novnc_proxy --vnc localhost:5901
   ```

   - Esto levanta un servidor web en el puerto **6080**.
3. **Accede desde tu navegador:**  
   Abre tu navegador y dirígete a:

   ```
   http://<IP_DE_TU_VM>:6080
   ```

   Si estás trabajando en la misma máquina, puedes usar `127.0.0.1:6080`.

---

## Paso 5: Persistencia y Repetición del Proceso

Los contenedores en Codeanywhere pueden ser efímeros, lo que significa que al reiniciar la VM se perderán los procesos en ejecución y, posiblemente, las instalaciones (a menos que tengas configurados volúmenes persistentes). Por ello, **cada vez que reinicies la VM deberás**:

1. **Iniciar el servidor VNC:**

   ```bash
   vncserver :1 -geometry 1280x720 -depth 24
   ```

2. **(Opcional) Iniciar noVNC** para acceso vía navegador:

   ```bash
   cd /workspaces/base-debian/noVNC
   ./utils/novnc_proxy --vnc localhost:5901
   ```

3. **Conectarte** desde tu cliente VNC o navegador usando la IP y puerto correspondiente.

*Consejo:* Revisa la documentación de Codeanywhere para ver si puedes configurar la persistencia (volúmenes o snapshots) y evitar tener que reiniciar manualmente cada vez.

---

## Conclusión

Para arrancar y acceder a tu escritorio en la VM de Codeanywhere, sigue estos pasos:

1. **Configura el entorno:**
   - Actualiza el sistema e instala LXDE y TigerVNC.
   - Crea el archivo `.Xresources` y configura `~/.vnc/xstartup` para iniciar LXDE.
2. **Arranca el servidor VNC:**

   ```bash
   vncserver :1 -geometry 1280x720 -depth 24
   ```

3. **Accede al escritorio:**
   - Usa un cliente VNC para conectarte a `IP:5901` **o**
   - Usa noVNC ejecutando:

     ```bash
     cd /workspaces/base-debian/noVNC
     ./utils/novnc_proxy --vnc localhost:5901
     ```
     
     Y luego accede a `http://<IP_DE_TU_VM>:6080` desde tu navegador.

Cada vez que reinicies la VM, repite estos pasos para arrancar el entorno gráfico. Esta guía resume todos los pasos, incluyendo la solución a errores comunes (como problemas de permisos en `xstartup` y la ausencia de `.Xresources`), para que puedas arrancar y utilizar tu escritorio en la VM de manera consistente.

Si tienes alguna otra duda o necesitas ayuda adicional, ¡no dudes en consultarme!
```
