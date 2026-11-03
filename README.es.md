# Mario Party 3 en Xbox — el port UWP

Cómo llevar **Mario Party 3: Recompiled**
([`n64hub/mario-party-3-recomp`](https://github.com/n64hub/mario-party-3-recomp)) —una recompilación
estática hecha con **N64Recomp + RT64**— a una **Xbox Series X en modo desarrollador**.

*(English: [README.md](README.md))*

La misma receta construye sus dos hermanos, que comparten motor
([`n64hub/mario-party-engine`](https://github.com/n64hub/mario-party-engine)) y fijan los mismos
cuatro commits de bibliotecas: [`marioparty2-xbox`](https://github.com/jajdp/marioparty2-xbox) y
[`marioparty-xbox`](https://github.com/jajdp/marioparty-xbox).

## En qué consola se probó

**Una Xbox Series X**, en modo desarrollador. Las dos consolas en las que se compiló y se probó
son Series X, así que es el único hardware que cubre lo que se afirma aquí.

**En una Series S no se ha probado nunca, ni una vez.** Es otra máquina: menos memoria disponible
para un juego y una GPU más débil. Puede que funcione, puede que vaya más lento o puede que ni
arranque; aquí no se ha medido nada de eso.

## En qué punto está el port

Construido desde la **v0.2.10** del juego. La receta se volvió a comprobar contra ella el
2026-11-02: las seis anclas de las que depende `tools/parche_uwp_build.py` siguen aplicando, sin
tocar nada, tanto en `v0.2.10` como en el `main` de ahora, así que nada de aquí hubo que
reescribirlo por las versiones intermedias. Es el único de los tres que se ha jugado de verdad,
así que es el que más tiene que contar. Medido en una Xbox Series X:

| | |
|---|---|
| ✅ Arranca, dibuja y responde al mando | Paquete «Mario Party 3 Recomp» 0.1.0.5, construido desde la v0.2.10 e instalado por el Device Portal el 2026-11-02 |
| ✅ **60 FPS**, medidos con el contador del propio juego | La lógica corre a 30 Hz y RT64 interpola; el contador mide fotogramas presentados |
| ✅ El launcher sale y la ROM se valida sola | Desde `LocalState`, sin diálogo |
| ✅ La intro se dibuja **completa**, a 1920×1080 y sin superficies negras | El arreglo del culling del ubershader y el de los modos del mipmap funcionan desde el primer arranque — ver [`docs/HOW-IT-WORKS.es.md`](docs/HOW-IT-WORKS.es.md) |
| ✅ **Los tableros y los minijuegos se juegan** | Probado con mando en una Xbox Series X el 2026-11-01 — con el paquete **0.1.0.3**, construido desde la v0.2.7. El 0.1.0.5 de arriba está arrancado y medido, pero aún no jugado con mando |
| ✅ La cinemática de apertura se dibuja a 4:3 exacto | 1438 × 1079 px, medidos en la propia consola, con el paquete 0.1.0.5 |

**Seis fallos encontrados jugando, y ninguno es del port.** Los seis son del proyecto del juego,
se reproducen en su propia compilación de Windows, y están reportados aguas arriba. Cuatro se
arreglaron en el día:

| | Qué era | Cómo está |
|---|---|---|
| [#43](https://github.com/n64hub/mario-party-3-recomp/issues/43) | Todo lo plano se dibujaba pálido y quemado: el juego apaga la corrección de gamma del N64 al arrancar y esa llamada no llegaba al código recompilado | **Arreglado** por su autor el mismo día, en la v0.2.7. Midiendo la misma escena antes y después, la versión vieja es exactamente la nueva elevada a 1/2,2 — el exponente que mejor ajusta es **0,455**, con 0,21 niveles de error medio sobre 256 |
| [#48](https://github.com/n64hub/mario-party-3-recomp/issues/48) | El barrido del cambio de turno no se dibuja, así que se ven las casillas del tablero irse de grupo en grupo | **Arreglado** en la v0.2.9 |
| [#49](https://github.com/n64hub/mario-party-3-recomp/issues/49) | Los minijuegos se estiran en vez de quedarse en 4:3, y un par de ellos se vuelven muy difíciles de jugar | **Arreglado** en la v0.2.8, con `RT64_SCISSOR_INSIDE_FRAMEBUFFER` |
| [#52](https://github.com/n64hub/mario-party-3-recomp/issues/52) | El tablero se redibujaba encogido a 4:3 en el primer paso del barrido del cambio de turno | **Arreglado** en la v0.2.10, comprobado aquí midiendo |
| [#54](https://github.com/n64hub/mario-party-3-recomp/issues/54) | Los **personajes** siguen estrechándose a 0,75 de su ancho durante el barrido, respecto al centro de la pantalla, mientras el tablero y el decorado no | **Abierto.** Medido en tres cambios de turno: le pasa a todos los personajes, no solo al que juega, y aguanta tanto con Vulkan como con la interpolación apagada |
| [#55](https://github.com/n64hub/mario-party-3-recomp/issues/55) | Con la interpolación encendida, los personajes se dibujan en el sitio de sus vecinos durante la cinemática de apertura | **Abierto**, pero ya hay un arreglo en `main` (PR #56, `model_ids.cpp`) que no está en ninguna versión publicada |

**Lo que NO se afirma aquí:** que una partida entera corra limpia de principio a fin.

## Qué es este repositorio

**Una receta, no un producto.** Aquí no hay ROM, ni código recompilado, ni ejecutable, ni
`.msix` — y eso es a propósito, no un descuido.

| Pieza | Qué es |
|---|---|
| [`tools/compilar.ps1`](tools/compilar.ps1) | **La receta entera**, en cinco pasos que se pueden repetir de uno en uno: ROM, recompilar, parchear, bibliotecas estáticas, appx. Con el *porqué* de cada decisión al lado |
| [`tools/parche_uwp_build.py`](tools/parche_uwp_build.py) | Los tres cambios que necesita **el juego mismo**: la opción `UWP_BUILD`, el renombrado de `main()` y el `SDL_INIT_HAPTIC`. Por anclas exactas, idempotente |
| [`tools/resolver_conflictos.py`](tools/resolver_conflictos.py) | Los cuatro conflictos que dan los parches UWP de worleydl sobre este árbol, y por qué los cuatro se resuelven igual: **quedándose con los dos lados** |
| [`tools/sellar.cmake`](tools/sellar.cmake) | Vuelve a sellar los parches de la web de `N64Recomp-browser` para que el siguiente configure no intente ponerlos otra vez y muera |
| [`docs/HOW-IT-WORKS.es.md`](docs/HOW-IT-WORKS.es.md) | Las dos compilaciones, y las diez trampas que costaron tiempo de verdad |
| [`docs/INSTALL.es.md`](docs/INSTALL.es.md) | Qué hace falta, los cinco comandos, y cómo llega a la consola |

A diferencia de casi todos los ports de este catálogo, **aquí sí se publican los cambios al
juego**: el juego es GPL-3.0, así que su código se puede citar. Cuánto se cita, qué y de dónde,
está medido en [`NOTICE.es.md`](NOTICE.es.md).

## Las dos cosas que hay que entender antes de empezar

**1. El generador de Visual Studio NO vale para las bibliotecas del juego.** MSBuild solo tiene
el toolset `v143` para «Application Type = UWP», no `ClangCL`, así que `-T ClangCL` con
`WindowsStore` da `error MSB8020`; y con `cl.exe` se cae el ecosistema entero, que está escrito
para clang (`-Wno-*` en ultramodern y recompinput, `C4146` como error en rabbitizer). Lo correcto
es **Ninja + clang-cl**, con las bibliotecas como objetivo de **escritorio**: lo que las vuelve
UWP es la macro `_UWP` que definen los parches. El único proyecto `WindowsStore` de verdad es el
del appx.

**2. El trabajo de plataforma viene de los forks de worleydl**, no de aquí: `rt64-dev` (rama
`uwp-dk64`), `N64ModernRuntime-dev` (`uwp-rebase`), `RecompFrontend-dev` (`uwp-compat`) y el
paquete [`worleydl/uwp-dep`](https://github.com/worleydl/uwp-dep). ⚠ Ese paquete **tiene una rama
por motor**: para esta familia es `rt64`, no el commit que fija el fork de Star Fox 64, cuyo
`libuwp` no exporta `uwp_PickMultipleFiles` y deja el enlace a medias.

**Y la buena noticia, que es lo que convirtió este port en el trabajo de una tarde:** Mario Party
fija `rt64` en `5473732`, `N64ModernRuntime` en `cdf5abb` y `RecompFrontend` en `b2c5c18` —a uno
o dos commits de aquello contra lo que están hechos los parches de worleydl—. Tres de los cuatro
aplican con `--3way`; el cuarto no da ni un conflicto.

## Las trampas

Todas salieron en la consola. Están desarrolladas en
[`docs/HOW-IT-WORKS.es.md`](docs/HOW-IT-WORKS.es.md); la lista corta:

| | Qué pasa |
|---|---|
| 1 | **`patches/` necesita un clang con backend MIPS.** El clang-cl de Visual Studio no lo trae |
| 2 | ⚠ **Pero el enlazador no puede ser LLD 22**: ya no acepta `-nostdlib`, que es lo que pasa `patches/Makefile`. LLD 19 sí |
| 3 | ⚠ **`vcvars64.bat` rehace el `PATH` y deja fuera el git de Windows.** El de MSYS2 contesta en rutas Unix, `apply.cmake` no encuentra su sello y el configure muere en una fusión a tres bandas |
| 4 | ⚠ **El sello de los parches de la web caduca** en cuanto el parche UWP toca los mismos archivos. Hay que volver a sellar antes de cada configure |
| 5 | **`git apply --3way` pide el estado en el índice**: `add -A` antes… y `reset` después, o se rompe la trampa 4 |
| 6 | **Un símbolo duplicado entre el parche y el juego** (`osPfsIsPlug_recomp`). Manda el del juego |
| 7 | **`AddVectoredExceptionHandler` no se declara para UWP** — `error C3861`. Se declara a mano |
| 8 | ⚠ **`SDL_INIT_HAPTIC` no existe en el SDL de WinRT**, y `SDL_Init` devuelve -1 **para todo el conjunto**, así que la ventana no se crea nunca |
| 9 | ⚠ **El modo portátil no sirve para la carpeta de datos.** `FOLDERID_LocalAppData` devuelve una carpeta que el Device Portal no expone; tiene que ser `LocalFolder` |
| 10 | **Para probar sin nadie delante de la consola** hacen falta dos interruptores, y los dos son **archivos** de `LocalState` |

## Qué no es

Ni **ROM**, ni código recompilado, ni ejecutable, ni appx, ni arte del juego. Tampoco los parches
de worleydl —están en sus repositorios—, ni el paquete UWP, ni el propio proyecto envoltorio
`uwp/`, que deriva del suyo. Lo que hay aquí es la receta, las cuatro herramientas y el registro
de lo que salió mal.

## Créditos y licencia

**Casi nada de lo que hace que esto funcione es nuestro.** Este repositorio es una receta que une el trabajo de otros; sin cualquiera de estas piezas no habría port.

| Qué hace | Quién lo hizo | Licencia |
|---|---|---|
| **[mario-party-3-recomp](https://github.com/n64hub/mario-party-3-recomp)** — recompila *Mario Party 3* | **n64hub** | GPL-3.0 |
| **[mario-party-engine](https://github.com/n64hub/mario-party-engine)** — el motor que comparten las recompilaciones de Mario Party | **n64hub** | GPL-3.0 |
| **[uwp-dep](https://github.com/worleydl/uwp-dep)** — Dawn, SDL3, nod y dxcompiler **compilados para UWP**: la única razón por la que algo de esto corre en una Xbox | **Daniel Worley** (worleydl) | libuwp: zlib, © 2025 |
| **[N64ModernRuntime](https://github.com/N64Recomp/N64ModernRuntime)** — el runtime sobre el que corren esas recompilaciones | the **N64Recomp** project | GPL-3.0 |
| **[RecompFrontend](https://github.com/N64Recomp/RecompFrontend)** — los menús y la configuración dentro del juego | the **N64Recomp** project | GPL-3.0 |
| **[N64Recomp](https://github.com/N64Recomp/N64Recomp)** — recompila estáticamente juegos de N64 a ejecutables nativos | **Mr-Wiseguy** and the N64Recomp project | MIT |
| **[rt64](https://github.com/rt64/rt64)** — el renderizador de N64, con sus mejoras | the **RT64** project | MIT |
| **[SDL](https://github.com/libsdl-org/SDL)** — ventana, mando y audio | **Sam Lantinga** and the SDL contributors | zlib |

Mención especial a **Daniel Worley**: las compilaciones UWP de `uwp-dep` son lo que hace posible este port a Xbox. No las hicimos nosotros, las usamos.

**La receta de empaquetado, las herramientas y esta documentación** son de **Recompilaciones**.


Receta y herramientas de **Recompilaciones**, bajo **[GPL-3.0](LICENSE)** —la licencia del
proyecto del juego que esta receta parchea,
[`n64hub/mario-party-3-recomp`](https://github.com/n64hub/mario-party-3-recomp), y la de
[N64ModernRuntime](https://github.com/N64Recomp/N64ModernRuntime) y
[RecompFrontend](https://github.com/N64Recomp/RecompFrontend), de los que las resoluciones de
conflictos citan algunas líneas—. [N64Recomp](https://github.com/N64Recomp/N64Recomp) y
[RT64](https://github.com/rt64/rt64) son MIT. Los forks UWP sobre los que se apoya la receta son
de **worleydl**.

El proyecto del juego está **escrito por IA** y lo dice él mismo; lo dirige y lo prueba una
persona. Esa es la declaración de sus autores, no una afirmación de aquí.

*Mario Party 3* es © Nintendo / Hudson Soft. Este proyecto no está afiliado con ellos, ni con
Microsoft, ni con los autores de N64Recomp, RT64, el proyecto del juego o los forks UWP.
Titulares de derechos: **jajdpmail@gmail.com**.
