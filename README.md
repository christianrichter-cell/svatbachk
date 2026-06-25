# Svatební web — Chris & Káťa

Statický onepager (svatební pozvánka). Jeden HTML soubor + jedna malá JS komponenta. Žádný build, žádné závislosti k instalaci — stačí naservírovat složku.

```
svatba-web/
├── index.html        # celá stránka (HTML + CSS + JS inline)
├── image-slot.js     # web-component pro drag&drop foto sloty v galerii
└── README.md         # tenhle soubor
```

---

## Spuštění lokálně

Protože `index.html` načítá `image-slot.js` přes `<script src>`, otevření přes `file://` může v některých prohlížečích blokovat. Spusť radši lokální server:

```bash
cd svatba-web
python3 -m http.server 8000
# → http://localhost:8000
```

## Nasazení na GitHub Pages

1. Vytvoř repozitář a nahraj obsah složky `svatba-web/` do kořene (nebo do `/docs`).
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch.**
3. Branch `main`, složka `/ (root)` (nebo `/docs` podle toho, kam jsi soubory dal).
4. Hotovo — web pojede na `https://<user>.github.io/<repo>/`.

Vlastní doména: přidej soubor `CNAME` s doménou a nastav DNS (A/ALIAS na GitHub Pages).

> Fonty se tahají z Google Fonts CDN — funguje to online out-of-the-box. Pokud chceš plně offline / self-hosted fonty, stáhni Cormorant Garamond, Pinyon Script, DM Sans, DM Mono a nahraď `<link>` v `<head>` lokálním `@font-face`.

---

## ⚠️ Co je placeholder a je potřeba doplnit

Projeď v `index.html` a nahraď reálnými daty:

| Co | Kde hledat | Teď tam je |
|---|---|---|
| Datum svatby | `data-tw="date_long"`, `date_short`, a `wedding_iso` v `TWEAKS` objektu | 29. 8. 2026 (placeholder — přelom srpna/září 2026) |
| Odpočet | `wedding_iso` v `const TWEAKS` (ISO formát) | `2026-08-29T13:00:00` |
| Adresa / mapa | sekce `#place`, odkaz „Otevřít v mapách" (`href="#"`) | jen název zříceniny |
| Časy programu | sekce `#program`, `.tl-row .time` | hrubý odhad |
| Menu A/B/C | sekce `#rsvp`, `.meal-card` | vymyšlené ukázkové pokrmy |
| Bankovní účet / IBAN / QR | sekce `#gifts`, `<dl>` | smyšlené číslo |
| Svědci + koordinátor | sekce `#contact` | „Doplníme jméno" + `+420 ___` |
| RSVP deadline | `data-tw="rsvp_deadline"` | 30. 6. 2026 |
| Fotky z cest | galerie `#gallery`, `<image-slot>` | prázdné drop-sloty |

Texty se opakují na víc místech (hero, nav, footer, tweaks panel). Nejjistější je hledat přes `data-tw="..."` atributy — všechny instance daného klíče se aktualizují stejnou hodnotou.

---

## 🔌 RSVP formulář — hlavní věc k dokódění

Teď je formulář **jen mock**: po odeslání se schová a ukáže poděkování, **nikam se nic neposílá**. Na statickém hostingu (GitHub Pages) nemáš backend, takže potřebuješ jednu z těchhle cest:

- **Formspree / Getform / Basin** — nejrychlejší. Stačí změnit `<form>` na `action="https://formspree.io/f/XXXX" method="POST"` a smazat `e.preventDefault()` handler (nebo nechat a poslat `fetch`). Sbírá i e-maily → můžeš posílat remindery.
- **Google Forms / Sheets** — postni data na Apps Script endpoint; zdarma, data rovnou v tabulce.
- **Vlastní endpoint** — pokud poběží i nějaká serverless funkce (Cloudflare Workers, Vercel function).

Formulář už sbírá vše potřebné: `name`, `email`, `attendance` (yes/no), `count`, `plus`, `meal` (a/b/c), `allergens`, `message`. Submit handler je dole v `<script>` u `getElementById('rsvp-form')`.

> Doporučení: server-side validace + honeypot proti spamu. Na e-mail reminder stačí ukládat `email` + `name` do tabulky.

---

## 🎛 Tweaks panel (poznámka)

V kódu je schovaný „Tweaks" panel (light/dark, editace textů, zapínání sekcí). Je to nástroj z prostředí, kde web vznikl — na ostrém webu **zůstane neviditelný** (čeká na zprávu z rodičovského okna, která nepřijde), takže nevadí. Když chceš čistý kód, klidně smaž `<aside class="tweaks-panel">` a související JS bloky (`persist`, `__edit_mode` listenery, `data-tw-*` handlery). Aplikace defaultních hodnot a odpočet jsou na nich nezávislé.

Light/dark přepínání jede přes `document.body.dataset.theme`. Pokud chceš na webu reálný přepínač pro hosty, napoj tlačítko na `applyTheme('dark'|'light')` a ulož volbu do `localStorage`.

---

## 🎨 Design tokeny (Varianta C — „Zřícenina")

Definované jako CSS proměnné v `:root` a `[data-theme="dark"]` na začátku `<style>`.

**Light**
- `--bg` `#E7E0CE` (zvětralý kámen) · `--paper` `#F0EADA`
- `--ink` `#25301F` (hluboká jehličnatá zeleň) · `--ink-soft` `#586049`
- `--accent` `#9E5A3C` (cihlová terakota) · `--accent-soft` `#C28B6E`
- `--rule` `rgba(37,48,31,.22)` · `--rule-soft` `rgba(37,48,31,.12)`

**Dark**
- `--bg` `#151B13` · `--paper` `#1E271B`
- `--ink` `#ECE3CF` · `--ink-soft` `#A6AE94`
- `--accent` `#CF9067` · `--accent-soft` `#8A6B52`

**Typografie**
- Display / nadpisy: **Cormorant Garamond**, italic, weight 300–400, letter-spacing ≈ −0.02em
- Jména (ligatura „&"): **Pinyon Script**
- Tělo / UI: **DM Sans**, weight 300
- Popisky / čísla / eyebrow: **DM Mono**, uppercase, letter-spacing 0.24em

**Layout**
- Max šířka obsahu `1180px` (`.wrap`), úzká `820px` (`.wrap-narrow`), padding 32px (22px mobil)
- Sekce: vertikální padding 110px (80px mobil)
- Breakpointy: 900 / 800 / 720 / 640 px

**Motiv**
- Hero: line-art kresba oblouku zříceniny + jehličnany (inline `<svg class="topo">`). Barva = `--accent`. Pozice/opacita v `.topo-1` / `.topo-2`.

---

## Struktura sekcí (pořadí v `<main>`)

`hero` → `#story` (o nás) → `#countdown` (živý odpočet) → `#program` (timeline) → `#place` (mapa) → `#dress` → `#gifts` → `#rsvp` → `#contact` → `#gallery` (foto sloty) → footer.

Každá sekce má `data-section="..."` (používá to vypínání sekcí v tweaks panelu) a `id` pro případnou navigaci/kotvy.

---

## Pozn. pro práci s Claude Code v terminálu

Tenhle `index.html` je **hotový, funkční design** — ne wireframe k překreslení. Ber ho jako zdroj pravdy pro vzhled. Realistické další kroky: napojit RSVP na backend, doplnit reálná data (tabulka výše), volitelně self-hostnout fonty, přidat hostům viditelný dark-mode přepínač, případně OG meta tagy + favicon pro sdílení pozvánky.
