# Der Die Das – Centar za njemački jezik

Rekonstruisani izvorni kod sajta [derdiedascentar.com](https://www.derdiedascentar.com/).

Original je napravljen u Bolt.new i njegov codebase je izgubljen. Ovaj projekat je
dobijen dekompilacijom objavljenog production build-a (`index-B9jGyeOz.js`) nazad u
čitljive React komponente.

## Stack

| | |
|---|---|
| Build | Vite 5 |
| UI | React 18.3.1 + TypeScript |
| Stilovi | Tailwind CSS 3 (font: Poppins) |
| Rute | react-router-dom 6 |
| Slajderi | Swiper 11 (`Autoplay`, `Pagination`, `EffectFade`) |
| Ikone | lucide-react 0.344.0 |
| `<head>` tagovi | react-helmet |

## Pokretanje

```bash
npm install
cp .env.example .env      # opciono: Supabase i Google Analytics
npm run dev               # http://localhost:5173
npm run build             # production build u dist/
npm run images            # skine i optimizuje slike (jednom, treba internet)
npm run build:prerender   # build + statički HTML po ruti (vidi Faza 2)
```

## Struktura

```
src/
├── main.tsx                    # ulazna tačka
├── App.tsx                     # BrowserRouter + 8 ruta
├── index.css                   # Tailwind + custom klase slajdera
├── components/
│   ├── Navbar.tsx              # fiksna navigacija + mobilni meni
│   ├── HeroSlider.tsx          # naslovni Swiper + dugmad Viber/WhatsApp/telefon
│   ├── AboutSection.tsx        # "O nama"
│   ├── WhyUsSection.tsx        # "Zašto mi?" – 3 kartice
│   ├── ProgramsSection.tsx     # "Naši kursevi" – 4 kartice
│   ├── ExamPrepSection.tsx     # "Pripremna nastava" – ÖSD / Goethe
│   ├── TestimonialsSection.tsx # "Iskustva naših polaznika"
│   ├── ContactSection.tsx      # kontakt podaci + Google Maps
│   ├── ContactModal.tsx        # forma "Zakaži besplatni uvodni razgovor"
│   ├── OsdFaq.tsx              # FAQ akordeon na /osd
│   ├── GoetheFaq.tsx           # FAQ akordeon na /goethe
│   └── Footer.tsx
├── pages/                      # 8 stranica (Home + 7 podstranica)
├── data/testimonials.ts        # 5 recenzija polaznika
└── utils/scrollToTop.ts
```

## Rute

| Putanja | Stranica |
|---|---|
| `/` | `Home` |
| `/kursevi-za-odrasle` | `KurseviZaOdrasle` |
| `/kursevi-za-djecu` | `KurseviZaDjecu` |
| `/individualni-casovi-odrasli` | `IndividualniCasoviOdrasli` |
| `/individualni-casovi-djeca` | `IndividualniCasoviDjeca` |
| `/individualni-casovi-online` | `IndividualniCasoviOnline` |
| `/osd` | `Osd` |
| `/goethe` | `Goethe` |

## Vjernost originalu

Rebuild je poređen sa DOM-om snimljenim sa žive stranice:

- **465 / 465** DOM čvorova
- **0** razlika u tagovima i Tailwind klasama
- `index.js` 412.5 kB vs original 413.2 kB, `index.css` 44.1 kB vs 44.4 kB

Razlike koje postoje su isključivo posljedica toga što je original snimljen kroz
"Save page as" (apsolutni URL-ovi umjesto relativnih, lokalne kopije slika).

## Problemi zatečeni u objavljenom buildu

### 1. Kontakt forma ne radi — POPRAVLJENO

U objavljenom buildu poziv izgleda ovako:

```js
fetch("undefined/functions/v1/send-contact-email", {
  headers: { Authorization: "Bearer undefined", apikey: undefined }
})
```

`VITE_SUPABASE_URL` i `VITE_SUPABASE_ANON_KEY` nisu bili postavljeni u trenutku
build-a, pa je Vite ubacio doslovni `undefined`. Svaka prijava preko forme
"Zakaži besplatni uvodni razgovor" tiho pada.

Sada `src/lib/contact.ts` prvo provjerava da li je backend uopšte podešen.
Ako jeste, šalje na Supabase. Ako nije — ili ako Supabase ne odgovara — forma
ne pada, nego prikaže dugmad za slanje iste prijave preko WhatsApp-a, Viber-a
ili e-maila, sa već popunjenom porukom. Prijava se ne gubi ni u jednom slučaju.

### 2. KONTAKT u mobilnom meniju ne radi — POPRAVLJENO

Desktop dugme skroluje na `contact`, a mobilno na `reviews`:

```tsx
onClick={() => goToSection("/", "reviews")}   // mobilni meni
```

Element sa `id="reviews"` ne postojao je nigdje na sajtu – postoji samo `id="contact"`.
Sada oba dugmeta koriste `contact`.

### 3. `scrollToId` se nikad ne čita — POPRAVLJENO

`Navbar` je prosljeđivao `navigate(path, { state: { scrollToId } })`, ali nijedna
stranica nije čitala `location.state`. Sada to radi `src/components/RouteEffects.tsx`,
koji usput šalje i pageview za analitiku pri svakoj promjeni rute.

### 4. Linkovi `/OSD` i `/Goethe` vs rute `/osd` i `/goethe` — POPRAVLJENO

Radilo je jer react-router v6 podrazumijevano nije case-sensitive, ali je stvaralo
dvije verzije istog URL-a – loše za SEO i za analitiku. Svi linkovi su sada mali.

### 5. Sve slike su hotlinkovane sa tuđih domena — RIJEŠENO ALATOM

`i.imgur.com`, `residenciasarria.com`, `corwin-connect.com`,
`this.deakin.edu.au`. Ako neko od tih izvora ukloni fajl ili blokira
hotlinking, slika nestaje sa sajta. `npm run images` ih skida i lokalizuje —
vidi Fazu 3.

## Deploy

`public/_redirects` je dodat radi SPA rutiranja na Netlify-u – bez njega direktan
ulazak na `/osd` vraća 404. Za druge hostove treba ekvivalentan rewrite na
`index.html`.

## Faza 1 — popravke konverzije

Šta je dodato u odnosu na original:

| fajl | šta radi |
|---|---|
| `src/config/site.ts` | telefon, e-mail, adresa, društvene mreže i spisak kurseva na jednom mjestu |
| `src/lib/contact.ts` | slanje prijave sa fallback-om na WhatsApp / Viber / e-mail |
| `src/lib/analytics.ts` | GA4, potpuno neaktivan dok se ne upiše `VITE_GA_ID` |
| `src/components/ContactCta.tsx` | sekcija za prijavu — sada na **svih 8 stranica**, ne samo na početnoj |
| `src/components/RouteEffects.tsx` | skrol do sekcije + pageview pri promjeni rute |

Forma je proširena poljima **„Šta vas zanima"** (predizabrano prema stranici sa
koje se otvara) i **„Poruka"**. Ako vaša Supabase funkcija strogo validira ulaz,
u `src/lib/contact.ts` izbacite `courseInterest` i `message` iz `body`.

