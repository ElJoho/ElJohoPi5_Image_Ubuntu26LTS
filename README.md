# ElJohoPi5_Image_Ubuntu26LTS

**Autor:** Johan Alejandro López Arias ([@ElJoho](https://github.com/ElJoho))

Este repositorio contiene un backup del sistema de la Raspberry Pi 5 usado para restablecer la imagen del proyecto **UNAL PhantomX Pincher** ([ElJoho_KIT_Phantom_X_Pincher_ROS2](https://github.com/ElJoho/ElJoho_KIT_Phantom_X_Pincher_ROS2)).

| Dato | Valor |
|---|---|
| Sistema | Ubuntu Desktop 26.04.1 LTS (arm64) |
| ROS 2 | Lyrical |
| Usuario / contraseña | `unpi` / `unpi` |
| IP de la Raspberry Pi (cable Ethernet) | `192.168.10.2` |
| Escritorio remoto | RDP (Escritorio remoto de GNOME): ver [Guides/conexionEscritorioRemoto.md](Guides/conexionEscritorioRemoto.md) |
| Versión de la imagen | 2, del 26 de noviembre de 2026 |

El backup final corresponde a una imagen comprimida:

```text
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
```

Debido a su tamaño, el archivo fue dividido en **4 partes** y publicado en la sección **Releases** de este repositorio (no dentro del repositorio, así que **no se necesita Git ni Git LFS** para descargarlo).

Las partes se llaman:

```text
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-001
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-002
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-003
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-004
```

> **Importante:** cada parte por separado **no** es un archivo `.xz` válido.
> Primero debes descargar correctamente las 4 partes y luego unirlas para reconstruir el archivo:
>
> ```text
> ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
> ```

No es necesario descomprimir manualmente el archivo `.img.xz`.
**Raspberry Pi Imager puede usar el archivo `.img.xz` directamente.**

---

## 1) Programas necesarios en Windows 11

Para restaurar el sistema desde este backup necesitas:

1. **Un navegador web**
   Se usa para descargar las 4 partes desde la sección Releases.

2. **CMD como administrador**
   Se usa para unir las 4 partes con el comando `copy /b` y para comprobar el archivo con `certutil`.

3. **Raspberry Pi Imager**
   Se usa para grabar la imagen `.img.xz` directamente en la microSD o SSD.

---

## 2) Descargar las 4 partes

### Opción A: con el navegador

1. Abre la página de la última versión: **[Releases → última versión](https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest)**.
2. En la sección **Assets**, haz clic en cada uno de estos archivos para descargarlos:

   | Archivo | Enlace directo |
   |---|---|
   | `ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-001` | [descargar](https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-001) |
   | `ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-002` | [descargar](https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-002) |
   | `ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-003` | [descargar](https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-003) |
   | `ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-004` | [descargar](https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-004) |
   | `SHA256SUMS.txt` (opcional, para comprobar) | [descargar](https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/SHA256SUMS.txt) |

3. Guarda las 4 partes en una misma carpeta, por ejemplo `C:\Users\user_name\Downloads\backup_pi`.

### Opción B: con CMD

Windows 11 ya trae `curl`. En **CMD** (no en PowerShell), entra a la carpeta donde quieres guardar las partes y ejecuta:

```bat
cd /d "C:\Users\user_name\Downloads\backup_pi"
curl -L -O https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-001
curl -L -O https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-002
curl -L -O https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-003
curl -L -O https://github.com/ElJoho/ElJohoPi5_Image_Ubuntu26LTS/releases/latest/download/ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-004
```

---

## 3) Verificar que los archivos se descargaron correctamente

En CMD, dentro de la carpeta de las partes:

```bat
dir
```

Cada parte debe pesar exactamente **1.217.997.032 bytes** (aproximadamente **1,22 GB**).

En el Explorador de Windows aparece como:

```text
1.189.451 KB
```

El punto es separador de miles, no significa que el archivo pese solo 1 KB.

Si una parte pesa menos, la descarga se interrumpió: bórrala y vuelve a descargarla.

---

## 4) Unir las 4 partes usando CMD

Para unir las partes en Windows se debe usar **CMD**.

> No uses `copy /b` directamente en PowerShell.
> En PowerShell, `copy` es un alias de `Copy-Item`, por eso el comando puede fallar.

---

### 4.1 Abrir CMD como administrador

1. Abre el menú de inicio.
2. Busca `cmd`.
3. Haz clic derecho sobre **Símbolo del sistema**.
4. Selecciona **Ejecutar como administrador**.

---

### 4.2 Entrar a la carpeta de las partes

En CMD, entra a la carpeta donde están las 4 partes.

Ejemplo:

```bat
cd /d "C:\Users\user_name\Downloads\backup_pi"
```

Verifica que las partes estén ahí:

```bat
dir
```

Debes ver estos archivos:

```text
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-001
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-002
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-003
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-004
```

---

### 4.3 Unir las partes

Ejecuta el siguiente comando en **una sola línea**:

```bat
copy /b /y ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-001+ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-002+ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-003+ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-004 ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
```

El orden es importante. Las partes deben unirse desde:

```text
part-001
```

hasta:

```text
part-004
```

Cuando el proceso termine, se creará el archivo:

```text
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
```

Verifica que el archivo final existe:

```bat
dir
```

El archivo reconstruido debe pesar exactamente **4.871.988.128 bytes** (aproximadamente **4,9 GB**; en el Explorador de Windows: `4.757.801 KB`).

---

### 4.4 Comprobar el archivo unido (recomendado)

```bat
certutil -hashfile ElJohoUNPI_UbuntuDesktop2026LTS.img.xz SHA256
```

La línea del medio de la respuesta debe ser exactamente:

```text
50791e4d71d1f0f4b105366741a74320ad963b40828241e4e5ac13992cdc4ccb
```

Si es distinta, alguna parte está dañada o incompleta: revisa los tamaños (sección 3), vuelve a descargar la parte que no coincida y vuelve a unir.

---

### 4.5 En Linux (alternativa)

En una terminal, dentro de la carpeta de las partes (con `SHA256SUMS.txt` descargado en la misma carpeta):

```bash
cat ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-00{1..4} > ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
sha256sum -c --ignore-missing SHA256SUMS.txt     # todas las líneas deben decir OK
```

---

## 5) Grabar la imagen con Raspberry Pi Imager

No descomprimas manualmente el archivo `.img.xz`.

Aunque el archivo expandido ocupa alrededor de **16,7 GB**, no necesitas generar manualmente el `.img`.
**Raspberry Pi Imager puede grabar directamente el archivo comprimido `.img.xz`.**

El archivo que debes usar es:

```text
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
```

No selecciones las partes individuales.

---

### 5.1 Abrir Raspberry Pi Imager como administrador

1. Abre el menú de inicio.
2. Busca **Raspberry Pi Imager**.
3. Haz clic derecho sobre el programa.
4. Selecciona **Ejecutar como administrador**.

Esto es recomendable porque Raspberry Pi Imager necesita permisos para formatear y escribir sobre la microSD o SSD.

---

### 5.2 Seleccionar el modelo de Raspberry Pi

En Raspberry Pi Imager:

1. Selecciona el modelo de Raspberry Pi.
2. Para este backup, selecciona **Raspberry Pi 5**.

---

### 5.3 Seleccionar la imagen personalizada

En la sección del sistema operativo:

1. Baja hasta el final de la lista.
2. Selecciona **Usar personalizado** o **Use custom**.
3. Busca el archivo reconstruido:

```text
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
```

4. Selecciónalo.

---

### 5.4 Seleccionar la microSD o SSD

Selecciona la microSD o SSD donde quieres restaurar el sistema.

> **Advertencia:** este proceso borrará todos los datos del dispositivo seleccionado.

Se recomienda usar una microSD o SSD de al menos **32 GB**. En el primer arranque, el sistema agranda automáticamente su partición para usar todo el espacio de la tarjeta.

---

### 5.5 Escribir la imagen

Haz clic en **Siguiente**.

Si Raspberry Pi Imager pregunta si quieres aplicar ajustes de personalización del sistema operativo (*OS customisation*), elige **No**: la imagen ya trae su configuración (usuario, IP y escritorio remoto).

Luego haz clic en **Write / Escribir**.

Raspberry Pi Imager escribirá la imagen y después realizará una verificación.

Este proceso puede tardar un rato, dependiendo de la velocidad de la microSD o SSD y del lector usado.

---

## 6) Iniciar la Raspberry Pi 5

Cuando Raspberry Pi Imager termine:

1. Expulsa la microSD o SSD de forma segura.
2. Conéctala a la Raspberry Pi 5.
3. Enciende la Raspberry Pi.

Como esta es una imagen restaurada del sistema anterior, la configuración del sistema ya está incluida. El primer arranque puede tardar un poco más, porque agranda la partición.

Para ver el escritorio de la Raspberry Pi desde un computador con Windows o Linux, conéctala por **cable Ethernet** y sigue la guía **[Conexión con Escritorio remoto (RDP)](Guides/conexionEscritorioRemoto.md)**: IP `192.168.10.2`, usuario `unpi`, contraseña `unpi`.

**Repositorio del kit:** para saber qué repositorio clonar y cómo hacerlo, abre el archivo `~/ros2/readme.txt` de la Raspberry Pi, por ejemplo en una terminal:

```bash
cat ~/ros2/readme.txt
```

Si la Raspberry Pi no aparece inmediatamente:

1. Espera un momento mientras termina de iniciar.
2. Verifica que esté encendida.
3. Revisa que el ventilador se active.
4. Revisa la IP del PC (sección 2 de la guía de escritorio remoto) e intenta conectarte nuevamente.

En algunos casos puede ser necesario reiniciar la Raspberry Pi 5 usando el botón físico:

1. Mantén presionado el botón hasta que cambie el estado de la luz.
2. Vuelve a presionarlo para encenderla.
3. Espera a que el sistema inicie correctamente.

---

## 7) Problemas comunes

### Una parte pesa menos de lo esperado

La descarga se interrumpió. Bórrala y vuelve a descargarla desde la sección Releases (sección 2).

Cada parte debe pesar exactamente **1.217.997.032 bytes**.

---

### `copy /b` no funciona

Asegúrate de estar usando **CMD como administrador**.

No ejecutes `copy /b` directamente en PowerShell.

El comando correcto debe ejecutarse en CMD:

```bat
copy /b /y ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-001+ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-002+ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-003+ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.part-004 ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
```

---

### El archivo `.xz` queda dañado

Revisa lo siguiente:

* Debes tener exactamente **4 partes**.
* Cada parte debe pesar exactamente **1.217.997.032 bytes**.
* Debes unirlas en orden desde `part-001` hasta `part-004`.
* No debes intentar abrir o descomprimir una parte individual.
* El SHA-256 del archivo unido debe coincidir con el de la sección 4.4.
* El archivo final debe llamarse:

```text
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
```

---

### Raspberry Pi Imager no muestra el archivo

Asegúrate de haber seleccionado:

```text
Use custom
```

o:

```text
Usar personalizado
```

El archivo que debes seleccionar es:

```text
ElJohoUNPI_UbuntuDesktop2026LTS.img.xz
```

No selecciones las partes individuales.

---

### Raspberry Pi Imager pide formatear la microSD o SSD

Es normal. El proceso de escritura borrará el contenido del dispositivo.

Por eso es importante verificar que seleccionaste la microSD o SSD correcta antes de continuar.

---

## 8) Resumen rápido del proceso

```text
1. Abrir la sección Releases de este repositorio.
2. Descargar las 4 partes en una misma carpeta.
3. Verificar que cada parte pese 1.217.997.032 bytes.
4. Abrir CMD como administrador.
5. Entrar a la carpeta de las partes.
6. Unir las partes con copy /b.
7. (Recomendado) Comprobar el archivo con certutil -hashfile ... SHA256.
8. Abrir Raspberry Pi Imager como administrador.
9. Seleccionar Use custom / Usar personalizado.
10. Seleccionar ElJohoUNPI_UbuntuDesktop2026LTS.img.xz.
11. Grabar la imagen en una microSD o SSD de al menos 32 GB.
12. Insertar la microSD o SSD en la Raspberry Pi 5 e iniciar el sistema.
13. Conectarse por escritorio remoto con la guía Guides/conexionEscritorioRemoto.md.
```

---

## 9) Guías de este repositorio

* **[Conexión con Escritorio remoto (RDP)](Guides/conexionEscritorioRemoto.md):** cómo ver y controlar el escritorio de la Raspberry Pi desde Windows o Linux, por cable Ethernet.
* **[Guide 1 — Initial setup](Guides/01-initial-setup.md)** (en inglés): cómo se preparó la imagen: Ubuntu Desktop 26.04.1, red por cable, escritorio remoto sin monitor y ventilador al 100 %.

---

<sub>**Contacto (solo estudiantes de la Universidad Nacional de Colombia):** si tienen dudas, pueden escribirme buscando mi nombre completo, *Johan Alejandro López Arias*, en el Gmail de la UNAL, o desde [johanalejandrolopezarias.com](https://johanalejandrolopezarias.com).</sub>
