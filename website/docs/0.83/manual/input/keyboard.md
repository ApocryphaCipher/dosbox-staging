# Keyboard

DOSBox Staging captures your keyboard input and passes it straight to the
emulated DOS environment --- most keys just work without any fuss.

The one wrinkle is that your host operating system intercepts certain key
combinations (++alt+tab++, ++cmd+h++, etc.) before DOSBox ever sees them.
The `keyboard_capture` setting lets you override this, so DOSBox gets first
dibs on those keys. This is especially useful in fullscreen mode or when a
game relies on key combinations that collide with OS shortcuts.

All key bindings can be customised through the [key mapper](keymapper.md)
(++ctrl+f1++ on Windows/Linux, ++cmd+f1++ on macOS). See
[Keyboard shortcuts](../appendices/shortcuts.md) for the full list of
default bindings.

## DOS keyboard layout and code pages

DOSBox Staging also emulates DOS's own keyboard layout and code page
(screen font) system, separate from the capture settings above. It's
managed from the DOS prompt with the `KEYB` and `CHCP` commands rather than
through configuration here --- see
[Keyboard layout and code pages](../system/localisation.md#keyboard-layout-and-code-pages)
for the full explanation.