Događaji koji se mjere kad je GA uključen: `cta_open`, `form_submit`,
`form_fallback`, `call_click`, `whatsapp_click`, `viber_click`, `email_click`.

Provjereno automatskim testom (`e2e`): 15/15 — forma bez backenda nudi
alternativu, kurs se predizabira, sekcija za prijavu postoji na svih 8 stranica,
KONTAKT radi i na mobilnom i sa podstranice.

## Faza 2 — SEO

| fajl | šta radi |
|---|---|
| `src/seo/pages.ts` | naslov, opis i OG slika za svaku od 8 stranica — jedini izvor istine |
| `src/seo/structuredData.ts` | JSON-LD: `LanguageSchool`, `Course`, `FAQPage` |
| `src/components/Seo.tsx` | ubacuje sve to u `<head>` preko react-helmet |
| `scripts/generate-sitemap.mjs` | generiše `public/sitemap.xml` prije svakog builda |
| `scripts/prerender.mjs` | upisuje gotov HTML svake rute u `dist/` |
| `public/robots.txt` | pušta indeksiranje i pokazuje na sitemap |

Popravljeno usput: `<html lang>` je sada `bs`, početna ima jedan `<h1>` umjesto
dva (slajder je pravio po jedan po slajdu), hero slike imaju opisan `alt`, a na
`/osd` i `/goethe` uklonjene su doslovne markdown zvjezdice iz teksta
(`**ÖSD certifikat**` se prikazivalo sa zvjezdicama).

