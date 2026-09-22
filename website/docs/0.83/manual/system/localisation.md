# Localisation

DOSBox Staging does not try to detect your host operating system's language,
country, or keyboard layout. It starts the way a typical PC did in the early
1990s: **US English**, the **US keyboard layout**, and **code page 437** ---
the character set of the original IBM PC and the default in MS-DOS 6.22.

DOS had no automatic regional detection. If you wanted another country's
formats or keyboard, you set it up yourself with commands like `COUNTRY` and
`KEYB`, and a machine without them behaved as a US machine. Most games and
programs were written and tested on such machines, and some depend on it. For
example, many draw their menus and borders with [box-drawing
characters](https://en.wikipedia.org/wiki/Box-drawing_characters) from code
page 437, and others expect US date and number formats. Under a different
setup, they can show garbled text or misbehave in less obvious ways.

Date, time, and number formatting follows the same idea. The default historic
[locale period](#locale_period) displays them the way a DOS PC of the era
would have. If you're playing a game in another language or want a different
format, the localisation settings discussed in this section let you change the
regional behaviour.

## Interface language

The [`language`](#language) setting controls the language of DOSBox Staging's
own interface messages (not the DOS programs themselves). It can be changed at
runtime, e.g. by running `language pl`, and has no effect on DOS programs, so
it doesn't matter for DOS compatibility.

The currently bundled translations are German, English, Spanish, French,
Italian, Dutch, Polish, Brazilian Portuguese, and Russian.

!!! warning

    DOSBox Staging's interface language translations are a community-based
    effort. Some translations may be incomplete or behind the current English
    version, so translated interface messages may occasionally be missing or
    out of date.


DOSBox Staging uses the [gettext](https://www.gnu.org/software/gettext/)
`.po` translation file format, which makes contributing translations
straightforward with tools like [Poedit](https://poedit.net/).

## Country and date/time formatting

The [`country`](#country) setting controls DOS-level formatting conventions:
date and time format, decimal separators, currency symbols, and so on.

The [`locale_period`](#locale_period) setting controls whether formatting
follows historic DOS conventions (how things looked on a real DOS PC of the
era), modern conventions (consistent with current-day practices).

## Keyboard layout and code pages

The [`keyboard_layout`](#keyboard_layout) setting selects the DOS keyboard
layout, determining which characters are produced by which keys. A layout can
include a code page suffix --- for example, `uk 850` selects the British
layout with a Western European screen font.

On a real MS-DOS, you must configure the keyboard layout and the screen font
separately; DOSBox Staging sets both from the provided layout and code.

Code pages control which character set is available on screen. DOSBox Staging
bundles the FreeDOS ISO, KOI, MAC, and WIN code page packages, providing broad
coverage of Latin, Cyrillic, and Greek scripts. After startup, use the `KEYB`
command to manage keyboard layouts and code pages (run `KEYB /?` for details)
, or the `CHCP` command to switch just the code page while keeping the current
keyboard layout (run `CHCP /?` for details).

To see what's available, start DOSBox Staging with the following command line
arguments:

<div class="compact" markdown>

- [`--list-countries`](../using-dosbox-staging/command-line.md#-list-countries)
  --- lists all supported countries with their numeric codes
- [`--list-layouts`](../using-dosbox-staging/command-line.md#-list-layouts)
  --- lists all supported keyboard layouts with their codes
- [`--list-code-pages`](../using-dosbox-staging/command-line.md#-list-code-pages)
  --- lists all bundled code pages (screen fonts)

</div>


## Configuration settings

### Interface language

You can set the interface language in the `[dosbox]` configuration section.

##### language

:   Select the language of DOSBox Staging's interface messages (`en` by
    default).

    Possible values are `de`, `en`, `es`, `fr`, `it`, `nl`, `pl`, `pt_BR`, and
    `ru`. 

    !!! note

        English is built-in; the rest is stored in the bundled
        `resources/translations` folder.

### Regional settings

You can set these in the `[dos]` configuration section.

##### country

:   Set DOS country code (`1` by default, which stands for US English). This
    affects country-specific information such as date, time, and decimal
    formats.

    !!! note

        The list of country codes can be displayed using the
        [`--list-countries`](../using-dosbox-staging/command-line.md#-list-countries)
        command-line argument.


##### keyboard\_layout

:   Keyboard layout code (`us` by default). The layout can be followed by the
    code page number; e.g., `uk 850` selects a Western European keyboard
    layout and screen font.

    !!! note "Notes"

        - On a real MS-DOS, you must configure the keyboard layout and the
          screen font separately; DOSBox Staging sets both from the provided
          layout and code.

        - The list of keyboard layout codes can be displayed using the
          [`--list-layouts`](../using-dosbox-staging/command-line.md#-list-layouts)
          command-line argument; e.g., `uk` is the British English layout.

        - The list of code pages can be displayed using the
          [`--list-code-pages`](../using-dosbox-staging/command-line.md#-list-code-pages)
          command-line argument; e.g., `437` is the original OEM-US code page.

        - Use the `KEYB` command to manage keyboard layouts and code pages at
          runtime (run `KEYB /?` for details).


##### locale\_period

:   Select which era of locale data to use.

    Possible values:

    <div class="compact" markdown>

    - `historic` -- If data is available for the given country, mimic old DOS
      behaviour when displaying time, dates, or numbers.

    - `modern` -- Follow current-day practices for a user experience more
      consistent with typical host systems.

    </div>
