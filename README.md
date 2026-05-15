# dbothur — Interactive Portfolio

> Portfolio interaktywne w stylu terminala / Terminal-style interactive portfolio
>
> **Live:** [dbothur.github.io/dbothur-interactive-portfolio](https://dbothur.github.io/dbothur-interactive-portfolio/) · Single-file · No dependencies · Vanilla JS

---

## Spis treści / Table of Contents

- [PL — Dokumentacja techniczna](#pl--dokumentacja-techniczna)
  - [Opis aplikacji](#opis-aplikacji)
  - [Struktura pliku](#struktura-pliku)
  - [Mechanizmy i funkcje](#mechanizmy-i-funkcje)
  - [Atrybuty data-testid](#atrybuty-data-testid)
- [EN — Technical Documentation](#en--technical-documentation)
  - [Application Overview](#application-overview)
  - [File Structure](#file-structure)
  - [Mechanisms and Functions](#mechanisms-and-functions)
  - [data-testid Attributes](#data-testid-attributes)

---

# PL — Dokumentacja techniczna

## Opis aplikacji

Interaktywne portfolio webowe w formie terminala. Użytkownik wpisuje komendy lub klika sekcje w panelu bocznym, aby zobaczyć informacje o autorze. Całość to **jeden plik HTML** bez żadnych zależności zewnętrznych (poza fontami Google i Nerd Fonts). Aplikacja nie wymaga serwera — działa bezpośrednio jako plik statyczny.

**Stack technologiczny:**
- Vanilla JavaScript (ES2020+, async/await)
- Vanilla CSS (custom properties, CSS Grid, Flexbox, keyframe animations)
- Brak bundlerów, frameworków, bibliotek JS

---

## Struktura pliku

```
index.html
├── <head>
│   ├── meta, title, Google Fonts, Nerd Fonts
│   └── <style> — cały CSS aplikacji
└── <body>
    ├── #app
    │   ├── <aside id="sidebar">   — panel boczny
    │   └── <main id="chat-main"> — obszar czatu
    ├── #sb-backdrop               — nakładka mobilna
    ├── #help-overlay              — modal pomocy/sekcji
    └── <script>                   — cała logika JS
```

---

## Mechanizmy i funkcje

### 1. System internacjonalizacji (i18n)

**Obiekt `T`** zawiera dwa słowniki kluczy: `pl` i `en`. Każdy klucz to string, HTML lub funkcja zwracająca interpolowany HTML.

```js
const T = {
  pl: { navAbout: 'O mnie', cmdNotFound: (cmd) => `...${cmd}...` },
  en: { navAbout: 'About Me', cmdNotFound: (cmd) => `...${cmd}...` }
};
```

| Funkcja | Opis |
|---|---|
| `l(key)` | Skrót: zwraca `T[currentLang][key]` |
| `applyLangUI(lang)` | Aktualizuje wszystkie elementy DOM: atrybuty `lang`, tekst przycisków, placeholder, elementy z atrybutem `data-i18n` |
| `switchLang(target?)` | Przełącza język (lub toggleuje jeśli brak argumentu), czyści czat i ponownie wywołuje `bootSequence()` |

Elementy HTML obsługujące tłumaczenia automatycznie posiadają atrybut `data-i18n="klucz"`.

---

### 2. System motywów (Light / Dark)

Motyw oparty na **CSS custom properties**. Tryb ciemny to wartości domyślne w `:root`, tryb jasny to nadpisania w selektorze `[data-theme="light"]`.

| Funkcja | Opis |
|---|---|
| `applyTheme(theme)` | Ustawia atrybut `data-theme` na `<html>`, zaznacza/odznacza toggle checkbox, zapisuje do `localStorage('theme')` |
| `toggleTheme()` | Odczytuje aktualny motyw i przełącza na przeciwny |

**Inicjalizacja (IIFE):** przy starcie odczytuje wartość z `localStorage`. Jeśli brak — wykrywa preferencje systemu przez `window.matchMedia('(prefers-color-scheme: light)')`.

Przejście między motywami jest animowane — elementy UI mają przypisane `transition: background-color .18s, color .18s, border-color .18s`.

---

### 3. System komend

Główny dyspozytor: `processCmd(raw)`. Wejście to surowy string z pola tekstowego lub kliknięcia przycisku.

**Przepływ:**
1. Trim + lowercase → `norm`
2. Jeśli nie zaczyna się od `/` → wyświetl błąd `cmdNotFound`
3. Dopasuj komendę → wykonaj akcję

**Dostępne komendy:**

| Komenda | Wymaga `/generate-data` | Opis |
|---|---|---|
| `/generate-data` | nie | Uruchamia animowany loader, ustawia `loaded = true` |
| `/about` | tak | Wyświetla sekcję „O mnie" |
| `/experience` | tak | Wyświetla doświadczenie zawodowe |
| `/skills` | tak | Wyświetla umiejętności techniczne |
| `/education` | tak | Wyświetla wykształcenie |
| `/languages` | tak | Wyświetla znajomość języków |
| `/contact` | tak | Wyświetla dane kontaktowe |
| `/whoami` | nie | Wyświetla skróconą wizytówkę |
| `/interests` | nie | Wyświetla zainteresowania |
| `/help` | nie | Wyświetla listę komend |
| `/clear` | nie | Czyści historię czatu (`msgsEl.innerHTML = ''`) |
| `/newchat` | nie | Reset sesji: czyści czat, resetuje `loaded`, ponownie uruchamia boot |
| `/lang [pl\|en]` | nie | Zmienia język interfejsu; bez argumentu pokazuje aktualny |

**Tablice pomocnicze:**
- `TAB_CMDS` — lista wszystkich komend do autouzupełniania
- `BUILDERS` — mapa `cmd → funkcja budująca HTML`
- `NAV_CMDS` — zestaw komend podświetlających element w nav

---

### 4. Budulce sekcji (Section Builders)

Każda sekcja ma dedykowaną funkcję zwracającą HTML.

| Funkcja | Guard `loaded` | Opis |
|---|---|---|
| `buildAbout()` | tak | Rola, opis, lista projektów, tło zawodowe |
| `buildContact()` | tak | Email, telefon, LinkedIn, GitHub |
| `buildSkills()` | tak | 8 kategorii umiejętności z kolorowymi tagami |
| `buildExperience()` | tak | Karty doświadczeń zawodowych z `l('jobs')` |
| `buildEducation()` | tak | Karta edukacji |
| `buildLanguages()` | tak | Lista języków z poziomami |
| `buildInterests()` | nie | Opis zainteresowań |
| `buildHelp()` | nie | Siatka komend z podziałem na kategorie |
| `buildWhoami()` | nie | Skrócona wizytówka: imię, rola, firma, lokalizacja |

**Guard `noData(name)`:** gdy `loaded === false` i sekcja tego wymaga, zwraca komunikat błędu z instrukcją uruchomienia `/generate-data`.

Funkcje pomocnicze renderowania:
- `e(s)` — escaping HTML (zamiana `<`, `>`, `&`)
- `tag(t, cls)` — buduje pojedynczy tag `<span class="tag cls">text</span>`
- `tags(arr, cls)` — wywołuje `tag()` dla każdego elementu tablicy

---

### 5. Loader modułów (`cmdNpx`)

Funkcja `cmdNpx()` symuluje ładowanie 6 modułów z animowanym outputem.

**Mechanizm:**
1. Tworzy element wiadomości z `id="_npx"`
2. Uruchamia `setInterval` (co 220ms)
3. Dla każdego modułu: losuje opóźnienie (80–200ms), dodaje linię z `✓` do outputu
4. Po załadowaniu wszystkich: ustawia `loaded = true`, aktualizuje przycisk w sidebarze (klasa `gen-done`, `disabled = true`, zmiana ikony i etykiety)

---

### 6. Panel boczny (Sidebar)

**Desktop:**
- `toggleSidebar()` dodaje klasę `genie-out` lub `collapsed`/`genie-in`
- Animacja "genie": CSS keyframes `@keyframes genie-out` (scale 1→0) i `@keyframes genie-in` (scale 0→1)
- Listener `animationend` czyści klasy po zakończeniu animacji

**Mobile (≤ 640px):**
- Sidebar jest pozycjonowany `fixed`, domyślnie przetłumaczony poza ekran (`translateX(-100%)`)
- Klasa `mobile-open` wysuwa sidebar (`translateX(0)`)
- Element `#sb-backdrop` (półprzezroczyste tło) aktywowany klasą `active`
- `closeMobileSidebar()` — zamyka sidebar i backdrop
- Automatyczne zamknięcie po kliknięciu elementu nav

**Inicjalizacja mobile (IIFE `initMobile`):**
- Ukrywa sidebar na małych ekranach, pokazuje przycisk `#sb-open`
- Nasłuchuje na `resize` — po przejściu do desktop resetuje wszystkie klasy mobilne

---

### 7. Overlay pomocy (Help Overlay)

Wielofunkcyjny modal (`#help-overlay` + `#help-panel`) może wyświetlać dowolną sekcję.

| Funkcja | Opis |
|---|---|
| `showOverlay(cmd)` | Ustawia tytuł z `OV_TITLES[cmd]`, buduje treść przez wywołanie `BUILDERS[cmd]()`, dodaje klasę `visible` do overlaya |
| `showHelpOverlay()` | Skrót: `showOverlay('help')` |
| `hideHelpOverlay()` | Usuwa klasę `visible` |
| `buildOverlayContent()` | Buduje siatkę komend + skróty klawiszowe (sekcja `ov-keyboard` ukryta na mobile) |

**Zdarzenia zamknięcia:**
- Kliknięcie poza panelem (event delegation na `#help-overlay`)
- Klawisz `Escape` (globalny listener `document.keydown`)

**Pierwsze uruchomienie:** `bootSequence()` wywołuje `showHelpOverlay()` z opóźnieniem 700ms tylko przy `firstLoad === true`.

---

### 8. Obsługa pola tekstowego

| Zdarzenie / Funkcja | Opis |
|---|---|
| `autoResize()` | Dynamicznie zmienia wysokość textarea (max 160px) |
| `Enter` (bez Shift) | Wysyła komendę: zapisuje do historii, czyści pole, wywołuje `processCmd()` |
| `Shift+Enter` | Nowa linia w textarea |
| `↑ / ↓` | Nawigacja po historii komend (`hist[]` + `hidx`) |
| `Tab` | Autouzupełnianie: filtruje `TAB_CMDS` po prefiksie; uzupełnia jeśli jest dokładnie jeden wynik |
| `sendEl click` | Przycisk wysyłania — ta sama logika co `Enter` |

**Historia:** tablica `hist[]` z nowym wpisem na początku (`unshift`), wskaźnik `hidx = -1` po wysłaniu.

---

### 9. Obsługa wiadomości

| Funkcja | Opis |
|---|---|
| `userMsg(text)` | Renderuje dymek użytkownika (wyrównany do prawej) z escapowanym tekstem |
| `asstMsg(html)` | Renderuje wiadomość asystenta z avatarem i imieniem; zwraca element DOM |
| `showSection(cmd)` | Async: dodaje animację "typing" (3 kropki) → po 520ms zastępuje treścią z `BUILDERS[cmd]()` + animacja `reveal` |
| `sc()` | Scrolla `#messages` na dół (`scrollTop = scrollHeight`) |

---

### 10. Sekwencja startowa (Boot Sequence)

`bootSequence()` wywoływana przy starcie i przy każdym resecie czatu:
1. Czeka 180ms (`sleep()`)
2. Renderuje ASCII art (baner "HELLO WORLD") + wynik `buildHelp()`
3. Auto-skaluje rozmiar fonta banera jeśli `scrollWidth > clientWidth`
4. Na mobile scrolluje do góry
5. Przy pierwszym uruchomieniu: pokazuje help overlay po 700ms

---

### 11. Responsywność (Mobile)

**Breakpoint:** `640px`

`syncPlaceholder()` używa **Canvas API** do pomiaru szerokości tekstu i przycina placeholder wielokropkiem jeśli jest za długi na ekranie mobilnym:

```js
ctx.font = '16px "JetBrains Mono",monospace';
const maxW = inp.clientWidth - 4;
// obcina znaki aż tekst + '...' zmieści się w maxW
```

---

## Atrybuty `data-testid`

Pełna lista atrybutów `data-testid` w HTML — podstawa dla stabilnych selektorów automatycznych:

| `data-testid` | Element | Opis |
|---|---|---|
| `app` | `#app` | Korzeń aplikacji |
| `sidebar` | `<aside>` | Panel boczny |
| `sidebar-top` | `.sb-top` | Górna część sidebara |
| `github-profile-link` | `<a>` | Link do profilu GitHub |
| `avatar-icon` | `.db-icon` | Kontener avatara |
| `avatar-img` | `<img>` | Obraz avatara |
| `profile-info` | `<div>` | Kontener imienia i roli |
| `profile-name` | `.sb-name` | Imię i nazwisko |
| `profile-role` | `.sb-role` | Rola zawodowa |
| `sidebar-toggle-btn` | `#sb-toggle` | Przycisk zamykania sidebara |
| `new-chat-btn` | `.sb-btn` | Przycisk „Nowy czat" |
| `new-chat-label` | `<span>` | Etykieta „Nowy czat" |
| `clear-chat-btn` | `.sb-btn-clear` | Przycisk „Wyczyść czat" |
| `generate-data-btn` | `#sb-gen-btn` | Przycisk „Generuj dane" |
| `general-label` | `.sb-label` | Nagłówek sekcji „Ogólne" |
| `nav-whoami` | `.nav-item` | Nawigacja „Kim jestem" |
| `nav-interests` | `.nav-item` | Nawigacja „Zainteresowania" |
| `cv-sections-label` | `.sb-label` | Nagłówek sekcji CV |
| `nav-about` | `.nav-item` | Nawigacja „O mnie" |
| `nav-experience` | `.nav-item` | Nawigacja „Doświadczenie" |
| `nav-skills` | `.nav-item` | Nawigacja „Umiejętności" |
| `nav-education` | `.nav-item` | Nawigacja „Wykształcenie" |
| `nav-languages` | `.nav-item` | Nawigacja „Języki" |
| `nav-contact` | `.nav-item` | Nawigacja „Kontakt" |
| `sidebar-footer` | `.sb-footer` | Stopka sidebara |
| `help-btn` | `.sb-btn` | Przycisk „Pomoc" |
| `chat-main` | `<main>` | Obszar czatu |
| `chat-header` | `#chat-header` | Nagłówek czatu |
| `sidebar-open-btn` | `#sb-open` | Przycisk otwierania sidebara |
| `header-icon` | `.hdr-icon` | Ikona GitHub w nagłówku |
| `header-title` | `.hdr-title` | Tytuł w nagłówku |
| `header-badge` | `.hdr-badge` | Badge roli |
| `theme-switch` | `<label>` | Kontener przełącznika motywu |
| `theme-toggle-input` | `<input>` | Checkbox przełącznika motywu |
| `theme-switch-track` | `.sw-track` | Tor przełącznika |
| `theme-switch-thumb` | `.sw-thumb` | Kciuk przełącznika |
| `theme-moon-icon` | `.sw-moon` | Ikona księżyca (tryb jasny) |
| `theme-sun-icon` | `.sw-sun` | Ikona słońca (tryb ciemny) |
| `lang-btn` | `#lang-btn` | Przycisk zmiany języka |
| `messages-container` | `#messages` | Kontener wiadomości |
| `input-area` | `#input-area` | Obszar inputu |
| `input-box` | `#input-box` | Kontener textarea i przycisku |
| `chat-input` | `#chat-input` | Pole tekstowe |
| `send-btn` | `#send-btn` | Przycisk wysyłania |
| `input-hint` | `.input-hint` | Podpowiedź skrótów klawiszowych |

---

## Instrukcja testów automatycznych

### Dla AI — jak zaplanować architekturę testów automatycznych

#### Kontekst i strategia

Aplikacja jest **jednotlikowym SPA bez routingu**. Nie ma wywołań API do backendu — wszystko dzieje się lokalnie w przeglądarce. Kluczowe dla testów jest zrozumienie **stanu aplikacji**, który zmienia się przez komendy.

**Zmienna stanu `loaded`** to flaga (boolean), która dzieli komendy na dwie grupy — testy muszą to uwzględniać.

**Rekomendowany runner:** [Playwright](https://playwright.dev/) — strona sama używa go w CI; ma natywną obsługę `data-testid` i `getByTestId()`.

---

#### Architektura testów

```
tests/
├── fixtures/
│   └── portfolio.fixture.ts     # Page fixture z page object
├── pages/
│   └── PortfolioPage.ts         # Page Object Model
├── helpers/
│   └── chatHelpers.ts           # Wspólne funkcje: sendCmd, waitForSection
└── specs/
    ├── boot.spec.ts             # Sekwencja startowa
    ├── theme.spec.ts            # Motywy
    ├── language.spec.ts         # Przełączanie języka
    ├── commands-basic.spec.ts   # Komendy nie wymagające loaded
    ├── commands-data.spec.ts    # Komendy wymagające /generate-data
    ├── generate-data.spec.ts    # Loader modułów
    ├── sidebar.spec.ts          # Panel boczny (desktop + mobile)
    ├── overlay.spec.ts          # Overlay / modal
    └── input.spec.ts            # Obsługa pola tekstowego
```

---

#### Page Object Model — `PortfolioPage.ts`

```typescript
// Przykładowa struktura klasy — AI powinien ją wypełnić metodami
class PortfolioPage {
  // Lokatory oparte na data-testid (stabilne przy refaktorze CSS/struktury)
  readonly chatInput    = this.page.getByTestId('chat-input');
  readonly sendBtn      = this.page.getByTestId('send-btn');
  readonly messages     = this.page.getByTestId('messages-container');
  readonly generateBtn  = this.page.getByTestId('generate-data-btn');
  readonly langBtn      = this.page.getByTestId('lang-btn');
  readonly themeToggle  = this.page.getByTestId('theme-toggle-input');
  readonly sidebar      = this.page.getByTestId('sidebar');
  readonly helpOverlay  = this.page.locator('#help-overlay');

  // Metody pomocnicze
  async sendCommand(cmd: string) { ... }
  async generateData() { ... }      // wysyła /generate-data, czeka na zakończenie
  async waitForSection(title: string) { ... }
  async getLastMessage(): Promise<string> { ... }
  async switchToEnglish() { ... }
}
```

**Zasada:** AI powinien wygenerować **lokatory wyłącznie przez `data-testid`** — nigdy przez klasy CSS (`.sb-btn-gen`) ani selektory pozycyjne (`nth-child`), bo te zmieniają się przy refaktorze stylów.

---

---

#### Ważne wskazówki dla AI przy generowaniu testów

1. **URL strony** — strona jest hostowana na GitHub Pages: `https://dbothur.github.io/dbothur-interactive-portfolio/`

2. **Czekaj na animacje** — `showSection()` ma 520ms delay; użyj `waitForSelector` lub `toBeVisible` z odpowiednim timeoutem zamiast twardych `sleep`.

3. **Stan `loaded` jest globalny** — jeśli test uruchamia `/generate-data`, ten stan pozostaje przez cały test (brak izolacji między testami w tej samej sesji). Każdy test powinien startować od świeżej strony (`page.reload()` lub nowy context).

4. **CSS transitions ≠ gotowość treści** — zmiana motywu jest natychmiastowa w DOM, CSS transition trwa 180ms. Testuj atrybut `data-theme`, nie stan wizualny.

5. **Selektory i pierwszeństwo:**
   - Priorytet 1: `getByTestId('...')` — zawsze stabilne
   - Priorytet 2: `getByRole(...)` z `name` — semantycznie poprawne
   - Priorytet 3: `.locator('#id')` — dla elementów z ID
   - Unikaj: klas CSS, indeksów potomnych

6. **Test `/newchat` i `/clear`:** po tych komendach sprawdź czy `messages-container` jest pusty lub zawiera tylko boot sequence.

7. **Test języka:** sprawdź atrybut `lang` na `<html id="html-root">` (zmienia się między `pl` i `en`).

---

# EN — Technical Documentation

## Application Overview

A terminal-style interactive web portfolio. Users type commands or click sidebar sections to view information about the author. The entire application is a **single HTML file** with no external JavaScript dependencies (only Google Fonts and Nerd Fonts). No server is required — it runs as a static file.

**Tech stack:**
- Vanilla JavaScript (ES2020+, async/await)
- Vanilla CSS (custom properties, CSS Grid, Flexbox, keyframe animations)
- No bundlers, frameworks, or JS libraries

---

## File Structure

```
index.html
├── <head>
│   ├── meta, title, Google Fonts, Nerd Fonts
│   └── <style> — all application CSS
└── <body>
    ├── #app
    │   ├── <aside id="sidebar">   — sidebar panel
    │   └── <main id="chat-main"> — chat area
    ├── #sb-backdrop               — mobile overlay backdrop
    ├── #help-overlay              — help/section modal
    └── <script>                   — all JS logic
```

---

## Mechanisms and Functions

### 1. Internationalization System (i18n)

The **`T` object** holds two key dictionaries: `pl` and `en`. Each value is either a string, raw HTML, or a function returning interpolated HTML.

```js
const T = {
  pl: { navAbout: 'O mnie', cmdNotFound: (cmd) => `...${cmd}...` },
  en: { navAbout: 'About Me', cmdNotFound: (cmd) => `...${cmd}...` }
};
```

| Function | Description |
|---|---|
| `l(key)` | Shorthand: returns `T[currentLang][key]` |
| `applyLangUI(lang)` | Updates all DOM elements: `lang` attribute, button labels, placeholder, elements with `data-i18n` attribute |
| `switchLang(target?)` | Switches language (or toggles if no argument), clears chat, re-runs `bootSequence()` |

HTML elements with translatable text carry the `data-i18n="key"` attribute for automatic updates.

---

### 2. Theme System (Light / Dark)

The theme is driven entirely by **CSS custom properties**. Dark mode is the default in `:root`; light mode overrides live in `[data-theme="light"]`.

| Function | Description |
|---|---|
| `applyTheme(theme)` | Sets `data-theme` on `<html>`, checks/unchecks the toggle checkbox, saves to `localStorage('theme')` |
| `toggleTheme()` | Reads current theme and flips it |

**Initialization (IIFE):** on startup reads `localStorage`. If absent — detects system preference via `window.matchMedia('(prefers-color-scheme: light)')`.

Theme transitions are animated — UI elements have `transition: background-color .18s, color .18s, border-color .18s`.

---

### 3. Command System

Main dispatcher: `processCmd(raw)`. Input is a raw string from the text field or a sidebar button click.

**Flow:**
1. Trim + lowercase → `norm`
2. If doesn't start with `/` → display `cmdNotFound` error
3. Match command → execute action

**Available commands:**

| Command | Requires `/generate-data` | Description |
|---|---|---|
| `/generate-data` | no | Runs animated loader, sets `loaded = true` |
| `/about` | yes | Displays "About Me" section |
| `/experience` | yes | Displays work experience |
| `/skills` | yes | Displays technical skills |
| `/education` | yes | Displays education |
| `/languages` | yes | Displays language proficiency |
| `/contact` | yes | Displays contact information |
| `/whoami` | no | Displays short profile card |
| `/interests` | no | Displays interests & hobbies |
| `/help` | no | Displays command list |
| `/clear` | no | Clears chat history (`msgsEl.innerHTML = ''`) |
| `/newchat` | no | Session reset: clears chat, resets `loaded`, re-runs boot |
| `/lang [pl\|en]` | no | Changes interface language; without argument shows current |

**Helper arrays/objects:**
- `TAB_CMDS` — full command list for Tab autocomplete
- `BUILDERS` — map of `cmd → HTML-building function`
- `NAV_CMDS` — set of commands that highlight a nav item

---

### 4. Section Builders

Each section has a dedicated function that returns HTML markup.

| Function | `loaded` guard | Description |
|---|---|---|
| `buildAbout()` | yes | Role, description, project list, professional background |
| `buildContact()` | yes | Email, phone, LinkedIn, GitHub |
| `buildSkills()` | yes | 8 skill categories with colored tags |
| `buildExperience()` | yes | Experience cards from `l('jobs')` |
| `buildEducation()` | yes | Education card |
| `buildLanguages()` | yes | Language list with proficiency levels |
| `buildInterests()` | no | Interests description |
| `buildHelp()` | no | Command grid split by category |
| `buildWhoami()` | no | Short card: name, role, company, location |

**`noData(name)` guard:** when `loaded === false` and a section requires it, returns an error message with instructions to run `/generate-data`.

Rendering utilities:
- `e(s)` — HTML escaping (`<`, `>`, `&`)
- `tag(t, cls)` — builds a single `<span class="tag cls">text</span>`
- `tags(arr, cls)` — calls `tag()` for every array element

---

### 5. Module Loader (`cmdNpx`)

The `cmdNpx()` function simulates loading 6 modules with animated terminal output.

**Mechanism:**
1. Creates a message element with `id="_npx"`
2. Runs a `setInterval` (every 220ms)
3. For each module: picks a random delay (80–200ms), appends a `✓` line to the output
4. After all modules: sets `loaded = true`, updates sidebar button (adds `gen-done` class, `disabled = true`, changes icon and label)

---

### 6. Sidebar

**Desktop:**
- `toggleSidebar()` adds class `genie-out` or `collapsed`/`genie-in`
- "Genie" animation: CSS keyframes `@keyframes genie-out` (scale 1→0) and `@keyframes genie-in` (scale 0→1)
- `animationend` listener cleans up classes after animation

**Mobile (≤ 640px):**
- Sidebar is `position: fixed`, translated off-screen by default (`translateX(-100%)`)
- Class `mobile-open` slides the sidebar in (`translateX(0)`)
- `#sb-backdrop` (semi-transparent overlay) activated by class `active`
- `closeMobileSidebar()` — closes sidebar and backdrop
- Auto-close after tapping any nav item

**Mobile initialization (IIFE `initMobile`):**
- Hides sidebar on small screens, shows `#sb-open` button
- Listens to `resize` — on switch to desktop resets all mobile classes

---

### 7. Help Overlay

A multi-purpose modal (`#help-overlay` + `#help-panel`) capable of displaying any section.

| Function | Description |
|---|---|
| `showOverlay(cmd)` | Sets title from `OV_TITLES[cmd]`, builds body via `BUILDERS[cmd]()`, adds class `visible` to overlay |
| `showHelpOverlay()` | Shorthand: `showOverlay('help')` |
| `hideHelpOverlay()` | Removes class `visible` |
| `buildOverlayContent()` | Builds command grid + keyboard shortcuts (`ov-keyboard` section hidden on mobile) |

**Close triggers:**
- Clicking outside the panel (event delegation on `#help-overlay`)
- `Escape` key (global `document.keydown` listener)

**First load:** `bootSequence()` calls `showHelpOverlay()` with 700ms delay only when `firstLoad === true`.

---

### 8. Input Field Handling

| Event / Function | Description |
|---|---|
| `autoResize()` | Dynamically adjusts textarea height (max 160px) |
| `Enter` (no Shift) | Sends command: saves to history, clears field, calls `processCmd()` |
| `Shift+Enter` | New line in textarea |
| `↑ / ↓` | Navigate command history (`hist[]` + `hidx`) |
| `Tab` | Autocomplete: filters `TAB_CMDS` by prefix; fills in if exactly one match |
| `sendEl click` | Send button — same logic as `Enter` |

**History:** `hist[]` array with new entries prepended (`unshift`), pointer `hidx = -1` after sending.

---

### 9. Message Rendering

| Function | Description |
|---|---|
| `userMsg(text)` | Renders a right-aligned user bubble with escaped text |
| `asstMsg(html)` | Renders an assistant message with avatar and name; returns the DOM element |
| `showSection(cmd)` | Async: adds "typing" animation (3 dots) → after 520ms replaces with `BUILDERS[cmd]()` content + `reveal` animation |
| `sc()` | Scrolls `#messages` to bottom (`scrollTop = scrollHeight`) |

---

### 10. Boot Sequence

`bootSequence()` is called on startup and on every chat reset:
1. Waits 180ms (`sleep()`)
2. Renders ASCII art banner ("HELLO WORLD") + result of `buildHelp()`
3. Auto-scales banner font size if `scrollWidth > clientWidth`
4. On mobile scrolls to top
5. On first load: shows help overlay after 700ms

---

### 11. Responsiveness (Mobile)

**Breakpoint:** `640px`

`syncPlaceholder()` uses the **Canvas API** to measure text width and truncates the placeholder with ellipsis if it's too long for the mobile screen:

```js
ctx.font = '16px "JetBrains Mono",monospace';
const maxW = inp.clientWidth - 4;
// trims characters until text + '...' fits within maxW
```

---

## `data-testid` Attributes

Complete list of `data-testid` attributes in the HTML — the foundation for stable automation selectors:

| `data-testid` | Element | Description |
|---|---|---|
| `app` | `#app` | Application root |
| `sidebar` | `<aside>` | Sidebar panel |
| `sidebar-top` | `.sb-top` | Top section of sidebar |
| `github-profile-link` | `<a>` | Link to GitHub profile |
| `avatar-icon` | `.db-icon` | Avatar container |
| `avatar-img` | `<img>` | Avatar image |
| `profile-info` | `<div>` | Name and role container |
| `profile-name` | `.sb-name` | Full name |
| `profile-role` | `.sb-role` | Job role |
| `sidebar-toggle-btn` | `#sb-toggle` | Sidebar close button |
| `new-chat-btn` | `.sb-btn` | "New Chat" button |
| `new-chat-label` | `<span>` | "New Chat" label |
| `clear-chat-btn` | `.sb-btn-clear` | "Clear Chat" button |
| `generate-data-btn` | `#sb-gen-btn` | "Generate Data" button |
| `general-label` | `.sb-label` | "General" section heading |
| `nav-whoami` | `.nav-item` | "Who am I" nav item |
| `nav-interests` | `.nav-item` | "Interests" nav item |
| `cv-sections-label` | `.sb-label` | "CV Sections" heading |
| `nav-about` | `.nav-item` | "About Me" nav item |
| `nav-experience` | `.nav-item` | "Experience" nav item |
| `nav-skills` | `.nav-item` | "Skills" nav item |
| `nav-education` | `.nav-item` | "Education" nav item |
| `nav-languages` | `.nav-item` | "Languages" nav item |
| `nav-contact` | `.nav-item` | "Contact" nav item |
| `sidebar-footer` | `.sb-footer` | Sidebar footer |
| `help-btn` | `.sb-btn` | "Help" button |
| `chat-main` | `<main>` | Chat area |
| `chat-header` | `#chat-header` | Chat header |
| `sidebar-open-btn` | `#sb-open` | Sidebar open button |
| `header-icon` | `.hdr-icon` | GitHub icon in header |
| `header-title` | `.hdr-title` | Header title |
| `header-badge` | `.hdr-badge` | Role badge |
| `theme-switch` | `<label>` | Theme toggle container |
| `theme-toggle-input` | `<input>` | Theme toggle checkbox |
| `theme-switch-track` | `.sw-track` | Toggle track |
| `theme-switch-thumb` | `.sw-thumb` | Toggle thumb |
| `theme-moon-icon` | `.sw-moon` | Moon icon (light mode active) |
| `theme-sun-icon` | `.sw-sun` | Sun icon (dark mode active) |
| `lang-btn` | `#lang-btn` | Language switch button |
| `messages-container` | `#messages` | Messages container |
| `input-area` | `#input-area` | Input area |
| `input-box` | `#input-box` | Textarea + button wrapper |
| `chat-input` | `#chat-input` | Text input field |
| `send-btn` | `#send-btn` | Send button |
| `input-hint` | `.input-hint` | Keyboard shortcut hint |

---

## Automated Testing Guide

### For AI — How to Plan the Test Automation Architecture

#### Context and Strategy

This application is a **single-file SPA with no routing**. There are no backend API calls — everything runs locally in the browser. The critical concept for testing is understanding the **application state**, which changes through commands.

**The `loaded` flag** (boolean) divides commands into two groups — tests must account for this.

**Recommended runner:** [Playwright](https://playwright.dev/) — the portfolio itself lists Playwright as a tool; it has native `data-testid` support via `getByTestId()`.

---

#### Test Architecture

```
tests/
├── fixtures/
│   └── portfolio.fixture.ts     # Page fixture with page object
├── pages/
│   └── PortfolioPage.ts         # Page Object Model
├── helpers/
│   └── chatHelpers.ts           # Shared helpers: sendCmd, waitForSection
└── specs/
    ├── boot.spec.ts             # Boot sequence
    ├── theme.spec.ts            # Themes
    ├── language.spec.ts         # Language switching
    ├── commands-basic.spec.ts   # Commands that don't require loaded state
    ├── commands-data.spec.ts    # Commands requiring /generate-data
    ├── generate-data.spec.ts    # Module loader
    ├── sidebar.spec.ts          # Sidebar (desktop + mobile)
    ├── overlay.spec.ts          # Overlay / modal
    └── input.spec.ts            # Input field behavior
```

---

#### Page Object Model — `PortfolioPage.ts`

```typescript
// Suggested class structure — AI should fill in the method bodies
class PortfolioPage {
  // Locators based on data-testid (stable across CSS/structure refactors)
  readonly chatInput    = this.page.getByTestId('chat-input');
  readonly sendBtn      = this.page.getByTestId('send-btn');
  readonly messages     = this.page.getByTestId('messages-container');
  readonly generateBtn  = this.page.getByTestId('generate-data-btn');
  readonly langBtn      = this.page.getByTestId('lang-btn');
  readonly themeToggle  = this.page.getByTestId('theme-toggle-input');
  readonly sidebar      = this.page.getByTestId('sidebar');
  readonly helpOverlay  = this.page.locator('#help-overlay');

  // Helper methods
  async sendCommand(cmd: string) { ... }
  async generateData() { ... }      // sends /generate-data, waits for completion
  async waitForSection(title: string) { ... }
  async getLastMessage(): Promise<string> { ... }
  async switchToEnglish() { ... }
}
```

**Rule:** AI should generate **locators exclusively via `data-testid`** — never via CSS classes (`.sb-btn-gen`) or positional selectors (`nth-child`), as these break during style refactoring.

---

---

#### Important Notes for AI When Generating Tests

1. **Page URL** — the page is hosted on GitHub Pages: `https://dbothur.github.io/dbothur-interactive-portfolio/`

2. **Wait for animations** — `showSection()` has a 520ms delay; use `waitForSelector` or `toBeVisible` with appropriate timeout instead of hard-coded `sleep`.

3. **`loaded` state is global** — once a test runs `/generate-data`, that state persists for the entire test session (no isolation between tests). Each test should start from a fresh page (`page.reload()` or a new browser context).

4. **CSS transitions ≠ DOM readiness** — theme changes are immediate in the DOM, but CSS transition lasts 180ms. Test the `data-theme` attribute, not the visual state.

5. **Selector priority:**
   - Priority 1: `getByTestId('...')` — always stable
   - Priority 2: `getByRole(...)` with `name` — semantically correct
   - Priority 3: `.locator('#id')` — for elements with an ID
   - Avoid: CSS classes, child index selectors

6. **Testing `/newchat` and `/clear`:** after these commands verify that `messages-container` is empty or contains only the boot sequence.

7. **Testing language:** check the `lang` attribute on `<html id="html-root">` (toggles between `pl` and `en`).

---

*Dokumentacja wygenerowana na podstawie `index.html` / Documentation generated from `index.html`*
