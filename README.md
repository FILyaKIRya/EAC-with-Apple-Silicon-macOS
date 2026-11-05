# EAC на Apple Silicon macOS

### SteamOS в виртуальной машине · FEX · Proton · Easy Anti-Cheat

> Практическая документация по экспериментальному запуску некоторых Windows-игр с Easy Anti-Cheat на Mac с Apple Silicon.
>
> **Важно:** это не официальный способ Valve, Epic Games, Apple или разработчиков игр. Совместимость зависит от конкретной игры, версии Proton, EAC и политики издателя.

[Русский](#русский) · [English](#english) · [日本語](#日本語) · [Español](#español) · [Башҡортса](#башҡортса)

---

## Содержание

- [Русский](#русский)
- [English](#english)
- [日本語](#日本語)
- [Español](#español)
- [Башҡортса](#башҡортса)
- [Источники](#источники--sources--источники--fuentes--сығанаҡтар)
- [Авторство](#авторство--authorship--авторлық)

---

<a name="русский"></a>

# Русский

## О проекте

Этот README описывает экспериментальную архитектуру запуска EAC-игр на Apple Silicon:

```text
macOS
└── FX Steam Launcher
    └── ARM64 SteamOS в виртуальной машине
        └── patched FEX
            └── x86-64 Steam Linux Runtime
                └── x86-64 Proton
                    └── Windows-игра + Linux-режим EAC
```

В отличие от CrossOver, Whisky или обычного Wine, здесь используется настоящий Linux-гость. Это необходимо для Linux/Proton-режима Easy Anti-Cheat. FEX переводит x86-64 Linux-процессы в ARM64, а графика передаётся из SteamOS в Metal через Vulkan или DXMT.

Метод не отключает, не подменяет и не скрывает EAC. Используется оригинальный EAC Runtime от Valve, а виртуальная машина и эмулятор остаются видимыми.

## Требования

- Mac на Apple Silicon: M1, M2, M3 или M4.
- macOS 15 или новее.
- Не менее 16 ГБ оперативной памяти.
- Около 20 ГБ свободного места плюс место для игр.
- Steam-аккаунт и стабильный интернет.

Дополнительные графические режимы:

- **MoltenVK / Vulkan** — macOS 15+.
- **DXMT** — macOS 15.5+, в основном для DirectX 10/11.
- **KosmicKrisp** — macOS 26+.

На Mac с 16 ГБ ОЗУ тяжёлые игры могут работать нестабильно из-за нехватки памяти. Оптимальнее использовать 24 или 32 ГБ.

## Установка

### 1. Установите FX Steam Launcher

Скачайте EAC-совместимую сборку:

- [Релизы steamac](https://github.com/StonkGuy/steamac/releases)
- [Репозиторий steamac](https://github.com/StonkGuy/steamac)

Перетащите `FX Steam Launcher.app` в `/Applications`. При блокировке Gatekeeper:

```bash
xattr -dr com.apple.quarantine "/Applications/FX Steam Launcher.app"
```

Или используйте Control-клик → **Open**, либо **System Settings → Privacy & Security → Open Anyway**.

### 2. Создайте виртуальный диск SteamOS

1. Запустите `FX Steam Launcher`.
2. Выберите **Create New Disk…**.
3. Примите лицензии Valve.
4. Выберите ветку `stable` — она рекомендуется.
5. Выберите размер домашнего раздела.
6. Задайте пароль пользователя `steamos`.
7. Дождитесь скачивания, проверки и установки SteamOS.
8. Разрешите доступ к микрофону.
9. Войдите в Steam внутри окна SteamOS.

Для первого запуска рекомендуется окно 16:10, например `1280×800`.

### 3. Установите игру и Proton

В Steam внутри SteamOS:

1. Установите игру.
2. Откройте **Properties → Compatibility**.
3. Включите принудительный выбор инструмента совместимости.
4. Выберите **Proton Experimental**.
5. Установите **Proton EasyAntiCheat Runtime**.

Не выбирайте ARM64 Proton: x86-64 EAC-клиент не сможет загрузиться. Для VRChat это также предотвращает установку неподходящей Android-версии.

### 4. Установите patched FEX

В `FX Steam Launcher` включите:

```text
Settings → Advanced → Enable SSH
```

Получите каталог `fex-eac` из [репозитория steamac](https://github.com/StonkGuy/steamac/tree/main/fex-eac), перенесите его в SteamOS и выполните внутри гостя:

```bash
cd fex-eac
sh build-fex.sh
sh install-fex-tool.sh
sh install-fex-tool.sh status
```

Команды делают следующее:

- собирают FEX-2610 с необходимыми исправлениями;
- устанавливают его как отдельный Steam compatibility tool;
- сохраняют резервную копию штатной версии FEX;
- позволяют проверить текущую установку.

Для возврата штатного FEX:

```bash
sh install-fex-tool.sh restore
```

После обновления Steam patched FEX может быть заменён штатной версией. В таком случае повторите:

```bash
sh install-fex-tool.sh
```

### 5. Настройте запуск VRChat и EAC

Откройте:

```text
VRChat → Properties → General → Launch Options
```

Используйте:

```text
env EAC_LAUNCHERDIR=$HOME/.local/share/Steam/steamapps/compatdata/438100/pfx/drive_c/users/steamuser/AppData/Roaming/EasyAntiCheat PROTON_EAC_RUNTIME="$HOME/.local/share/Steam/steamapps/common/Proton EasyAntiCheat Runtime" WINEDEBUG=-all PROTON_USE_XALIA=0 WINE_CPU_TOPOLOGY=16:0,1,2,3,0,1,2,3,0,1,2,3,0,1,2,3 %command%
```

`438100` — AppID VRChat.

Основные переменные:

- `EAC_LAUNCHERDIR` — каталог EAC launcher в Wine-префиксе;
- `PROTON_EAC_RUNTIME` — установленный Proton EasyAntiCheat Runtime;
- `PROTON_USE_XALIA=0` — отключает дополнительный слой ввода;
- `WINE_CPU_TOPOLOGY` — сообщает Wine топологию виртуального CPU и устраняет длительную задержку перед входом в игру.

Если конфигурация VM использует другое количество vCPU, список идентификаторов нужно адаптировать к выводу `nproc` внутри SteamOS.

## Выбор графики

Откройте **Settings → Advanced** в `FX Steam Launcher`.

| Режим | Когда использовать |
|---|---|
| **Vulkan Classic** | DirectX 12, сложные игры и случаи, когда DXMT работает нестабильно |
| **DXMT** | DirectX 10/11, если игра корректно работает с этим режимом |

После смены режима перезапустите launcher или виртуальную машину.

Если после DXMT Vulkan-игры перестали запускаться, в Steam выполните **Proton → Properties → Installed Files → Verify integrity of tool files**.

## Проверка успешного запуска

В логе EAC должно появиться:

```text
Launcher finished with: 301, 'Easy Anti-Cheat successfully loaded in-game'
```

В логе игры:

```text
AntiCheat Session Begin: Success
```

Первый запуск может быть медленным из-за компиляции Metal-пайплайнов. После посещения локаций повторные запуски обычно быстрее.

## GameHub и не-Steam игры

GameHub, CrossOver, Whisky и macOS-версия Steam не заменяют SteamOS-гостя: им не хватает Linux kernel, Linux EAC Runtime и x86-64 Linux process tree.

Рабочая схема для GameHub может выглядеть так:

```text
GameHub на macOS
└── FX Steam Launcher
    └── SteamOS
        └── Proton + patched FEX
            └── игра
```

Практически надёжнее запускать EAC-игру непосредственно из Steam внутри SteamOS. Для Epic Games Store в исходной работе упоминается `legendary`, но для каждой игры потребуется отдельная настройка. Универсального профиля GameHub для всех EAC-игр нет.

## Ограничения и безопасность

Не следует считать этот метод гарантированным обходом EAC. Не используйте:

- изменённые DLL игры;
- подмену или отключение EAC;
- spoofing DMI/SMBIOS;
- сокрытие гипервизора или FEX;
- `strace`, `perf` или отладчики на защищённом процессе;
- асинхронные DXVK DLL, если они требуют замены файлов игры.

Не ожидается работа игр, которые:

- требуют Windows kernel-level anti-cheat;
- блокируют виртуальные машины;
- используют собственные дополнительные anti-debug или anti-tamper проверки;
- не включили Linux/Proton-режим EAC.

Совместимость одной игры не означает совместимость другой. Игры и издатели могут изменить политику в любой момент. Использование VM и эмуляции может быть официально неподдерживаемым и несёт ненулевой риск для аккаунта.

---

<a name="english"></a>

# English

## About

This README documents an experimental way to run some EAC-protected Windows games on Apple Silicon Macs:

```text
macOS → FX Steam Launcher → ARM64 SteamOS VM → patched FEX → x86-64 Proton → game + Linux EAC
```

Unlike CrossOver, Whisky, or ordinary Wine, this setup provides a real Linux guest. FEX translates x86-64 Linux processes to ARM64, while Vulkan or DXMT forwards graphics to Apple Metal.

This does **not** disable, replace, hide, or bypass EAC. The stock Proton EasyAntiCheat Runtime is used, and the VM and translator remain visible.

## Requirements

- Apple Silicon Mac: M1, M2, M3, or M4.
- macOS 15 or newer.
- At least 16 GB RAM.
- About 20 GB of free storage, plus space for games.
- A Steam account and a stable internet connection.

Graphics requirements:

- **MoltenVK / Vulkan:** macOS 15+.
- **DXMT:** macOS 15.5+, mainly for DirectX 10/11.
- **KosmicKrisp:** macOS 26+.

## Installation

1. Download an EAC-compatible build from [steamac Releases](https://github.com/StonkGuy/steamac/releases).
2. Copy `FX Steam Launcher.app` to `/Applications`.
3. If Gatekeeper blocks it, use Control-click → **Open**, or run:

   ```bash
   xattr -dr com.apple.quarantine "/Applications/FX Steam Launcher.app"
   ```

4. Open the launcher and select **Create New Disk…**.
5. Accept the Valve licences, choose the `stable` SteamOS branch, select the disk size, and create the `steamos` password.
6. Allow microphone access and sign in to Steam inside the SteamOS window.
7. Keep the display at a 16:10 size such as `1280×800`.

## Steam and Proton

Inside SteamOS:

1. Install the game.
2. Open **Properties → Compatibility**.
3. Force the compatibility tool.
4. Select **Proton Experimental**.
5. Install **Proton EasyAntiCheat Runtime**.

Do not select ARM64 Proton: the x86-64 EAC client cannot load there.

## Patched FEX

Enable **Settings → Advanced → Enable SSH** in the launcher. Copy the [`fex-eac`](https://github.com/StonkGuy/steamac/tree/main/fex-eac) directory into SteamOS and run:

```bash
cd fex-eac
sh build-fex.sh
sh install-fex-tool.sh
sh install-fex-tool.sh status
```

To restore Valve’s original FEX:

```bash
sh install-fex-tool.sh restore
```

Steam updates may replace the patched tool. Reinstall it with `sh install-fex-tool.sh` when necessary.

## VRChat launch options

In **VRChat → Properties → General → Launch Options**, use:

```text
env EAC_LAUNCHERDIR=$HOME/.local/share/Steam/steamapps/compatdata/438100/pfx/drive_c/users/steamuser/AppData/Roaming/EasyAntiCheat PROTON_EAC_RUNTIME="$HOME/.local/share/Steam/steamapps/common/Proton EasyAntiCheat Runtime" WINEDEBUG=-all PROTON_USE_XALIA=0 WINE_CPU_TOPOLOGY=16:0,1,2,3,0,1,2,3,0,1,2,3,0,1,2,3 %command%
```

If the VM has a different number of vCPUs, adapt the CPU list to the output of `nproc` inside SteamOS.

A successful EAC startup normally contains:

```text
Launcher finished with: 301, 'Easy Anti-Cheat successfully loaded in-game'
AntiCheat Session Begin: Success
```

## Graphics

- Use **Vulkan Classic** for DirectX 12, complex titles, or DXMT failures.
- Use **DXMT** for DirectX 10/11 titles when the game works correctly with it.
- Restart the launcher or VM after changing the graphics path.

If Proton files remain modified after switching from DXMT, use **Proton → Properties → Installed Files → Verify integrity of tool files**.

## GameHub and non-Steam games

GameHub, CrossOver, Whisky, and macOS Steam do not provide the Linux kernel, Linux EAC Runtime, or the required x86-64 Linux process tree by themselves.

The practical architecture is:

```text
GameHub on macOS → FX Steam Launcher → SteamOS → Proton + patched FEX → game
```

For EAC titles, launching directly from Steam inside SteamOS is recommended. Non-Steam stores may require a custom launcher or wrapper; there is no universal GameHub profile for all EAC games.

## Limitations and safety

Do not modify game DLLs, replace EAC, hide the VM, spoof hardware identity, or attach `strace`, `perf`, or a debugger to a protected process.

Games requiring Windows kernel anti-cheat, blocking VMs, using additional anti-tamper checks, or lacking Linux/Proton EAC support are not expected to work. Compatibility is title-specific and unofficial. Publishers may change their policies, and account risk cannot be reduced to zero.

---

<a name="日本語"></a>

# 日本語

## 概要

この README は、Apple Silicon Mac で一部の Easy Anti-Cheat 対応 Windows ゲームを動かすための実験的な構成を説明します。

```text
macOS → FX Steam Launcher → ARM64 SteamOS VM → patched FEX → x86-64 Proton → ゲーム + Linux EAC
```

CrossOver、Whisky、通常の Wine とは異なり、Linux のゲスト環境を使用します。FEX が x86-64 Linux プロセスを ARM64 に変換し、Vulkan または DXMT がグラフィックスを Apple Metal に渡します。

この方法は EAC を無効化、置換、隠蔽するものではありません。Valve の Proton EasyAntiCheat Runtime を使用し、VM と FEX は検出可能な状態のままです。

## 必要条件

- Apple Silicon Mac（M1 / M2 / M3 / M4）。
- macOS 15 以降。
- RAM 16 GB 以上。
- 空き容量約 20 GB とゲーム用の追加容量。
- Steam アカウントと安定したインターネット接続。

グラフィックスの条件:

- **MoltenVK / Vulkan:** macOS 15 以降。
- **DXMT:** macOS 15.5 以降。主に DirectX 10/11 用。
- **KosmicKrisp:** macOS 26 以降。

## インストール手順

1. [steamac Releases](https://github.com/StonkGuy/steamac/releases) から EAC 対応ビルドをダウンロードします。
2. `FX Steam Launcher.app` を `/Applications` にコピーします。
3. Gatekeeper にブロックされた場合は Control キーを押しながら開くか、次を実行します。

   ```bash
   xattr -dr com.apple.quarantine "/Applications/FX Steam Launcher.app"
   ```

4. Launcher を開き、**Create New Disk…** を選択します。
5. Valve のライセンスに同意し、`stable` ブランチ、ディスク容量、`steamos` ユーザーのパスワードを設定します。
6. マイクへのアクセスを許可し、SteamOS 内の Steam にログインします。
7. 画面サイズは `1280×800` など 16:10 を推奨します。

## Steam と Proton

SteamOS 内でゲームをインストールし、次の設定を行います。

- **Properties → Compatibility → Proton Experimental**
- **Proton EasyAntiCheat Runtime** をインストール

ARM64 Proton は選択しないでください。x86-64 の EAC クライアントを読み込めません。

## patched FEX の導入

Launcher の **Settings → Advanced → Enable SSH** を有効にします。[`fex-eac`](https://github.com/StonkGuy/steamac/tree/main/fex-eac) ディレクトリを SteamOS にコピーし、次を実行します。

```bash
cd fex-eac
sh build-fex.sh
sh install-fex-tool.sh
sh install-fex-tool.sh status
```

Valve の FEX に戻す場合:

```bash
sh install-fex-tool.sh restore
```

Steam の更新で patched FEX が上書きされる場合があるため、そのときは再度 `sh install-fex-tool.sh` を実行します。

## VRChat の起動オプション

VRChat の **Properties → General → Launch Options** に次を設定します。

```text
env EAC_LAUNCHERDIR=$HOME/.local/share/Steam/steamapps/compatdata/438100/pfx/drive_c/users/steamuser/AppData/Roaming/EasyAntiCheat PROTON_EAC_RUNTIME="$HOME/.local/share/Steam/steamapps/common/Proton EasyAntiCheat Runtime" WINEDEBUG=-all PROTON_USE_XALIA=0 WINE_CPU_TOPOLOGY=16:0,1,2,3,0,1,2,3,0,1,2,3,0,1,2,3 %command%
```

成功時には次のログが表示されます。

```text
Launcher finished with: 301, 'Easy Anti-Cheat successfully loaded in-game'
AntiCheat Session Begin: Success
```

## グラフィックス

- DirectX 12 や DXMT で問題が起きるゲームには **Vulkan Classic**。
- DirectX 10/11 のゲームには **DXMT**。
- 設定変更後は Launcher または VM を再起動します。

## GameHub と非 Steam ゲーム

GameHub、CrossOver、Whisky、macOS 版 Steam だけでは Linux kernel、Linux EAC Runtime、x86-64 Linux process tree が用意されません。

推奨構成は次のとおりです。

```text
GameHub → FX Steam Launcher → SteamOS → Proton + patched FEX → ゲーム
```

EAC ゲームは SteamOS 内の Steam から直接起動する方が確実です。すべての EAC ゲームに使える万能な GameHub プロファイルはありません。

## 制限事項

ゲーム DLL の変更、EAC の置換、VM の隠蔽、ハードウェア情報の偽装、保護プロセスへの `strace`・`perf`・デバッガー接続は禁止してください。

Windows kernel anti-cheat、VM 拒否、追加 anti-tamper、Linux/Proton EAC 非対応のゲームは動作しない可能性があります。これは公式サポート外であり、ゲームごとに結果が異なります。

---

<a name="español"></a>

# Español

## Descripción

Este README documenta un método experimental para ejecutar algunos juegos de Windows protegidos por Easy Anti-Cheat en Macs con Apple Silicon.

```text
macOS → FX Steam Launcher → SteamOS ARM64 en una VM → patched FEX → Proton x86-64 → juego + EAC para Linux
```

A diferencia de CrossOver, Whisky o Wine normal, se utiliza un sistema Linux invitado. FEX traduce los procesos Linux x86-64 a ARM64 y Vulkan o DXMT envían los gráficos a Metal.

El método no desactiva, reemplaza ni oculta EAC. Se utiliza el Proton EasyAntiCheat Runtime original y la máquina virtual permanece visible.

## Requisitos

- Mac con Apple Silicon: M1, M2, M3 o M4.
- macOS 15 o posterior.
- Como mínimo 16 GB de RAM.
- Aproximadamente 20 GB libres, además del espacio de los juegos.
- Cuenta de Steam e Internet estable.

Compatibilidad gráfica:

- **MoltenVK / Vulkan:** macOS 15+.
- **DXMT:** macOS 15.5+, principalmente para DirectX 10/11.
- **KosmicKrisp:** macOS 26+.

## Instalación

1. Descarga una compilación compatible con EAC desde [steamac Releases](https://github.com/StonkGuy/steamac/releases).
2. Copia `FX Steam Launcher.app` a `/Applications`.
3. Si Gatekeeper la bloquea, usa Control-clic → **Open** o ejecuta:

   ```bash
   xattr -dr com.apple.quarantine "/Applications/FX Steam Launcher.app"
   ```

4. Abre el launcher y selecciona **Create New Disk…**.
5. Acepta las licencias de Valve, selecciona la rama `stable`, el tamaño del disco y la contraseña de `steamos`.
6. Permite el acceso al micrófono e inicia sesión en Steam dentro de SteamOS.
7. Se recomienda una ventana 16:10, por ejemplo `1280×800`.

## Steam y Proton

Dentro de SteamOS:

1. Instala el juego.
2. Abre **Properties → Compatibility**.
3. Obliga el uso de una herramienta de compatibilidad.
4. Selecciona **Proton Experimental**.
5. Instala **Proton EasyAntiCheat Runtime**.

No selecciones Proton ARM64: el cliente EAC x86-64 no puede cargarse allí.

## Instalación de patched FEX

Activa **Settings → Advanced → Enable SSH** en el launcher. Copia el directorio [`fex-eac`](https://github.com/StonkGuy/steamac/tree/main/fex-eac) a SteamOS y ejecuta:

```bash
cd fex-eac
sh build-fex.sh
sh install-fex-tool.sh
sh install-fex-tool.sh status
```

Para restaurar el FEX original de Valve:

```bash
sh install-fex-tool.sh restore
```

Una actualización de Steam puede reemplazar patched FEX. Si ocurre, ejecuta de nuevo `sh install-fex-tool.sh`.

## Opciones de lanzamiento para VRChat

En **VRChat → Properties → General → Launch Options**, utiliza:

```text
env EAC_LAUNCHERDIR=$HOME/.local/share/Steam/steamapps/compatdata/438100/pfx/drive_c/users/steamuser/AppData/Roaming/EasyAntiCheat PROTON_EAC_RUNTIME="$HOME/.local/share/Steam/steamapps/common/Proton EasyAntiCheat Runtime" WINEDEBUG=-all PROTON_USE_XALIA=0 WINE_CPU_TOPOLOGY=16:0,1,2,3,0,1,2,3,0,1,2,3,0,1,2,3 %command%
```

Un inicio correcto suele mostrar:

```text
Launcher finished with: 301, 'Easy Anti-Cheat successfully loaded in-game'
AntiCheat Session Begin: Success
```

## Gráficos

- Usa **Vulkan Classic** para DirectX 12, juegos complejos o problemas con DXMT.
- Usa **DXMT** para DirectX 10/11 cuando el juego sea compatible.
- Reinicia el launcher o la VM después de cambiar el modo gráfico.

## GameHub y juegos que no son de Steam

GameHub, CrossOver, Whisky y Steam para macOS no proporcionan por sí solos el kernel Linux, el Runtime de EAC para Linux ni el árbol de procesos Linux x86-64 necesario.

La arquitectura práctica es:

```text
GameHub en macOS → FX Steam Launcher → SteamOS → Proton + patched FEX → juego
```

Para juegos con EAC se recomienda iniciarlos directamente desde Steam dentro de SteamOS. No existe un perfil universal de GameHub para todos los juegos con EAC.

## Limitaciones y seguridad

No modifiques DLL del juego, no reemplaces EAC, no ocultes la VM, no falsifiques la identidad del hardware y no uses `strace`, `perf` ni depuradores sobre procesos protegidos.

Los juegos con anti-cheat a nivel de kernel de Windows, bloqueo de máquinas virtuales, anti-tamper adicional o sin soporte EAC para Linux/Proton probablemente no funcionarán. El método no tiene soporte oficial y el resultado depende de cada juego y editor.

---

<a name="башҡортса"></a>

# Башҡортса

## Проект тураһында

Был README Apple Silicon процессорлы Mac компьютерҙарында Easy Anti-Cheat менән һаҡланған ҡайһы бер Windows уйындарын эшләтеүҙең эксперименталь ысулын аңлата.

```text
macOS → FX Steam Launcher → виртуаль машинала ARM64 SteamOS → patched FEX → x86-64 Proton → уйын + Linux EAC
```

CrossOver, Whisky йәки ғәҙәти Wine менән сағыштырғанда, бында тулы Linux ҡунаҡ системаһы ҡулланыла. FEX x86-64 Linux процестарын ARM64-кә тәржемә итә, ә Vulkan йәки DXMT графиканы Apple Metal-ға тапшыра.

Был ысул EAC-ты һүндермәй, алмаштырмай һәм йәшермәй. Valve-тың Proton EasyAntiCheat Runtime компоненттары ҡулланыла, виртуаль машина һәм FEX күренеп тора.

## Талаптар

- Apple Silicon Mac: M1, M2, M3 йәки M4.
- macOS 15 йәки яңы версия.
- Кәм тигәндә 16 ГБ оператив хәтер.
- Яҡынса 20 ГБ буш урын һәм уйындар өсөн өҫтәмә урын.
- Steam аккаунты һәм тотороҡло интернет.

Графика режимдары:

- **MoltenVK / Vulkan:** macOS 15 һәм яңыраҡ.
- **DXMT:** macOS 15.5 һәм яңыраҡ, башлыса DirectX 10/11 өсөн.
- **KosmicKrisp:** macOS 26 һәм яңыраҡ.

## Урындағырыу

1. [steamac Releases](https://github.com/StonkGuy/steamac/releases) битенән EAC-ты хуплаған йыйылманы йөкләгеҙ.
2. `FX Steam Launcher.app` файлын `/Applications` папкаһына күсерегеҙ.
3. Gatekeeper эшләтеүгә ҡаршы булһа, Control-клик → **Open** һайлағыҙ йәки терминалда башҡарығыҙ:

   ```bash
   xattr -dr com.apple.quarantine "/Applications/FX Steam Launcher.app"
   ```

4. Launcher-ҙы асып, **Create New Disk…** һайлағыҙ.
5. Valve лицензияларын ҡабул итегеҙ, `stable` тармағын, диск күләмен һәм `steamos` ҡулланыусыһы өсөн пароль билдәләгеҙ.
6. Микрофонға рөхсәт бирегеҙ һәм SteamOS эсендә Steam-ға инегеҙ.
7. 16:10 нисбәтле тәҙрә ҡулланыу яҡшы, мәҫәлән `1280×800`.

## Steam һәм Proton

SteamOS эсендә:

1. Уйынды урынлаштырығыҙ.
2. **Properties → Compatibility** бүлеген асығыҙ.
3. Compatibility tool-ды көсләп һайлауҙы ҡабыҙығыҙ.
4. **Proton Experimental** һайлағыҙ.
5. **Proton EasyAntiCheat Runtime** урынлаштырығыҙ.

ARM64 Proton һайламағыҙ: x86-64 EAC клиенты унда йөкләнә алмай.

## Patched FEX урынлаштырыу

Launcher эсендә **Settings → Advanced → Enable SSH** ҡабыҙығыҙ. [`fex-eac`](https://github.com/StonkGuy/steamac/tree/main/fex-eac) папкаһын SteamOS-ҡа күсереп, ҡунаҡ системала башҡарығыҙ:

```bash
cd fex-eac
sh build-fex.sh
sh install-fex-tool.sh
sh install-fex-tool.sh status
```

Valve-тың ғәҙәти FEX версияһына ҡайтыу өсөн:

```bash
sh install-fex-tool.sh restore
```

Steam яңыртылғанда patched FEX өҫтөнән яҙылырға мөмкин. Был осраҡта `sh install-fex-tool.sh` командаһын ҡабат башҡарығыҙ.

## VRChat өсөн эшләтеү параметрҙары

Steam-да **VRChat → Properties → General → Launch Options** бүлеген асып, ошо юлды ҡуйығыҙ:

```text
env EAC_LAUNCHERDIR=$HOME/.local/share/Steam/steamapps/compatdata/438100/pfx/drive_c/users/steamuser/AppData/Roaming/EasyAntiCheat PROTON_EAC_RUNTIME="$HOME/.local/share/Steam/steamapps/common/Proton EasyAntiCheat Runtime" WINEDEBUG=-all PROTON_USE_XALIA=0 WINE_CPU_TOPOLOGY=16:0,1,2,3,0,1,2,3,0,1,2,3,0,1,2,3 %command%
```

EAC уңышлы эшләһә, логта ошондай юлдар күренә:

```text
Launcher finished with: 301, 'Easy Anti-Cheat successfully loaded in-game'
AntiCheat Session Begin: Success
```

## Графика

- DirectX 12 һәм DXMT менән проблема булған уйындар өсөн **Vulkan Classic** ҡулланығыҙ.
- DirectX 10/11 уйындары өсөн, әгәр ул дөрөҫ эшләһә, **DXMT** һайлағыҙ.
- Режимды алмаштырғандан һуң launcher-ҙы йәки виртуаль машинаны яңынан эшләтегеҙ.

## GameHub һәм Steam-дан тыш уйындар

GameHub, CrossOver, Whisky һәм macOS өсөн Steam үҙҙәре генә Linux ядроһын, Linux EAC Runtime-ын һәм кәрәкле x86-64 Linux процесс ағасын бирмәй.

Ҡулланыу схемаһы:

```text
macOS-та GameHub → FX Steam Launcher → SteamOS → Proton + patched FEX → уйын
```

EAC менән уйындарҙы SteamOS эсендәге Steam аша туранан-тура эшләтеү ышаныслыраҡ. Барлыҡ EAC уйындары өсөн универсаль GameHub профиле юҡ.

## Сикләүҙәр һәм хәүефһеҙлек

Уйын DLL-файлдарын үҙгәртмәгеҙ, EAC-ты алмаштырмағыҙ, виртуаль машинаны йәшермәгеҙ, ҡоролма мәғлүмәттәрен ялғанламағыҙ. Һаҡланған процесҡа `strace`, `perf` йәки отладчик ҡушмағыҙ.

Windows kernel anti-cheat талап иткән, виртуаль машиналарҙы блоклаған, өҫтәмә anti-tamper тикшереүҙәрен ҡулланған йәки Linux/Proton EAC хупламаған уйындар эшләмәҫкә мөмкин. Был рәсми хупланған ысул түгел, ә һөҙөмтә һәр уйынға айырым бәйле.

---

<a name="источники--sources--источники--fuentes--сығанаҡтар"></a>

# Источники / Sources / Источники / Fuentes / Сығанаҡтар

Основные материалы, использованные при подготовке этой документации:

1. **Исходный PDF:** `EAC-APPLE-SILICON.pdf`, переданный автором этого репозитория.
2. **Публикация, откуда был получен PDF:** [Reddit — A Mac running games with Easy Anti-Cheat](https://www.reddit.com/r/macgaming/comments/1x49ypl/a_mac_running_games_with_easy_anticheat/).
3. [StonkGuy/steamac — основной репозиторий](https://github.com/StonkGuy/steamac).
4. [steamac — SETUP.md](https://github.com/StonkGuy/steamac/blob/main/SETUP.md).
5. [steamac — EAC/FEX scripts](https://github.com/StonkGuy/steamac/tree/main/fex-eac).
6. [steamac — releases](https://github.com/StonkGuy/steamac/releases).
7. [StonkGuy/eac-arm64-emulation-poc](https://github.com/StonkGuy/eac-arm64-emulation-poc).
8. [FEX-Emu](https://github.com/FEX-Emu/FEX).
9. [libkrun](https://github.com/containers/libkrun).
10. [Mesa Venus](https://docs.mesa3d.org/drivers/venus.html).
11. [Valve gamescope](https://github.com/ValveSoftware/gamescope).
12. [Legendary — Epic Games Store CLI](https://github.com/derrod/legendary).

Ссылки на проекты приведены как справочные источники. Этот README не является официальной документацией Valve, Epic Games, Apple, VRChat или разработчиков указанных проектов.

---

<a name="авторство--authorship--авторлық"></a>

# Авторство / Authorship / Авторлыҡ

Текст этого README подготовлен **Claude Opus 5.5 от Anthropic** по запросу владельца репозитория на основе переданного PDF, указанной публикации Reddit и публичной документации проектов.

- Автор технического README: **Claude Opus 5.5, Anthropic**.
- Переводы: подготовлены Claude Opus 5.5.
- Автор репозитория и пользователь, предоставивший исходные материалы: **владелец этого GitHub-репозитория**.

Материал носит информационный характер. Перед использованием проверьте лицензии, правила конкретной игры и требования её издателя.

---

## Disclaimer

This project is experimental and unofficial. It does not disable or bypass Easy Anti-Cheat. It does not guarantee that any game will launch or that an account will remain unaffected. Use it only with games and accounts for which you accept the risks and the applicable terms of service.
