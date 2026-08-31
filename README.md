# Espejo mágico — actualizaciones

Este repo existe para una sola cosa: que la PC de un evento pueda bajar una
actualización sin credenciales. El código fuente del espejo está en un repo
privado; acá sólo viven los paquetes y el manifiesto.

`latest.json` es lo que la app consulta al abrir. Si la versión que dice es
más nueva que la que está corriendo, la consola muestra un aviso y el
operador decide cuándo aplicarla. **Nunca se actualiza sola ni durante un
evento.**

## Publicar una versión nueva

1. En el repo del código, subir `version:` en `pubspec.yaml` y `appVersion`
   en `lib/app_version.dart` (hay un test que falla si se desincronizan).
2. `flutter analyze && flutter test`
3. `flutter build windows --release`
4. Compilar el instalador:

```bash
"/c/Program Files (x86)/Inno Setup 6/ISCC.exe" //DMyAppVersion=<version> windows/installer/espejo_magico.iss
```

   Sale en `build/dist/espejo-magico-<version>-setup.exe`.

5. Sacar el hash:

```bash
sha256sum build/dist/espejo-magico-<version>-setup.exe
```

6. Crear el release acá con ese .exe adjunto:

```bash
gh release create v<version> build/dist/espejo-magico-<version>-setup.exe --repo lovera2025/magic-booth-releases --title "v<version>" --notes "Qué cambió"
```

7. Actualizar `latest.json` con la versión, la URL del adjunto, el hash y el
   tamaño en bytes, y commitearlo. **Recién en este paso les llega el aviso
   a las PCs**: hasta que `latest.json` no cambia, nadie ve nada.

El instalador se instala en `%LOCALAPPDATA%\Programs\Espejo magico`, sin
permisos de administrador — por eso la actualización automática puede
correrlo en silencio sin que aparezca ningún cartel de Windows.

Nunca borra `%APPDATA%\com.magicbooth`, que es donde viven los eventos y las
fotos. Tampoco al desinstalar.

Ese orden importa. Si se publica el manifiesto antes que el .zip, las PCs
van a intentar bajar algo que todavía no existe.

## Volver atrás

Si una versión salió mal, alcanza con dejar `latest.json` apuntando a la
anterior. Las PCs que ya actualizaron no vuelven solas — hay que bajarles el
.zip viejo a mano — pero las que todavía no lo hicieron dejan de ver el
aviso.
