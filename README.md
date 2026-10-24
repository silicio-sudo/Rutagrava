# RutaGrava

App Android para **ciclistas**: busca una ruta en bicicleta y te dice **cuántos metros hay de
asfalto, grava, pista de tierra y suelo mixto**, con la **información de tráfico** de las vías
por las que pasa.

> Versión 1.0 · buscador de rutas + desglose de superficie (sin grabación GPS ni navegación
> turn-by-turn).

---

## Qué hace

1. **Escribes el punto de partida y el destino** (buscador con autocompletado) o **mantienes
   pulsado el mapa** para fijarlos con el dedo.
2. Pulsas **CALCULAR** y la app traza la ruta con BRouter (servicio gratuito sobre OpenStreetMap).
3. La ruta se dibuja **coloreada por tipo de terreno** y aparece un panel con:

   | Contenido | Ejemplo |
   |---|---|
   | Distancia, tiempo y desnivel | `28,4 km · 1 h 42 min · +620 m` |
   | Barra y lista de superficie con metros y % | `Grava — 12.340 m — 43 %` |
   | Tráfico | `Tráfico intenso · 18,4 km de vía rodada · 5,8 km con riesgo de retención` |
   | Alternativas | `1ª / 2ª / 3ª` (otras variantes de la misma ruta) |

4. Puedes **Guardar** la ruta en el móvil, exportarla en **GPX** (se abre con cualquier visor o
   app de navegación) o **borrarla**.

Si tocas un tramo de la ruta se abren las **etiquetas OSM** de esa vía (`surface`, `highway`,
`tracktype`…) con las que se ha calculado.

---

## De dónde salen los datos

| Dato | Servicio | Coste |
|---|---|---|
| Cálculo de ruta y etiquetas de superficie | **BRouter** `brouter.de/brouter` | gratuito, sin clave |
| Buscador de lugares y nombres al pulsar el mapa | **Nominatim (OpenStreetMap)** | gratuito, sin clave |
| Mapa base | **osmdroid 6.1.20** (tiles OSM) | gratuito |
| Tráfico real en vivo | **TomTom Flow + Incidents** | requiere clave (viene una por defecto) |

### Cómo se calcula la superficie

Cada tramo de la ruta llega con sus etiquetas OSM (`WayTags`). El orden de prioridad es
**`surface` → `tracktype` → `highway`**:

- **Asfalto** — `surface=asphalt, concrete, paved, chipsal/chipsing, hardened, metal`, o una vía
  carretera (`motorway…residential, service, road`) sin etiqueta de superficie.
- **Grava** — `surface=gravel, fine_gravel, shingle, pebblestone, compacted, rock/stones`, y
  caminos con `tracktype=grade1|grade2`.
- **Pista** — `highway=track` sin `tracktype` fino, `surface=dirt, earth, clay, mud, wood,
  woodchip`, y cualquier vía con `tracktype=grade3|grade4|grade5` (sin preparar).
- **Suelo mixto** — `surface=unpaved, ground, grass, sand`, adoquines (`sett, cobblestone,
  paving_stones, tiles`), sendas, pistas de bicis, escaleras…
- **Sin datos** — tramos sin etiquetas utilizables.

El orden de prioridad es **`surface` → `tracktype` → `highway`**.

La regla completa está en la propia app: botón **«¿Cómo se calcula?»**.

### Tráfico: estimado y real

- **Estimado (por defecto):** combina el tipo de vía de OSM con la hora punta (7–10 y 17–20 en
  días laborables) y si la vía está en un núcleo urbano. No consume API key.
- **Real (opcional, con clave):** consulta el tráfico en vivo de TomTom sobre la vía rodada de la
  ruta (hasta 10 puntos cada ~1,5 km en **Flow**: velocidad actual vs. velocidad sin tráfico) y
  añade las **incidencias** del entorno. Se usa automáticamente si hay clave en *Ajustes*.

---

## Compilar y ejecutar

Requisitos: **JDK 17+** (Gradle 9.3.1 no funciona con JRE 8/11), Android SDK y `local.properties`
apuntando al SDK.

```powershell
$env:JAVA_HOME = "$env:LOCALAPPDATA\Temp\opencode\jdk\jdk-21.0.12.1+1"
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"

cd "C:\Users\help\Documents\Proyecto predeterminado\RutaGrava"
.\gradlew.bat assembleDebug      # APK de pruebas
.\gradlew.bat assembleRelease    # APK firmado para distribuir
```

- Debug: `app\build\outputs\apk\debug\app-debug.apk`
- Release: `app\build\outputs\apk\release\app-release.apk`

El release se firma con `rutagrava.keystore` (contraseña en `keystore.properties`). **Si publicas
el código o el APK en un repositorio público, cambia la clave de TomTom** en *Ajustes* → no
compartmentes la tuya. `keystore.properties` y `*.keystore` deben quedar fuera del repo.

---

## Cómo se usa

| Acción | Cómo |
|---|---|
| Fijar origen/destino | Escribe en los campos y elige de la lista, o mantén pulsado el mapa |
| Cambiar el origen por tu posición | Botón **Mi posición** (pide permiso de ubicación) |
| Calcular | Botón **CALCULAR** |
| Ver otra variante | `1ª` `2ª` `3ª` del panel |
| Ver las etiquetas OSM de un tramo | Toca ese tramo de la ruta |
| Guardar / GPX / borrar | Botones del panel |
| Rutas guardadas | Botón **Guardadas** (pulsa para abrir, mantén pulsado para borrar) |
| Perfil o clave de TomTom | Botón **Ajustes** |

### Perfiles de BRouter

- **Mixta (trekking)** — busca el equilibrio, admite pistas y grava.
- **Rápida (fastbike)** — prioriza asfalto y vías rápidas.

---

## Estructura del proyecto

```
RutaGrava/
├── app/src/main/java/com/rutagrava/app/
│   ├── MainActivity.java          mapa, buscador, panel de superficie y tráfico
│   ├── GuardadasActivity.java     lista de rutas guardadas
│   ├── BRouterClient.java         cálculo de ruta + reparto de tramos con WayTags
│   ├── Superficie.java            reglas de clasificación (asfalto/grava/pista/mixto)
│   ├── Trafico.java               estimación y tráfico real de TomTom
│   ├── NominatimClient.java       geocodificación
│   ├── Ruta.java                  modelo, formato y guardado local (JSON)
│   ├── Gpx.java                   generación y compartición de GPX
│   ├── Net.java                   peticiones HTTP
│   └── Preferencias.java          clave TomTom y perfil
└── app/src/main/res/              layouts, colores, textos y drawables
```
