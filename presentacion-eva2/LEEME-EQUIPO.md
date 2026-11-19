# Presentación de la defensa · EVA 2 (Grupo G07)

Presentación hecha con el template de Slidev de Sabrina. Son 16 diapositivas en 6 bloques, y cada una tiene su guion en las notas del orador.

## Requisitos (una sola vez)

- Node.js 22 o superior: https://nodejs.org
- pnpm: en PowerShell, ejecutar `npm i -g pnpm`

## Cómo abrirla

1. Clonar o descargar el repositorio.
2. Abrir PowerShell **dentro** de la carpeta `presentacion-eva2`.
3. Instalar las dependencias (solo la primera vez, tarda alrededor de un minuto):

   ```bash
   pnpm install
   ```

4. Iniciar la presentación:

   ```bash
   pnpm dev
   ```

   Se abre en el navegador en `http://localhost:3030`.

5. Modo presentador, con notas del orador, cronómetro y diapositiva siguiente: abrir `http://localhost:3030/presenter`.
6. Para cerrar: **Ctrl + C** en PowerShell.

## Atajos

| Tecla | Acción |
| --- | --- |
| → o espacio | Siguiente diapositiva |
| ← | Diapositiva anterior |
| F | Pantalla completa |
| O | Vista de todas las diapositivas |

## Dónde se edita

- Todo el contenido y las notas están en `src/slides.md`. Cada nota es el bloque `<!-- Guion (Bloque …) -->` al final de cada diapositiva.
- Las imágenes están en `src/assets/img/`.
- Los cambios se ven al instante mientras `pnpm dev` está corriendo.

## Exportar a PDF o PowerPoint

El template no descarga su propio navegador, así que se exporta usando Edge:

```bash
npx slidev export src/slides.md --output .reports/presentacion-eva2.pdf --executable-path "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
npx slidev export src/slides.md --format pptx --output .reports/presentacion-eva2.pptx --executable-path "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
```

## Problemas comunes

- **«pnpm no se reconoce»:** cerrar y volver a abrir PowerShell después de instalarlo.
- **Windows pide permiso de firewall:** aceptar «Redes privadas».
- **El puerto 3030 está ocupado:** Slidev usa otro número; revisar la dirección que muestra PowerShell.
