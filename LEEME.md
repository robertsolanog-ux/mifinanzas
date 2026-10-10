# Mi Finanzas · App Android independiente

App que **no usa Google Sheets ni Apps Script**. Todo funciona dentro del teléfono:

- **Datos locales**: se guardan en el teléfono y funciona sin internet.
- **Cuenta de Google opcional**: si entras con Google, tus datos se respaldan en una carpeta **privada de tu Drive** (solo esta app la ve) y los recuperas en otro teléfono. Si eliges "Comenzar sin cuenta", todo queda solo en el teléfono.
- **Widget** de pantalla de inicio: disponible del mes, ingresos/gastos, próximo pago y botones **Ingreso** y **Gasto** que abren directo el diálogo de registro.
- **Botón en ajustes rápidos** ("Registrar") y atajos al mantener pulsado el ícono (Gasto / Ingreso).
- Tema claro / oscuro / sistema, igual que la versión web.

> Aviso honesto: este proyecto se probó en navegador (lógica, pantallas, respaldo simulado), pero no se pudo compilar aquí porque el entorno no tiene el Android SDK. La primera compilación en GitHub o Android Studio es la prueba real; si falla, copia el error y lo corregimos.

---

## 1. Obtener el APK (sin Android Studio) — GitHub Actions
1. Crea un repositorio **privado** en github.com (importante: privado, porque incluye tu llave de firma).
2. Sube **todo el contenido de esta carpeta** (incluida la carpeta oculta `.github` y el archivo `mifinanzas.jks`).
3. Entra a la pestaña **Actions** → **Compilar APK**. Corre sola al subir los archivos (o pulsa *Run workflow*).
4. Cuando salga el check verde (3 a 6 min), abre la ejecución y descarga **MiFinanzas-APK** en *Artifacts*. Es un .zip; adentro está `app-release.apk`.
5. Pásalo al teléfono e instálalo (permite "instalar apps desconocidas" para tu explorador de archivos).

### Alternativa: Android Studio
**File → Open** en esta carpeta → espera el *Gradle sync* → **Build → Build APK(s)** → **locate**.
Por consola, con Gradle 8.5: `gradle assembleRelease` (el APK queda en `app/build/outputs/apk/release/`).

---

## 2. Activar "Continuar con Google" (una sola vez, ~10 min, gratis)
Sin este paso la app funciona igual, pero el botón de Google mostrará un error. Necesitas una cuenta de Google.

1. Entra a **console.cloud.google.com** → crea un proyecto nuevo (ej. "Mi Finanzas").
2. **APIs y servicios → Biblioteca** → busca **Google Drive API** → **Habilitar**.
3. **Pantalla de consentimiento OAuth** (o *Google Auth Platform*):
   - Tipo de usuario: **Externo**. Nombre de la app: Mi Finanzas. Correo de asistencia: el tuyo.
   - **Acceso a los datos / Scopes**: agrega `.../auth/drive.appdata`, `email` y `profile`.
   - Mientras esté en **Prueba**, agrega tu correo (y el de quien la use) como *usuario de prueba*.
   - Recomendado: pulsa **Publicar aplicación**. Con estos permisos (no sensibles) no exige verificación, y evita que Google te pida volver a iniciar sesión cada 7 días.
4. **Credenciales → Crear credenciales → ID de cliente de OAuth** → tipo **Android**:
   - Nombre del paquete: `com.mifinanzas.app`
   - Huella SHA-1: `B0:09:A4:49:A0:EE:68:45:08:77:71:DB:BF:81:DD:68:5A:47:69:07`
5. Listo. No hay que copiar ningún ID dentro del código: Google reconoce la app por el paquete y la huella.

La huella SHA-1 corresponde a la llave `mifinanzas.jks` incluida, así que **no cambia entre compilaciones**. Si algún día generas otra llave, debes registrar su nueva SHA-1 en el paso 4.

---

## 3. Primer uso
1. Abre la app. Elige **Comenzar sin cuenta** o **Continuar con Google**.
2. **Widget**: mantén pulsada la pantalla de inicio → *Widgets* → **Mi Finanzas** (4×2; se puede redimensionar).
3. **Ajustes rápidos**: desliza dos veces desde arriba → lápiz → arrastra **Registrar**.
4. **Atajos**: mantén pulsado el ícono de la app → *Gasto* o *Ingreso*.
5. **Respaldo**: Más → Configuración → *Cuenta y respaldo* (respaldar ahora, restaurar, respaldo automático, cerrar sesión).
   En un teléfono nuevo: instala, toca *Continuar con Google* y elige **Restaurar copia de Drive**.

## 4. Cosas a tener en cuenta
- **Sin cuenta**: si desinstalas la app o borras sus datos, se pierde la información. Para evitarlo, entra con Google.
- El widget se actualiza cada vez que registras algo, al salir de la app y cada ~30 min. Al cambiar de mes pide abrir la app una vez.
- Si ya usabas la versión con Google Sheets, esta app **no** importa esos datos automáticamente (son independientes). Se puede añadir una importación aparte.
- Los datos van dentro del dispositivo; el respaldo de Drive es una copia completa (un solo archivo JSON) que se sobrescribe.

## 5. Estructura
```
app/src/main/assets/www/   index.html (interfaz) · engine.js (base de datos local) · native.js (puente Android)
app/src/main/java/.../     MainActivity.kt (WebView + Google) · Drive.kt (respaldo) · QuickWidget.kt · QuickTileService.kt
app/src/main/res/          widget, colores día/noche, atajos, ícono
mifinanzas.jks             llave de firma (mantén el repositorio privado)
.github/workflows/         build-apk.yml (compilación en GitHub)
```
