# Murther in the Limbo — Correcciones sugeridas

Revisión técnica del prototipo de **Five Star Studio** sobre `main` (`0dea7b8`). Este documento describe **solo los cambios de la revisión** (rama `Correcciones-sugeridas`, integrada en `main` el 25/11/2026). El reporte de avances del equipo está en [`Docs/ProyectoIntegrador_Avances_FiveStarStudio.pdf`](ProyectoIntegrador_Avances_FiveStarStudio.pdf).

Todo se hizo y verificó en **Unity 6000.3.23f1**: compilación sin errores ni advertencias (C# y shaders), pruebas automáticas en Play Mode (entrada, física de Murther, Build Settings y carga de escenas) y capturas desde las mismas vistas en las dos versiones. **Falta probarlo con un control físico, jugarlo de principio a fin a mano y compilar el ejecutable.** Cada commit corresponde a un bloque, así que cualquiera se puede revisar o revertir por separado.

| Commit | Bloque |
|---|---|
| `5f05e40` | 1. Proyecto: render pipeline, entrada, Build Settings, nombre y `.gitignore` |
| `9350745` | 2. Herramientas de editor que regeneraban escenas solas |
| `c92da21` | 3. Flujo y menú |
| `e09f147` | 4. Ajustes visuales |
| `5568768` | 5. Documentos en `Docs/` y el reporte |
| `e1c25bc` | 6. Revisión gráfica: shader de tinta para Murther |
| `83d78d6` | 7. Revisión gráfica: post procesado y gotas de tinta |
| `bf66903` | 8. Iluminación de los fondos 2D |
| `b597c9d` | 8. Iluminación: Murther bajo la misma luz que el escenario |
| *(último)* | README y capturas actualizados |
| *(integración)* | Reporte movido a `Docs/Correcciones.md` y `.gitignore` contra carpetas de Unity dentro de `Assets` (sección 11) |

## Comparación con la versión original

Capturas renderizadas en Unity, en Play Mode, a 1280×720, con el mismo recorrido en las dos versiones: Murther avanza 4 s a la derecha y hace doble salto.

| Original (`main` en `0dea7b8`) | Con las correcciones |
|---|---|
| **Nivel 1 al iniciar** | |
| ![Nivel 1 al iniciar, original](Comparacion/original_4_nivel1_inicio.jpg) | ![Nivel 1 al iniciar, corregido](Comparacion/corregido_4_nivel1_inicio.jpg) |
| **Nivel 1 corriendo (aparece un Corredor)** | |
| ![Nivel 1 corriendo, original](Comparacion/original_5_nivel1_corriendo.jpg) | ![Nivel 1 corriendo, corregido](Comparacion/corregido_5_nivel1_corriendo.jpg) |
| **Doble salto en la orilla derecha** | |
| ![Doble salto, original](Comparacion/original_6_nivel1_doble_salto.jpg) | ![Doble salto, corregido](Comparacion/corregido_6_nivel1_doble_salto.jpg) |
| **Nivel 2** | |
| ![Nivel 2, original](Comparacion/original_7_nivel2.jpg) | ![Nivel 2, corregido](Comparacion/corregido_7_nivel2.jpg) |
| **Nivel 3** | |
| ![Nivel 3, original](Comparacion/original_8_nivel3.jpg) | ![Nivel 3, corregido](Comparacion/corregido_8_nivel3.jpg) |

En la versión original los niveles se ven **negros** y Murther **magenta**: el proyecto no tenía asignado el render pipeline URP (bloque 1). El menú, que es solo interfaz, se ve igual en las dos versiones.

Para separar ese error de los ajustes visuales, aquí está Murther de cerca en tres pasos: **solo con URP asignado**, con los ajustes del **bloque 4** (giro y luz) y con la **revisión gráfica** (bloques 6 y 7):

| Solo con URP asignado | Bloque 4 | Bloques 6 y 7 (estado final) |
|---|---|---|
| ![Murther quieto, solo URP](Comparacion/soloURP_murther_cerca.jpg) | ![Murther quieto, bloque 4](Comparacion/bloque4_murther_cerca.jpg) | ![Murther quieto, final](Comparacion/corregido_murther_cerca.jpg) |
| ![Murther corriendo, solo URP](Comparacion/soloURP_murther_corriendo.jpg) | ![Murther corriendo, bloque 4](Comparacion/bloque4_murther_corriendo.jpg) | ![Murther corriendo, final](Comparacion/corregido_murther_corriendo.jpg) |

- **Cara:** de perfil estricto el casco de calavera es un rectángulo blanco. Girado 35° hacia la cámara se le ven los ojos y los dientes hacia los dos lados.
- **Luz:** con URP/Lit el casco se quemaba en blanco puro; el bloque 4 le devolvió el volumen.
- **Estilo:** con el shader de tinta Murther deja de verse como un modelo 3D "de plástico" pegado sobre un dibujo: tiene sombra malva con rayado como las columnas del fondo, contorno de tinta y brillos de tinta húmeda en el cuerpo.

## 1. Proyecto: render pipeline, entrada, Build Settings, nombre y `.gitignore`

Estos tres primeros errores hacían que el juego no funcionara al abrirlo en otra computadora.

| Cambio | Por qué |
|---|---|
| **URP asignado** (`GraphicsSettings` → `PC_RPAsset`) | No había ningún render pipeline asignado, así que Unity usaba Built-in. Los materiales de Murther (URP/Lit) salían **magenta** y los fondos (Sprite-Unlit-Default de URP) **no se dibujaban**. |
| **Active Input Handling → Input System** | Estaba en *Input Manager (Old)*, pero todos los scripts leen `Keyboard.current` y `Gamepad.current` del Input System nuevo. Esos valores eran `null` (0 dispositivos): **Murther no respondía** al teclado ni al control. |
| **Build Settings con las 8 escenas** | La lista estaba vacía, así que `SceneManager.LoadScene` no encontraba ninguna escena y **"Iniciar Juego" dejaba la pantalla en negro**. Orden: MenuPrincipal → Cinematica → Controles_Nivel1 → Nivel1 → … → Nivel3. |
| `PlayerController3D` (prototipo anterior, en `Assets/Assets`) pasa al Input System | Usaba `Input.GetAxis`/`GetKey`, que fallan con el Input Manager desactivado. Se conservan las mismas teclas (y se agregan los botones del control) por si el equipo reutiliza el dash o el ataque. |
| Menús del prototipo anterior en **Juego → Prototipo anterior** | Se mezclaban con las herramientas actuales; *Poner fondo 2D* o *Poner mis modelos 3D* modificaban la escena abierta. |
| `productName` "Juego_Unity" → **"Murther in the Limbo"**; `companyName` → "Five Star Studio" | Es el nombre de la ventana y del ejecutable. |
| Se restaura el **`.gitignore`** de Unity (+ `.DS_Store`) | Se borró en el último commit de `main`. Sin él, la siguiente subida incluiría `Library/` (varios GB), `Logs/` y `UserSettings/`. |
| `unity-mcp` fijado a un commit | Seguía la rama `main` del repositorio: cada integrante podía bajar una versión distinta del paquete. |

## 2. Herramientas de editor que regeneraban escenas solas

`ConstruirFlujoDelJuego` y `ConfiguradorFase1` tenían `[InitializeOnLoadMethod]` y guardaban la marca de "ya se ejecutó" en `EditorPrefs`, que **es local a cada computadora**. La primera vez que alguien abría el proyecto en otra máquina:

- `ConstruirFlujoDelJuego` **volvía a crear todas las escenas desde cero**, incluidos los niveles sin Murther, sin suelo y sin enemigos.
- `ConfiguradorFase1` borraba y recreaba el Animator Controller, los materiales y a Murther con **GUIDs nuevos**, y rehacía las pantallas de carga.

Se perdían los cambios hechos a mano, y cada integrante podía subir versiones distintas de las mismas escenas.

| Cambio | Por qué |
|---|---|
| Las dos herramientas solo corren desde el menú **Juego** y **piden confirmación** | Ya no se ejecutan solas al abrir el proyecto. |
| *Construir flujo de escenas* ya no toca los niveles ni las pantallas de carga que existen; solo crea las que faltan | Antes sobrescribía el trabajo de *Fase 1* con niveles vacíos. |
| Nuevo **Juego → Registrar escenas en Build Settings** | Actualiza la lista sin regenerar nada. |
| Los generadores usan los mismos valores que el bloque 4 | Así volver a ejecutarlos no deshace las correcciones visuales. |

## 3. Flujo y menú

| Cambio | Por qué |
|---|---|
| `MenuPrincipalUI`: al abrir un panel se selecciona su primer botón (en *Salir*, "Cancelar") | Ningún botón quedaba seleccionado, así que **con control o con las flechas no se podía usar el menú**, aunque el juego está pensado para control (pantalla de carga). |
| `MenuPrincipalUI`: **Esc** o **B** cierran la ventana de salir y regresan de Créditos | Solo se podía salir de esas pantallas con el ratón. |
| `MenuPrincipalUI`: doble clic en "Iniciar Juego" ya no reinicia el fundido | |
| `BotonResaltado`: el ratón mueve la selección | Evita que queden dos botones resaltados a la vez (uno con el control y otro bajo el cursor). |
| `FlujoFader`: si una escena no está en Build Settings, avisa en la consola y vuelve a mostrar la escena actual | Antes la pantalla se quedaba **negra para siempre** y bloqueaba los clics. Era justo lo que pasaba en `main` al pulsar "Iniciar Juego". |
| `FlujoFader`: carga asíncrona y tope de 1/30 s por cuadro en el fundido | El cuadro largo que sigue a una carga se comía el fundido de entrada y la escena aparecía de golpe. |
| `IntroLogos`: cualquier tecla, clic, **A** o **Start** salta el video del logo | Había que verlo completo cada vez. |
| `CinematicaComic`: **Start** o **B** la saltan | Con control no había forma de saltarla; solo con Esc o con el ratón. |

## 4. Ajustes visuales

| Cambio | Por qué |
|---|---|
| `MurtherController`: nuevo **`giroHaciaCamara`** (35°) | De perfil estricto el casco de calavera se veía como un rectángulo blanco sin cara (ver capturas). Ahora se ven los ojos y los dientes hacia los dos lados, y al voltear pasa por el frente. Con 0 vuelve al perfil. |
| Luz del Nivel 1: 1.3 → **1.05**, cálida, desde el frente y arriba; ambiente plano 0.55 → **degradado** cielo/horizonte/suelo | La luz anterior, el ambiente plano y un casco casi blanco sumaban más de 1: la cabeza se **quemaba en blanco puro**. Además, la luz lateral iluminaba distinto según hacia dónde mirara Murther. |
| Material del casco: 93 % → **80 %** de blanco | Deja margen para que se vean la luz y la sombra. Los brillos de los ojos siguen al 93 %. |
| Paredes invisibles de los tres niveles **0.7 m hacia dentro** | El casco es más ancho que el collider: al llegar a la orilla, la mitad de Murther quedaba fuera de cuadro. |
| Fondos y arte del menú: **mipmaps** con filtro trilineal y compresión de **alta calidad** (BC7) | Los fondos miden ~3900 px y se muestran reducidos ~1.7× en 1080p. Sin mipmaps las líneas de tinta **parpadeaban** al mover la cámara, y la compresión normal (DXT) les ponía bloques. |
| `PC_RPAsset`: **MSAA 4×** | Estaba desactivado: los bordes 3D de Murther se veían dentados sobre el dibujo. |
| `MurtherController`: el estirado de respaldo respeta la escala original de `Visual` | Solo se usa si no hay hueso de cabeza; antes fijaba la escala Y en 1. |

**Para volver al perfil estricto:** en `Murther` → `MurtherController`, poner *Giro Hacia Camara* en 0.

> El material del casco y la luz de este bloque los reemplazó después el shader de tinta (bloque 6), que tiene sus propios valores.

## 5. Documentos

- `Readme.md` era una carpeta con el PDF de avances; ahora el PDF está en `Docs/`, para que `README.md` pueda ser un archivo (en macOS y Windows los dos nombres chocan).
- El video `WhatsApp Video 2026-11-12 at 5.13.23 PM.mp4` de la raíz pasó a `Docs/Video_avance_2026-11-12.mp4`.

## 6. Revisión gráfica: shader de tinta para Murther

Los fondos están dibujados a mano con tinta: sombras planas en pocos tonos, rayado en las zonas oscuras y contornos negros. Murther usaba **URP/Lit**, un sombreado suave y realista que lo hacía ver como un modelo 3D pegado encima del dibujo. El nuevo shader **`Five Star/Murther Tinta`** (`Assets/Shaders/MurtherTinta.shader`) lo dibuja con las mismas reglas que el escenario.

| Mirando a la derecha | Mirando a la izquierda |
|---|---|
| ![Murther con shader de tinta, derecha](Comparacion/tinta_derecha.jpg) | ![Murther con shader de tinta, izquierda](Comparacion/tinta_izquierda.jpg) |

| Qué hace | Por qué |
|---|---|
| **Sombreado por bandas** (luz / sombra) con sombra **malva** | Como el sombreado posterizado del fondo. El malva sale de las sombras de los muros y rocas del escenario. |
| **Trama de tinta** en la sombra y **trama cruzada** en la sombra profunda | Imita el rayado de las columnas. Las líneas viajan con Murther, no se quedan pegadas a la pantalla. Tienen un temblor leve de trazo a mano y 7 líneas por metro (≈13 px entre líneas en 1080p): más densas producían moiré. |
| **Contorno negro** de grosor constante | El contorno de tinta de todo el dibujo. Se hace inflando una copia de la malla (casco invertido). |
| **Borde de luz** y **brillo duro** en cuerpo y manos | El cuerpo es negro y se perdía contra las zonas negras del fondo. El brillo parece tinta húmeda. |
| **Luz de personaje** que gira con Murther (`MurtherController` → *Luz Tinta*) | Con la luz fija del mundo no se puede tener la cara iluminada mirando a los dos lados: hacia uno de ellos quedaba en sombra. Ahora la cara siempre está en luz y el costado en sombra con trama. |
| `NormalesSuavesContorno.cs` (Editor) | Al importar los FBX de `Assets/Murther/Modelos`, guarda normales promediadas en los tangentes. Con las normales originales el contorno se abría en las esquinas duras del cuerpo y los pies. **Los FBX no se modifican**: solo cambian los datos importados. |
| Pases de sombra, profundidad y normales | Murther sigue apareciendo en el prepass de SSAO y puede proyectar sombra si se activa en la luz. |

Los cuatro materiales de `Assets/Murther/Materiales` conservan su GUID: la escena no cambió de referencias. Además, `Murther_fullblack` tenía color de emisión pero no la palabra clave `_EMISSION`, así que el "brillo propio" del cuerpo **nunca se aplicaba**.

**Cómo ajustarlo** (en cada material, sin tocar código):

| Propiedad | Para qué |
|---|---|
| *Color de la sombra* | Tono de la parte en sombra (multiplica al color base). |
| *Umbral luz / sombra* | Más alto = más superficie en sombra. |
| *Trama → Intensidad, Líneas por metro, Grosor* | Cantidad y densidad del rayado. Intensidad 0 lo quita. |
| *Borde de luz*, *Brillo duro* | Separación del cuerpo negro contra el fondo. |
| *Contorno → Grosor* | En metros; 0 lo quita. |

La dirección de la luz de personaje se ajusta en `Murther` → `MurtherController` → *Luz Tinta* (frente, arriba, hacia la cámara), y su color en *Color Luz Tinta* (ver sección 8).

## 7. Revisión gráfica: post procesado y gotas de tinta

| Antes (bloque 4) | Después |
|---|---|
| ![Nivel 1 antes de la revisión gráfica](Comparacion/bloque4_4_nivel1_inicio.jpg) | ![Nivel 1 después de la revisión gráfica](Comparacion/corregido_4_nivel1_inicio.jpg) |

| Cambio | Por qué |
|---|---|
| Perfil `Niveles_PostProcesado`: **viñeta** (0.3) y **grano** fino (0.18), con un *Volumen Global* en cada nivel | El grano cae igual sobre el dibujo y sobre Murther: los dos parecen impresos en la misma hoja. La viñeta oscurece las orillas como la sombra del menú. **Sin tonemapping ni corrección de color**, que cambiarían los colores del dibujo. |
| Post procesado activado en las cámaras de los niveles | Estaba apagado: la cámara ignoraba cualquier Volume. |
| **Reflejo de tinta húmeda** en las texturas de gota y mancha | Las gotas eran negras sobre las zonas negras del fondo y no se veían. |
| `MurtherTinta` → *Color Tinta*: casi negro → blanco | Multiplicaba la textura y apagaba su borde y el reflejo; ahora el color sale de la textura. |
| Luz del Nivel 1 un poco menos cálida | Con el shader nuevo solo ilumina a Murther, y teñía el casco de crema. |

`Juego → Fase 1` crea todo esto igual (materiales, post procesado y texturas), así que volver a ejecutarlo no lo deshace.

## 8. Iluminación de los fondos 2D

Los fondos eran una imagen sin iluminar: el dibujo se veía igual en todas partes, como pegado detrás de Murther. Ahora la escena está en penumbra y las luces le dan forma: cada fuente de luz dibujada ilumina su zona con su color, el relieve del dibujo reacciona a la luz y lo que queda lejos se apaga.

| Antes | Después |
|---|---|
| **Nivel 1:** Murther lleva una luz cálida; por el hueco se ven las columnas del vacío | |
| ![Nivel 1 antes](Comparacion/iluminacion_antes_n1.jpg) | ![Nivel 1 después](Comparacion/iluminacion_despues_n1.jpg) |
| **Nivel 2:** luz de luna en las ventanas | |
| ![Nivel 2 antes](Comparacion/iluminacion_antes_n2.jpg) | ![Nivel 2 después](Comparacion/iluminacion_despues_n2.jpg) |
| **Nivel 3:** el vitral ilumina; por el hueco se ve el bosque del limbo | |
| ![Nivel 3 antes](Comparacion/iluminacion_antes_n3.jpg) | ![Nivel 3 después](Comparacion/iluminacion_despues_n3.jpg) |

| Farol del nivel 2 (parpadea como llama) | Murther y un Corredor en el nivel 1 |
|---|---|
| ![Farol del nivel 2](Comparacion/iluminacion_farol_nivel2.jpg) | ![Murther y Corredor](Comparacion/iluminacion_murther_y_enemigo.jpg) |

**Mapa de normales generado solo a partir del dibujo** (izquierda, la ilustración; derecha, el mapa):

![Ilustración y su mapa de normales](Comparacion/iluminacion_mapa_normales.jpg)

| Cambio | Por qué |
|---|---|
| Shader **`Five Star/Fondo Iluminado`** (`Assets/Shaders/FondoIluminado.shader`) en los fondos | El material de URP para sprites no recibe luz. Este usa la luz principal y las luces puntuales de la escena (Forward+), con el mapa de normales del dibujo. |
| **Mapas de normales generados del dibujo** (`GeneradorNormalesFondo.cs`, en `Assets/Fondos/Normales`) | La altura sale de la propia ilustración: los contornos de tinta son los bordes de cada piedra, ladrillo o columna, y dentro de ellos la forma sube redondeada; lo que el artista pintó más claro queda más cerca. El punteado de los muros **no** cuenta como contorno (si contara, el muro se vería acolchado). Se guardan a la mitad de resolución (3.9 MB los cinco). **Los PNG originales no se modifican.** |
| **Penumbra**: ambiente bajo y frío, luz principal casi apagada sobre los fondos | Como una escena con antorcha: lejos de las luces el dibujo queda a ~30 % de su brillo; cerca, sube a más del doble y toma el color de la luz. Es lo que hace que deje de verse como una ilustración pegada. Murther no cambia: su shader usa su propia luz. |
| **Luces donde el dibujo tiene fuentes de luz** (*Luces del Escenario* en cada nivel) | Nivel 2: los dos faroles (cálidos, con parpadeo) y la luna en las dos ventanas. Nivel 3: el vitral amarillo y luz fría sobre el bosque. Nivel 1: el dibujo no tiene ninguna fuente de luz, así que **Murther lleva una luz cálida** (*Luz de Murther*) que descubre el relieve a su paso. |
| **Capas traseras** que el equipo dibujó pero no estaban en la escena (*Fondo Lejano*) | *Columnas del vacío* (nivel 1) y *Bosque del limbo* (nivel 3) tienen el mismo lienzo que el fondo y llenan sus huecos transparentes, que antes se veían negros. Van más oscuras y frías, con **parallax** (`CapaParallax`): se mueven más lento que el fondo y dan profundidad. |
| **Halo** de las luces sobre la tinta | Multiplicar la luz por un dibujo negro da negro: sin el halo las luces no se notaban en las zonas de tinta. |
| **Bloom** (umbral 1.1) y viñeta 0.3 → 0.38 en `Niveles_PostProcesado` | Los cristales de los faroles y el vitral, al recibir mucha luz, resplandecen. El umbral queda por encima del blanco de Murther. |
| Los Corredores usan el mismo shader, sin relieve (`GeneradorCorredores` → *Material Sprite*) | Si siguieran sin iluminar, brillarían al 100 % en medio de la penumbra. Solo se aplica en los niveles: en las pantallas de controles siguen igual. |
| Los niveles 2 y 3 reciben una *Luz Direccional* | No tenían ninguna luz. |

**Murther bajo la misma luz.** Con el escenario en penumbra, Murther seguía con su luz blanca y su sombra malva clara: era lo único a pleno brillo y volvía a verse pegado encima.

| Antes | Después |
|---|---|
| ![Murther antes del ajuste de luz](Comparacion/iluminacion_murther_antes.jpg) | ![Murther bajo la luz del escenario](Comparacion/iluminacion_murther_despues.jpg) |

| Cambio | Por qué |
|---|---|
| La luz de personaje toma el **color de la luz que Murther lleva** (`MurtherController` → *Color Luz Tinta*, *Intensidad Luz Tinta*) | El casco y el muro a su alrededor quedan bajo la misma luz cálida de vela, un poco menos saturada en el casco para que siga leyéndose como hueso. Intensidad 0 vuelve a la luz principal. |
| **Sombra del casco más oscura y fría** (0.5, 0.42, 0.48 → 0.34, 0.31, 0.38) | La sombra malva clara brillaba más que las zonas iluminadas del fondo. Ahora cae en el mismo tono que la penumbra; la trama sigue igual. |
| El shader recibe las **luces puntuales** de la escena, en bandas | Al pasar junto a un farol o el vitral, Murther toma su color como el muro. |
| **Contraluz** (*Contraluz* en los materiales del casco y del cuerpo) | La luz que Murther proyecta sobre el muro queda detrás de él y le enciende el borde de la silueta: lo separa del fondo oscuro sin recortarlo con una línea fija. |

**Cómo ajustarlo** (sin tocar código):

| Dónde | Qué |
|---|---|
| Materiales de `Assets/Fondos/Materiales` | *Luz ambiente* (qué tan oscura es la penumbra), *Fuerza de la luz principal*, *Relieve*, *Halo*, *Brillo húmedo*. `FondoLejano_*` son las capas traseras; `Sprite_Iluminado`, los enemigos. |
| *Luces del Escenario* en cada nivel | Mover, cambiar color, intensidad y alcance. *Luz Parpadeo* → *Variación* controla el temblor. |
| `Fondo Lejano` → `CapaParallax` → *Factor* | 0 = se mueve igual que el fondo; más alto = parece más lejos. |
| **Juego → Iluminación de fondos** | *Aplicar a los tres niveles* rehace luces, capas y materiales (pide confirmación). *Regenerar mapas de normales* vuelve a crearlos si cambia un dibujo. *Fase 1* también la aplica al final. |

## 9. Cómo probar

1. Abrir la carpeta raíz del repositorio (la que contiene `Assets`, **no** la carpeta `Assets`) con Unity 6000.3.23f1, en `main` o en `Correcciones-sugeridas`. **No** debe aparecer ningún diálogo ni regenerarse ninguna escena.
2. Abrir **MenuPrincipal** y dar Play:
   - El logo se salta con cualquier tecla.
   - Con flechas o control: navegar el menú, abrir **Créditos** y regresar con Esc/B, abrir **Cerrar Juego** y cancelar.
   - **Iniciar Juego** → cinemática (se salta con el botón, Esc, Start o B) → pantalla de carga → Nivel 1.
3. En el **Nivel 1**: moverse con A/D, flechas o stick; saltar con Espacio, W o A del control; doble salto en el aire. Llegar a las dos orillas: Murther debe verse completo.
4. Caminar por el Nivel 1 de una orilla a otra: la luz de Murther ilumina el muro y las columnas a su paso, y las columnas del fondo se desplazan más lento que el muro. Abrir Nivel2 y Nivel3 y mover la cámara en la vista de escena para ver los faroles y el vitral.
5. Ajustar a gusto *Giro Hacia Camara* y *Luz Tinta* (`MurtherController`), los materiales de `Assets/Murther/Materiales` y `Assets/Fondos/Materiales`, las *Luces del Escenario* y el perfil `Assets/Settings/Niveles_PostProcesado`.

## 10. Pendientes para el equipo (no aplicados)

Son decisiones de contenido o de diseño, así que se dejaron como estaban:

- **Textos de relleno en `TextosJuego.cs`:** frases de los niveles 2 y 3 (`[PDF] …`), integrantes en los créditos y las 6 páginas `[Cómic] Página N`. La cinemática dura 1:30 mostrando ese texto porque `paginas` está vacío.
- **Música y sonido:** existe `Assets/Musica.meta` pero no la carpeta, y el `AudioSource` del menú no tiene clip.
- **Niveles 2 y 3** no tienen a Murther, y **ningún nivel tiene final:** faltan meta, llaves, puerta, Game Over (los videos de `Cinematicas GameOver/` no se usan), vida, daño y combate.
- **Controles anunciados que no existen:** la pantalla de carga muestra *Ataque ligero/Interactuar* y *Recargar*, y `TextosJuego.Controles` menciona *Esquivar (Shift/J)* y *Atacar (K/X)*. El dash y el ataque están en el prototipo anterior (`PlayerController3D`), no en `MurtherController`.
- **No hay pausa ni forma de volver al menú** desde un nivel.
- **Prototipo anterior (`Assets/Assets/`, ~2400 líneas):** ninguna escena lo usa. Conviene decidir si se integra (vida, enemigos, flechas) o se borra.
- **Raya clara en el costado del casco:** con el shader de tinta se ve una línea horizontal iluminada dentro de la sombra. Es un bisel del modelo que atrapa la luz; si no es intencional, se corrige en el modelo (Maya).
- **Iluminación:** la penumbra es una decisión de ambientación; si se quiere más clara, basta subir *Luz ambiente* en los materiales de `Assets/Fondos/Materiales`. En los niveles 2 y 3 la cámara está fija en el centro y los faroles quedan justo fuera de cuadro; cuando esos niveles tengan a Murther y la cámara lo siga, se verán al caminar. Si el equipo agrega lámparas o antorchas al dibujo, conviene ponerles una luz en *Luces del Escenario*.
- **Doble salto:** la cabeza sube 1.15 m y deja un hueco grande respecto al cuerpo. Si no es el efecto buscado, bajar *Altura Cabeza* en `MurtherController`.
- `Juego → Fase 1` sigue creando el Animator Controller y los materiales con GUIDs nuevos cada vez que se ejecuta; solo debe usarse cuando haga falta rehacer a Murther.
- `SampleScene.unity` sobra, y los paquetes `com.unity.ai.assistant` (pre) y `com.unity.pipeline` (experimental) son versiones de prueba.

## 11. Integración en `main`

El PR #1 se fusionó en `main` el 24/11 y después se revirtió. Entre los dos, el commit *Juego cambios* subió dentro de `Assets/` las carpetas `Library`, `ProjectSettings`, `Packages`, `Temp` y `UserSettings` (688 archivos). Unity las crea cuando se abre **la carpeta `Assets`** como si fuera el proyecto: esa copia se llamaba "Assets", de "DefaultCompany", y no tenía nada del juego.

| Cambio | Por qué |
|---|---|
| Se reaplican las correcciones (deshace el revert de #1) | El revert dejó `main` como antes de la revisión, con esas carpetas encima. |
| Se quitan `Assets/Library`, `Assets/ProjectSettings`, `Assets/Packages`, `Assets/Temp` y `Assets/UserSettings` | Unity las importa como parte del juego. `Library/PackageCache` trae otra copia de los `.asmdef` de *Multiplayer Center*, un paquete que el proyecto ya usa, así que aparecen dos ensamblados con el mismo nombre. El proyecto real usa `ProjectSettings/` y `Packages/` de la raíz. |
| `.gitignore`: ignora esas carpetas dentro de `Assets` | Si alguien vuelve a abrir `Assets` como proyecto, ya no se suben. |

**Para el equipo:** al bajar `main`, borrar del disco esas cinco carpetas si siguen dentro de `Assets/`. El `.gitignore` deja de mostrarlas en Git, pero Unity las seguiría importando. En Unity Hub hay que abrir la carpeta raíz del repositorio (la que contiene `Assets`, `Packages` y `ProjectSettings`).
