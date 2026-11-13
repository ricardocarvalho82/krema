# Krema — Full-Width Opaque Dock Background

**Author:** Ricardo Carvalho
**Date:** November 12, 2026
**Base version:** Krema 0.13.0
**Environment:** CachyOS, KDE Plasma, Wayland
**Status:** Experimental modification — validated locally

## 1. Objective

Modify the Krema dock to display a full-width opaque background bar along the bottom edge of the screen, while preserving the original dock's centered icons and animations.

The modification should:

- Extend the background across the entire screen width.
- Preserve the original dock's icon positioning and animations.
- Allow regular windows to appear behind the opaque background.
- Preserve the reserved screen area for maximized windows.
- Continue functioning correctly after system suspension and resume.
- Keep the original Krema installation intact, allowing the modification to be reversed.

## 2. Implementation

**Modified file:** `src/qml/main.qml`

The implementation introduces a new QML `Rectangle` named `fullWidthBackground`, positioned behind the original dock panel.

```qml
// Faixa opaca experimental de largura total
Rectangle {
    id: fullWidthBackground
    x: 0
    y: dockPanel.y
    width: root.width
    height: dockPanel.height
    color: DockView.backgroundColor
    visible: !DockView.isVertical
    opacity: dockPanel.opacity
    z: dockPanel.z - 2
}
```

### 2.1 Design decisions

- **Full-screen width:** `root.width` makes the background span the available width of the dock's root QML item.
- **Vertical alignment:** The background follows `dockPanel.y` and `dockPanel.height`.
- **Consistent appearance:** `DockView.backgroundColor` reuses the dock's configured background color.
- **Horizontal dock only:** The background is visible only when `DockView.isVertical` is false.
- **Opacity synchronization:** The new background follows the original panel's opacity.
- **Layer ordering:** The lower `z` value places the new rectangle behind the original dock panel.

### 2.2 Scope

This modification changes the dock's visual presentation without changing its icon positioning or animation logic.

The original Krema executable remains untouched.

## 3. Build and Test Environment

The modified Krema executable was compiled locally from the Krema 0.13.0 source code using CMake.

**Project directory:** `~/projetos/krema-laboratorio`

**Build directory:** `build-lab/`

**Generated executable:** `build-lab/bin/krema`

### 3.1 Isolated testing

Before running the modified executable in the main KDE session, the application was tested inside a nested KWin Wayland compositor.

The isolated test environment used a resolution of 1280 × 720.

The following behaviors were visually verified:

- The opaque background extended across the screen width.
- Dock icons remained centered and animated.
- Application windows could appear behind the opaque background.
- The modified dock operated without requiring changes to the original system executable.

### 3.2 System protection

The original `/usr/bin/krema` executable was preserved.

A verified backup was maintained separately from the Git repository's tracked files.

The modified executable was built and tested independently before being activated in the main KDE session.

## 4. Reversible Deployment

The modified Krema executable was deployed to the main KDE Plasma session without replacing the system-installed binary.

### 4.1 Deployment strategy

The original executable remains available at:

`/usr/bin/krema`

The modified executable is located at:

`~/projetos/krema-laboratorio/build-lab/bin/krema`

Instead of modifying the system-wide autostart configuration, a user-specific override was created at:

`~/.config/autostart/com.bhyoo.krema.autostart.desktop`

The override changes the `Exec` entry to launch the locally compiled executable.

### 4.2 Activation

After creating the user-specific autostart override, the systemd user manager configuration was reloaded and the Krema autostart service was restarted.

The service was verified as active, with its `ExecStart` pointing to the modified executable.

### 4.3 Rollback strategy

To restore the original Krema executable, the user-specific autostart override can be removed or disabled, followed by reloading the systemd user manager and restarting the appropriate Krema service.

The original system package and executable remain unchanged.

**Note:** The rollback procedure should be verified before being used in an automated recovery script.

## 5. Validation Results

The modified Krema dock was tested on November 12, 2026.

| Test | Result |
|---|---|
| Local CMake compilation | PASS |
| Launch in nested KWin Wayland compositor | PASS |
| Full-width opaque background | PASS |
| Centered dock icons and animations | PASS |
| Window interaction behind the background | PASS |
| Launch in the main KDE Plasma session | PASS |
| Maximized window layout | PASS |
| System suspend and resume | PASS |
| Dock restoration after resume | PASS |
| Original system executable preserved | PASS |
| Fresh login autostart | PENDING |
| Rollback procedure | PENDING |

### 5.1 Suspend and resume validation

The computer was suspended using `systemctl suspend`.

After resuming, the modified Krema dock and the KDE Plasma desktop returned to their expected state without requiring manual intervention.

The opaque background, dock behavior, and window layout remained correct.

### 5.2 Final validation

The following additional checks were completed successfully:

- Confirmed automatic startup after a fresh KDE login.
- Verified rollback to the original packaged executable.
- Verified restoration of the experimental executable and autostart configuration.
