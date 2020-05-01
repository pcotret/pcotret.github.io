# Image processing


## Convert an SVG to PNG in commande line using Inkscape (on Windows)
Change export-height if you want a bigger image:

```batch
@echo off
for %%f in (%*) do (
    echo %%~f
    "C:\Program Files\Inkscape\inkscape.exe" ^
      -z ^
      --export-background-opacity=0 ^
      --export-height=48 ^
      --export-png="%%~dpnf.png" ^
      --file="%%~f"

)
```

## Resize an image with Inkscape
```powershell
for %%f in (*.png) do convert.exe %%f -resize 384x384 -background black -gravity center -extent 384x384 %%f
```