### Zašto prerender

`react-helmet` postavlja meta tagove tek kad se React pokrene. Facebook, Viber i
WhatsApp **ne pokreću JavaScript** kad prave pregled linka — pa dijeljen link na
bilo koju podstranicu nije imao ni naslov ni sliku. `npm run build:prerender`
upisuje gotov HTML u `dist/osd/index.html`, `dist/goethe/index.html` itd.

Traži playwright, koji namjerno **nije** u `package.json` da `npm install` ne bi
skidao browser svima. Ako nije instaliran, skripta se uredno preskoči:

```bash
npm i -D playwright && npx playwright install chromium
npm run build:prerender
```

Netlify sam servira te statičke fajlove prije `_redirects` pravila, pa
klijentsko rutiranje i dalje radi normalno.

### Provjereno

| | prije | poslije |
|---|---|---|
| jedinstven `<title>` | 1 za svih 8 stranica | 8 različitih |
| meta description | nema nijedne | 8, po jedna po stranici |
| Open Graph tagovi | 0 | 7 po stranici |
| canonical | nema | na svakoj stranici |
| `<h1>` na početnoj | 2 | 1 |
| structured data | nema | `LanguageSchool` + `Course` ×7 + `FAQPage` ×2 (8 pitanja svaki) |
| `<html lang>` | `en` | `bs` |

## Faza 3 — brzina

### Slike

`npm run images` skida svaku sliku sa `remote` URL-a iz
`src/config/imageSources.ts`, pravi WebP u više širina (400/800/1200/1600) plus
fallback u originalnom formatu, i upisuje dimenzije u `imageManifest.json`.
Komponenta `<Img>` onda servira `<picture>` sa `srcset`-om, a za sliku koja još
nije obrađena tiho koristi originalni URL — sajt radi u svakom slučaju.

U ovom paketu je **14 od 24 slike već obrađeno** (sve sa početne stranice).
Preostalih 10 su hero slike podstranica — nisu se mogle skinuti iz okruženja u
kojem je rebuild rađen. Pokreni `npm run images` sa internetom i obradiće i njih.

| | prije | poslije |
|---|---|---|
| slike na početnoj (mobilni) | 4.19 MB | **173 kB** |
| najveći pojedinačni fajl | 555 kB | 46 kB |

### Code splitting

Sve rute se učitavaju lijeno. Chunk trenutne rute se učitava **prije** prvog
rendera (`preloadCurrentRoute` u `main.tsx`), pa Suspense fallback nikad ne
trepne preko prerenderovanog sadržaja.

| | prije | poslije |
|---|---|---|
| JS na početnoj | 413 kB | 338 kB |
| JS na `/osd` | 413 kB | **235 kB** (bez Swiper-a) |
| CSS na `/osd` | 44 kB | 32 kB |
| ukupno `/osd` | — | 330 kB |

Uklonjen je i Google Font **DM Sans**, koji se učitavao a nigdje se nije
koristio — u CSS-u je samo Poppins.

### Pristupačnost

| | prije | poslije |
|---|---|---|
| elementi za klik manji od 44px (mobilni) | 27 | **0** |
| slike sa `alt="Hero"` ili bez opisa | 14 | 0 |
| dugmad bez pristupačnog imena | 1 po stranici | 0 |
| dropdown bez `aria-expanded` | 4 | 0 |
| Google Maps iframe bez `title` | 1 | 0 |
| preskoci u hijerarhiji naslova | 1 | 0 |

### Ispravka ranije procjene

U prvoj analizi sam naveo da nedostatak `width`/`height` na slikama pravi
poskakivanje sadržaja. **Izmjereno je da to nije bilo tačno**: CLS je bio
0.0000 i prije izmjena, jer sve slike stoje u kontejnerima fiksne visine
(`h-[75vh]`, `aspect-[16/9]`) sa `object-cover`. Dimenzije su svejedno dodate
jer ih `<Img>` dobija besplatno iz manifesta, ali to nije bio problem.
