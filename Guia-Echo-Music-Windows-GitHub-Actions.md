# Echo Music Desktop para Windows: compilar con GitHub Actions (Opción A)

## Paso 1: Crear el archivo del workflow

1. Abre tu repositorio en GitHub: `https://github.com/Zoel1975/Echo-Music-Desktop`
2. Pulsa el botón **Add file** y luego **Create new file**.
3. En el campo del nombre escribe exactamente esto (las `/` crean las carpetas automáticamente):

   ```
   .github/workflows/build-windows.yml
   ```

   > La carpeta `.github` ya existe en tu repo (tiene `FUNDING.yml`). Si al escribir `.github/` te lo reconoce, sigue con `workflows/build-windows.yml`.

4. En el área de texto pega **todo** el código de la sección [Código del archivo](#código-del-archivo).
5. Pulsa **Commit changes...** y luego **Commit changes** (puedes dejarlo en la rama principal).

---

## Paso 2: Verificar que Actions está habilitado

1. Entra a la pestaña **Actions** de tu repo.
2. Como es un fork, GitHub puede mostrar un aviso tipo *"Workflows aren't being run on this forked repository"*. Si lo ves, pulsa el botón verde **I understand my workflows, go ahead and enable them**.
3. En la columna izquierda debe aparecer **Build Windows**.

---

## Paso 3: Ejecutar la compilación

1. En **Actions**, selecciona **Build Windows** en la columna izquierda.
2. Pulsa **Run workflow** (a la derecha), deja la rama por defecto y pulsa el botón verde **Run workflow**.
3. Refresca la página. Aparecerá una ejecución con un círculo amarillo (en curso).
4. Espera entre **10 y 15 minutos** aproximadamente. Cuando termine verás un check verde.

---

## Paso 4: Descargar el programa

1. Entra a la ejecución terminada (haz clic en su nombre).
2. Baja hasta la sección **Artifacts**.
3. Descarga **EchoMusic-Windows**. Se baja un `.zip` que contiene otro `EchoMusic-Windows.zip`.
4. Descomprime todo en una carpeta de tu PC (por ejemplo `C:\EchoMusic`).
5. Abre la carpeta y ejecuta **`echo.exe`**.

> Importante: no muevas `echo.exe` solo a otro lugar. Necesita los archivos `.dll` y la carpeta `data` que están junto a él. Si quieres un acceso directo, haz clic derecho sobre `echo.exe` y elige **Enviar a > Escritorio (crear acceso directo)**.

Si Windows muestra la pantalla azul de SmartScreen ("Windows protegió su PC"), pulsa **Más información** y luego **Ejecutar de todas formas**. Aparece porque el programa no está firmado digitalmente.

---

## Opcional: publicar un Release en tu repo

Para que el `.zip` quede en la sección **Releases** (con enlace de descarga directo):

1. En tu PC, en una terminal dentro de la carpeta del repo, ejecuta:

   ```
   git tag v1.0.0
   git push origin v1.0.0
   ```

2. Esto lanza el workflow automáticamente. Al terminar, el `.zip` aparece en `Releases` de tu repo.

Alternativa sin terminal: en GitHub ve a **Releases > Draft a new release**, escribe `v1.0.0` en "Choose a tag", selecciona **Create new tag on publish** y pulsa **Publish release**.

---

## Código del archivo

Archivo: `.github/workflows/build-windows.yml`

```yaml
name: Build Windows

on:
  workflow_dispatch:      # botón "Run workflow" en la pestaña Actions
  push:
    tags:
      - "v*"              # al crear un tag como v1.0.0 también publica un Release

permissions:
  contents: write

jobs:
  build-windows:
    runs-on: windows-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Instalar Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true

      - name: Habilitar soporte de escritorio Windows
        run: flutter config --enable-windows-desktop

      - name: Dependencias
        run: flutter pub get

      - name: Compilar (release)
        run: flutter build windows --release

      - name: Comprimir en .zip
        shell: pwsh
        run: |
          Compress-Archive -Path build\windows\x64\runner\Release\* -DestinationPath EchoMusic-Windows.zip

      - name: Subir artefacto
        uses: actions/upload-artifact@v4
        with:
          name: EchoMusic-Windows
          path: EchoMusic-Windows.zip

      - name: Publicar Release (solo con tag)
        if: startsWith(github.ref, 'refs/tags/')
        uses: softprops/action-gh-release@v2
        with:
          files: EchoMusic-Windows.zip
```

---

## Solución de problemas

| Problema | Qué hacer |
|---|---|
| No aparece "Build Windows" en Actions | Revisa que el archivo esté exactamente en `.github/workflows/build-windows.yml` y que hayas habilitado los workflows (Paso 2). |
| No aparece el botón **Run workflow** | El archivo debe estar en la rama principal y contener `workflow_dispatch:`. Refresca la página. |
| La compilación falla (X roja) | Abre la ejecución, haz clic en el paso que falló y copia el error. Con ese mensaje se puede ajustar el workflow. |
| Error al publicar el Release | En el repo ve a **Settings > Actions > General > Workflow permissions** y activa **Read and write permissions**. |
| El programa no abre o falta una DLL | Asegúrate de haber descomprimido **toda** la carpeta, no solo `echo.exe`. |

---

## Nota sobre las actualizaciones

La app todavía consulta las actualizaciones del autor original (archivo `lib/services/update_service/update_service.dart`, variable `owner = 'iad1tya'`). Si más adelante quieres que apunte a tu propio repo, hay que cambiar esa configuración.

> Esta guía no fue probada con una ejecución real de GitHub Actions. Si algún paso falla, comparte el mensaje de error para corregirlo.
