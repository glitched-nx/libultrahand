# ☆ libultrahand

[![platform](https://img.shields.io/badge/platform-Switch-898c8c?logo=C++.svg)](https://gbatemp.net/forums/nintendo-switch.283/?prefix_id=44)
[![language](https://img.shields.io/badge/language-C++-ba1632?logo=C++.svg)](https://github.com/topics/cpp)
[![GPLv2 License](https://img.shields.io/badge/license-GPLv2-189c11.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![Latest Version](https://img.shields.io/github/v/release/ppkantorski/libultrahand?label=latest%20version&color=blue)](https://github.com/ppkantorski/libultrahand/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/ppkantorski/libultrahand/total?color=6f42c1)](https://github.com/ppkantorski/libultrahand/graphs/traffic)
[![GitHub issues](https://img.shields.io/github/issues/ppkantorski/libultrahand?color=222222)](https://github.com/ppkantorski/libultrahand/issues)
[![GitHub stars](https://img.shields.io/github/stars/ppkantorski/libultrahand)](https://github.com/ppkantorski/libultrahand/stargazers)


Expanded [**libtesla**](https://github.com/WerWolv/libtesla) (originally by [WerWolv](https://github.com/WerWolv)) + **libultra** libraries for overlay development on the Nintendo Switch

![libultrahand Logo](.pics/libultrahand.png)

## Usage

### Overriding Themes and Wallpapers

To customize theme, wallpaper and / or allow direct language translations for your overlay, you can override the default settings by adding the following lines to your `Makefile`:

```
# Enable appearance overriding
UI_OVERRIDE_PATH := /config/<OVERLAY_NAME>/
CFLAGS += -DUI_OVERRIDE_PATH="\"$(UI_OVERRIDE_PATH)\""
```

Replace `<OVERLAY_NAME>` with the desired name of your overlay config directory.

Users can specify custom Ultrahand `theme.ini` and `wallpaper.rgba` files for the overlay to use located in your SD card's `/config/<OVERLAY_NAME>/` or `{UI_OVERRIDE_PATH}` directory.

#### **Troubleshooting**

**Notice:** Makefile directives also must be added to `CXXFLAGS`.
You can do this by adding the directives before `CXXFLAGS := $(CFLAGS)` gets defined, or include `CXXFLAGS += ...`.


There are rare occurences where the theme and wallpaper are still not being loaded.  This may have to do with how the GUI class is used in some projects. For a work around, you can try adding the `INITIALIZE_IN_GUI_DIRECTIVE` directive. 

```
# For theme / wallpaper loading in GUI class method (add to project if theme does not appear)
INITIALIZE_IN_GUI_DIRECTIVE := 1
CFLAGS += -DINITIALIZE_IN_GUI_DIRECTIVE=$(INITIALIZE_IN_GUI_DIRECTIVE)
```

This fix will work for many projects, but other projects may not like this directive or may not need it at all so use with that in mind.

---

### Overriding Languages

For language translation, `UI_OVERRIDE_PATH` must be defined.  Translations are performed direction on the rederer's `drawString` method. Direct strings can be added to a json located in `/config/<OVERLAY_NAME>/lang/` or `{UI_OVERRIDE_PATH}/lang/`.

Jsons will need to be named ISO 639-1 format (en, de, fr, es, etc...) and will only be used in accordance with the current language set in the Ultrahand Overlay `/config/ultrahand/config.ini`.

The format for language jsons is as follows.

```json
{
  "English String": "Translated String",
  "Another String": "Another Translation",
  ...
}
```

---

### Ultrahand Overlay Widget

To add the Ultrahand Overlay widget to your `OverlayFrame`'s, add the following directive to your `Makefile`:

```
# Enable Widget
USING_WIDGET_DIRECTIVE := 1 
CFLAGS += -DUSING_WIDGET_DIRECTIVE=$(USING_WIDGET_DIRECTIVE)
```

---

### Forcing use of `<stdio.h>` instead of `<fstream>`

To compile without utilizing `fstream`, add the following directive to your `Makefile`:

```
# Disable fstream
NO_FSTREAM_DIRECTIVE := 1 
CFLAGS += -DNO_FSTREAM_DIRECTIVE=$(NO_FSTREAM_DIRECTIVE)
```

---

### Initializing Settings

Ultrahand Overlay theme variables and settings for your overlay are read automatically upon initialization.  Themes loading implementation is currently set within `OverlayFrame` and `HeaderOverlayFrame`.  

However if you are breaking your project up into individual parts that only import `tesla.hpp` and modify elements, you may need to declare `/libultrahand/libultra/include` at the start of your `INCLUDES` in your make file.

If that still is not working, then you may need to add this line somewhere for the theme to be applied to that element.

```cpp
tsl::initializeThemeVars(); // Initialize variables for ultrahand themes
```

### Download Methods

To utilize the `libultra` download methods in your project, you will need to add the following line to your `initServices` function:

```cpp
initializeCurl();
```

As well as the following line to your `exetServices` function:

```cpp
cleanupCurl();
```

These lines will ensure `curl` functions properly within the overlay.

## Compiling

### Necessary Libraries

Developers should include the following libararies in their `Makefile` if they want full `libultra` functionality in their projects.

```
LIBS := -lcurl -lz -lzzip -lmbedtls -lmbedx509 -lmbedcrypto -ljansson -lnx
```

### Active Services

Service conflictions can occur, so if you are already using the following libnx services, you may want to remove them from your project.

```cpp
i2cInitialize();
fsdevMountSdmc();
splInitialize();
spsmInitialize();
ASSERT_FATAL(socketInitializeDefault());
ASSERT_FATAL(nifmInitialize(NifmServiceType_User));
ASSERT_FATAL(smInitialize()); // needed to prevent issues with powering device into sleep
```

Service `i2cInitialize` however is only utilized in accordance with `USING_WIDGET_DIRECTIVE`.

### Optional Compilation Flags

```
-ffunction-sections -fdata-sections
```

These options are present in both CFLAGS and CXXFLAGS. They instruct the compiler to place each function and data item in its own section, which allows the linker to more easily identify and remove unused code.

```
-Wl,--gc-sections
```

Included in LDFLAGS. This linker flag instructs the linker to remove unused sections that were created by -ffunction-sections and -fdata-sections. This ensures that functions or data that are not used are removed from the final executable.

```
-flto (Link Time Optimization)
```

Present in CFLAGS, CXXFLAGS, and LDFLAGS. It enables link-time optimization, allowing the compiler to optimize across different translation units and remove any unused code during the linking phase. You also use -flto=6 to control the number of threads for parallel LTO, which helps speed up the process.

```
-fuse-linker-plugin
```

This flag allows the compiler and linker to better collaborate when using LTO, which further helps in optimizing and eliminating unused code.
Together, these flags (-ffunction-sections, -fdata-sections, -Wl,--gc-sections, and -flto) ensure that any unused functions or data are stripped out during the build process, leading to a smaller and more optimized final binary.


## Build Examples

- [Ultrahand Overlay](https://github.com/ppkantorski/Ultrahand-Overlay)

- [Tetris Overlay](https://github.com/ppkantorski/Tetris-Overlay)

- [Status Monitor Overlay](https://github.com/ppkantorski/Status-Monitor-Overlay)

- [Edizon Overlay](https://github.com/ppkantorski/EdiZon-Overlay)

- [Sysmodules](https://github.com/ppkantorski/ovl-sysmodules)

- [sys-clk](https://github.com/ppkantorski/sys-clk)

- [FPSLocker](https://github.com/ppkantorski/FPSLocker)

- [ReverseNX-RT](https://github.com/ppkantorski/ReverseNX-RT)

- [QuickNTP](https://github.com/ppkantorski/QuickNTP)

- [SysDVR Overlay](https://github.com/ppkantorski/sysdvr-overlay)

- [Fizeau](https://github.com/ppkantorski/Fizeau)

- [NX-FanControl](https://github.com/ppkantorski/NX-FanControl)

- [DNS-MITM_Manager](https://github.com/ppkantorski/DNS-MITM_Manager)

## Contributing

Contributions are welcome! If you have any ideas, suggestions, or bug reports, please raise an [issue](https://github.com/ppkantorski/libultrahand/issues/new/choose), submit a [pull request](https://github.com/ppkantorski/libultrahand/compare) or reach out to me directly on [GBATemp](https://gbatemp.net/threads/ultrahand-overlay-the-fully-craft-able-overlay-executor.633560/).

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/X8X3VR194)

## License

This project is licensed and distributed under [GPLv2](LICENSE) with a [custom library](libultra) utilizing [CC-BY-4.0](SUB_LICENSE).

Copyright (c) 2024 ppkantorski

---
# ☆ libultrahand

[![Plattform](https://img.shields.io/badge/platform-Switch-898c8c?logo=C++.svg)](https://gbatemp.net/forums/nintendo-switch.283/?prefix_id=44)
[![Sprache](https://img.shields.io/badge/language-C++-ba1632?logo=C++.svg)](https://github.com/topics/cpp)
[![GPLv2 Lizenz](https://img.shields.io/badge/license-GPLv2-189c11.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![Neueste Version](https://img.shields.io/github/v/release/ppkantorski/libultrahand?label=latest%20version&color=blue)](https://github.com/ppkantorski/libultrahand/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/ppkantorski/libultrahand/total?color=6f42c1)](https://github.com/ppkantorski/libultrahand/graphs/traffic)
[![GitHub Probleme](https://img.shields.io/github/issues/ppkantorski/libultrahand?color=222222)](https://github.com/ppkantorski/libultrahand/issues)
[![GitHub Sterne](https://img.shields.io/github/stars/ppkantorski/libultrahand)](https://github.com/ppkantorski/libultrahand/stargazers)

Erweiterte [**libtesla**](https://github.com/WerWolv/libtesla) (ursprünglich von [WerWolv](https://github.com/WerWolv)) + **libultra** Bibliotheken für die Entwicklung von Overlays auf der Nintendo Switch.

![libultrahand Logo](.pics/libultrahand.png)

## Verwendung

### Anpassen von Themes und Hintergrundbildern

Um das Theme, das Hintergrundbild und/oder direkte Sprachübersetzungen für Dein Overlay anzupassen, kannst Du die Standardeinstellungen überschreiben, indem Du die folgenden Zeilen zu Deinem `Makefile` hinzufügst:

```makefile
# Erscheinungsbild-Überschreibung aktivieren
UI_OVERRIDE_PATH := /config/<OVERLAY_NAME>/
CFLAGS += -DUI_OVERRIDE_PATH="\"$(UI_OVERRIDE_PATH)\""
```

Ersetze `<OVERLAY_NAME>` durch den gewünschten Namen Deines Overlay-Konfigurationsverzeichnisses.

Benutzer können benutzerdefinierte Ultrahand `theme.ini` und `wallpaper.rgba` Dateien für das Overlay verwenden, die sich auf der SD-Karte im Verzeichnis `/config/<OVERLAY_NAME>/` oder `{UI_OVERRIDE_PATH}` befinden.

#### **Fehlerbehebung**

**Hinweis:** Makefile-Direktiven müssen auch zu `CXXFLAGS` hinzugefügt werden. Du kannst dies tun, indem Du die Direktiven hinzufügst, bevor `CXXFLAGS := $(CFLAGS)` definiert wird, oder indem Du `CXXFLAGS += ...` einfügst.

Es gibt seltene Fälle, in denen das Theme und das Hintergrundbild immer noch nicht geladen werden. Dies kann damit zusammenhängen, wie die GUI-Klasse in einigen Projekten verwendet wird. Als Workaround kannst Du versuchen, die Direktive `INITIALIZE_IN_GUI_DIRECTIVE` hinzuzufügen.

```makefile
# Für das Laden von Theme/Hintergrundbild in der GUI-Klassenmethode (zum Projekt hinzufügen, wenn das Theme nicht erscheint)
INITIALIZE_IN_GUI_DIRECTIVE := 1
CFLAGS += -DINITIALIZE_IN_GUI_DIRECTIVE=$(INITIALIZE_IN_GUI_DIRECTIVE)
```

Dieser Fix funktioniert für viele Projekte, aber andere Projekte mögen diese Direktive möglicherweise nicht oder benötigen sie überhaupt nicht, also verwende sie mit Bedacht.

---

### Überschreiben von Sprachen

Für die Sprachübersetzung muss `UI_OVERRIDE_PATH` definiert sein. Übersetzungen werden direkt in der `drawString`-Methode des Renderers durchgeführt. Direkte Zeichenfolgen können in einer JSON-Datei hinzugefügt werden, die sich im Verzeichnis `/config/<OVERLAY_NAME>/lang/` oder `{UI_OVERRIDE_PATH}/lang/` befindet.

JSON-Dateien müssen im ISO 639-1-Format benannt werden (en, de, fr, es, etc...) und werden nur in Übereinstimmung mit der aktuell in der Ultrahand Overlay `/config/ultrahand/config.ini` eingestellten Sprache verwendet.

Das Format für Sprach-JSONs ist wie folgt.

```json
{
  "Englischer Text": "Übersetzter Text",
  "Ein weiterer Text": "Eine weitere Übersetzung",
  ...
}
```

---

### Ultrahand Overlay Widget

Um das Ultrahand Overlay-Widget zu Deinen `OverlayFrame`'s hinzuzufügen, füge die folgende Direktive zu Deinem `Makefile` hinzu:

```makefile
# Widget aktivieren
USING_WIDGET_DIRECTIVE := 1 
CFLAGS += -DUSING_WIDGET_DIRECTIVE=$(USING_WIDGET_DIRECTIVE)
```

---

### Erzwingen der Verwendung von `<stdio.h>` anstelle von `<fstream>`

Um ohne die Nutzung von `fstream` zu kompilieren, füge die folgende Direktive zu Deinem `Makefile` hinzu:

```makefile
# fstream deaktivieren
NO_FSTREAM_DIRECTIVE := 1 
CFLAGS += -DNO_FSTREAM_DIRECTIVE=$(NO_FSTREAM_DIRECTIVE)
```

---

### Initialisieren von Einstellungen

Ultrahand Overlay-Theme-Variablen und Einstellungen für Dein Overlay werden automatisch beim Initialisieren gelesen. Die Implementierung des Theme-Ladens ist derzeit innerhalb von `OverlayFrame` und `HeaderOverlayFrame` festgelegt.

Wenn Du Dein Projekt jedoch in einzelne Teile aufteilst, die nur `tesla.hpp` importieren und Elemente ändern, musst Du möglicherweise `/libultrahand/libultra/include` am Anfang Deiner `INCLUDES` in Deinem Makefile deklarieren.

Wenn das immer noch nicht funktioniert, musst Du möglicherweise diese Zeile irgendwo hinzufügen, damit das Theme auf dieses Element angewendet wird.

```cpp
tsl::initializeThemeVars(); // Variablen für Ultrahand-Themes initialisieren
```

### Download-Methoden

Um die `libultra`-Download-Methoden in Deinem Projekt zu nutzen, musst Du die folgende Zeile zu Deiner `initServices`-Funktion hinzufügen:

```cpp
initializeCurl();
```

Sowie die folgende Zeile zu Deiner `exetServices`-Funktion:

```cpp
cleanupCurl();
```

Diese Zeilen stellen sicher, dass `curl` innerhalb des Overlays ordnungsgemäß funktioniert.

## Kompilieren

### Notwendige Bibliotheken

Entwickler sollten die folgenden Bibliotheken in ihrem `Makefile` einbinden, wenn sie die volle `libultra`-Funktionalität in ihren Projekten wünschen.

```makefile
LIBS := -lcurl -lz -lzzip -lmbedtls -lmbedx509 -lmbedcrypto -ljansson -lnx
```

### Aktive Dienste

Dienstkonflikte können auftreten, daher solltest Du die folgenden libnx-Dienste aus Deinem Projekt entfernen, wenn Du sie bereits verwendest.

```cpp
i2cInitialize();
fsdevMountSdmc();
splInitialize();
spsmInitialize();
ASSERT_FATAL(socketInitializeDefault());
ASSERT_FATAL(nifmInitialize(NifmServiceType_User));
ASSERT_FATAL(smInitialize()); // notwendig, um Probleme beim Einschalten des Geräts in den Schlafmodus zu vermeiden
```

Der Dienst `i2cInitialize` wird jedoch nur in Übereinstimmung mit `USING_WIDGET_DIRECTIVE` verwendet.

### Optionale Kompilierungsflags

```makefile
-ffunction-sections -fdata-sections
```

Diese Optionen sind sowohl in CFLAGS als auch in CXXFLAGS vorhanden. Sie weisen den Compiler an, jede Funktion und jedes Datenelement in einen eigenen Abschnitt zu platzieren, was es dem Linker erleichtert, ungenutzten Code zu identifizieren und zu entfernen.

```makefile
-Wl,--gc-sections
```

In LDFLAGS enthalten. Dieses Linker-Flag weist den Linker an, ungenutzte Abschnitte zu entfernen, die durch -ffunction-sections und -fdata-sections erstellt wurden. Dies stellt sicher, dass Funktionen oder Daten, die nicht verwendet werden, aus der endgültigen ausführbaren Datei entfernt werden.

```makefile
-flto (Link Time Optimization)
```

In CFLAGS, CXXFLAGS und LDFLAGS vorhanden. Es ermöglicht die Optimierung zur Linkzeit, sodass der Compiler über verschiedene Übersetzungseinheiten hinweg optimieren und ungenutzten Code während der Verknüpfungsphase entfernen kann. Du kannst auch -flto=6 verwenden, um die Anzahl der Threads für paralleles LTO zu steuern, was den Prozess beschleunigt.

```makefile
-fuse-linker-plugin
```

Dieses Flag ermöglicht es dem Compiler und dem Linker, bei der Verwendung von LTO besser zusammenzuarbeiten, was weiter zur Optimierung und Eliminierung ungenutzten Codes beiträgt. Zusammen stellen diese Flags (-ffunction-sections, -fdata-sections, -Wl,--gc-sections und -flto) sicher, dass ungenutzte Funktionen oder Daten während des Build-Prozesses entfernt werden, was zu einer kleineren und optimierten endgültigen Binärdatei führt.

## Build-Beispiele

- [Ultrahand Overlay](https://github.com/ppkantorski/Ultrahand-Overlay)

- [Tetris Overlay](https://github.com/ppkantorski/Tetris-Overlay)

- [Status Monitor Overlay](https://github.com/ppkantorski/Status-Monitor-Overlay)

- [Edizon Overlay](https://github.com/ppkantorski/EdiZon-Overlay)

- [Sysmodules](https://github.com/ppkantorski/ovl-sysmodules)

- [sys-clk](https://github.com/ppkantorski/sys-clk)

- [FPSLocker](https://github.com/ppkantorski/FPSLocker)

- [ReverseNX-RT](https://github.com/ppkantorski/ReverseNX-RT)

- [QuickNTP](https://github.com/ppkantorski/QuickNTP)

- [SysDVR Overlay](https://github.com/ppkantorski/sysdvr-overlay)

- [Fizeau](https://github.com/ppkantorski/Fizeau)

- [NX-FanControl](https://github.com/ppkantorski/NX-FanControl)

## Mitwirken

Beiträge sind willkommen! Wenn Du Ideen, Vorschläge oder Fehlerberichte hast, eröffne bitte ein [Issue](https://github.com/ppkantorski/libultrahand/issues/new/choose), reiche einen [Pull Request](https://github.com/ppkantorski/libultrahand/compare) ein oder kontaktiere mich direkt auf [GBATemp](https://gbatemp.net/threads/ultrahand-overlay-the-fully-craft-able-overlay-executor.633560/).

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/X8X3VR194)

## Lizenz

Dieses Projekt ist lizenziert und wird unter [GPLv2](LICENSE) mit einer [benutzerdefinierten Bibliothek](libultra) verteilt, die [CC-BY-4.0](SUB_LICENSE) verwendet.

Copyright (c) 2024 ppkantorski