# Šta je u ovoj verziji

Finalna verzija sajta. Osnova je faza 3 (SEO, prerendering, optimizacija slika),
u koju su prenesene popravke iz faze 1 i kompletan besplatni test znanja iz faze 2.

Prethodne verzije ostaju netaknute: `derdiedas-centar/` (faza 3) i
`project-bolt-sb1-uufna5qj/derdiedas-faza2/` (faza 2).

---

## Pokretanje

```powershell
npm install
npm run dev                  # http://localhost:5173
```

Produkcijski build sa statičkim HTML-om za svaku rutu:

```powershell
npm install
npx playwright install chromium    # jednom, za prerender
npm run build:prerender
npm run preview                    # http://localhost:4173
```

`npm run build` radi i bez playwrighta — prerender se tada uredno preskoči, samo
pregled linka na Viberu i Facebooku ostaje bez naslova i slike.

Da bi se prijave i rezultati testa zaista čuvali:

```powershell
cp .env.example .env         # popuni VITE_SUPABASE_URL i VITE_SUPABASE_ANON_KEY
supabase db push
supabase functions deploy send-contact-email
supabase functions deploy send-test-result
```

Bez `.env` sajt radi normalno — i kontakt forma i test ponude slanje preko
WhatsAppa, Vibera ili e-maila sa već popunjenom porukom, pa se nijedan lead ne gubi.

---

## Prenesene popravke iz faze 1

**Dugme „Zakaži besplatni uvodni razgovor" nije radilo na telefonu.**
`ContactModal` je bio renderovan unutar `<Swiper>` komponente. Swiper na svoj
wrapper stavlja `transform`, što `position: fixed` pretvara u pozicioniranje u
odnosu na taj element, a `overflow-hidden` na slajderu ga je onda odsijecao. Na
laptopu se još naziralo, na telefonu je ispadalo da dugme ne radi. Modal sada ide
kroz portal u `<body>`, uz zatvaranje na ESC i zaključan skrol pozadine.

**Plutajuća Viber/WhatsApp/poziv dugmad.** Stajala su unutar `<SwiperSlide>`, pa
su bila zarobljena istim transformom: nisu pratila skrol, duplirala su se po
slajdu i postojala su samo na početnoj. Izdvojena su u `FloatingContact.tsx` i
renderuju se jednom po stranici, na svih devet ruta. Sva tri dugmeta su sada
48×48 sa ikonicom 22×22, a dugme za poziv je u brend plavoj umjesto žute, koja se
stapala sa glavnim CTA dugmetom.

**Favicon.** U `index.html` su stajala dva taga: jedan na `https://imgur.com/02Xxtpy`
— što je stranica galerije, ne slika, pa je browser dobijao HTML umjesto ikonice —
i Vite-ov podrazumijevani logo. Zato je u Google rezultatima stajala pogrešna
ikonica. Sada je brend oznaka (govorni oblačić sa tri tačke) u svim veličinama koje
traže browseri i Google, plus `site.webmanifest`.

## Prenesene primjedbe klijenta koje faza 3 nije pokrivala

**Kontakt sekcija.** Uklonjen isprekidani okvir, tekst skraćen sa dva pasusa na
dvije rečenice, sve ikonice u istom okviru od 40px (bile su izmiješano 32px i
36px), a adresa izdvojena iznad mape umjesto da bude zakopana na dnu liste.

**„Pripremna nastava" izgledala je šablonski isto kao „Zašto mi?"** — obje su
bile tamne kartice sa istim žutim krugom na vrhu. Sada su svijetle kartice sa
žutom linijom, ikonicom u zaobljenom kvadratu i listom stavki.

**Recenzije nisu bile poravnate.** Slajder je imao fiksnu visinu od 505px, a
kartice su uzimale visinu sadržaja — pa su avatari ispod mjehurića „skakali" iz
reda u red. Sada su kartice jednake visine i avatari su u istoj liniji. Usput je
uklonjena pozadinska tekstura koja se vukla sa `imgs.search.brave.com`, tuđeg
proxyja koji može nestati bilo kad.

**Hero.** Umjesto slajdera sa dvije slike sada je jedna statična. Klijent je to
sam predložio ako ubrzava učitavanje, a usput nestaje i Swiper sa prvog ekrana.
Dodan je prelaz ka sekciji ispod — gradijent i zakrivljeni razdjelnik umjesto
naglog reza, što je bila jedna od primjedbi.

## Nove optimizacije u ovoj verziji

**Suvišan preload na svakoj stranici.** Svih osam stranica je imalo
`preloadImage="https://i.imgur.com/..."` — sliku koju kod više ne koristi otkad
su slike lokalne. Svaka posjeta je povlačila jednu nepotrebnu sliku sa tuđeg
servera. Uklonjeno; `<Img priority>` i prerenderovani HTML to rade bolje.

**og:image je pokazivao na imgur.** Pregled linka na Viberu, Facebooku i
WhatsAppu zavisio je od tuđeg servera. Sada su to apsolutni URL-ovi lokalnih slika.

**Razmak ispod navbara.** Navbar je `h-20` (80px), a početna je ostavljala
`pt-16` (64px) — hero je ulazio 16px pod header.

**Kontakt forma je gubila podatke.** Forma šalje i „Šta vas zanima" i poruku, ali
ih `contact_requests` tabela nije imala, pa su se tiho odbacivali na upisu — a to
je najkorisniji dio prijave. Dodana je migracija i edge funkcija ih sada čuva i
šalje u e-mailu.

---

## Besplatni test znanja

Nova ruta `/test-znanja`, prenesena iz faze 2 i uklopljena u konvencije faze 3.

Posjetilac odgovara na 24 pitanja (po šest za A1, A2, B1 i B2), jedno po jedno,
sa trakom napretka i mogućnošću vraćanja. Na kraju ostavlja ime, prezime i email
(telefon je opcion) uz izričitu saglasnost, pa odmah vidi procijenjeni nivo,
rezultat po nivoima, preporuku programa sa linkom, dugme za zakazivanje razgovora
i pregled svih tačnih odgovora sa objašnjenjima.

Škola za svaki završen test dobija zapis u tabeli `test_results` i e-mail sa
nivoom i rezultatom. To je topao kontakt — osoba je sama zatražila procjenu.

Ulazi na test: istaknuto dugme u meniju (desktop i mobilni), drugo dugme na
heroju, baner na početnoj i link u footeru.

### Odluke koje vrijedi znati

**Nivo se računa kumulativno.** Priznaje se najviši savladan nivo samo ako su
savladani i svi niži (prag 4 od 6). Provjereno na osam scenarija, uključujući
„pogodio sva B2 pitanja, ne zna A1" — rezultat ostaje `ispod A1`.

**Tačan odgovor je u bazi pitanja uvijek prvi** radi lakšeg održavanja, ali se
opcije miješaju deterministički po id-u pitanja. Raspodjela po pozicijama je
7/4/7/6, bez prepoznatljivog šablona.

**Rezultat je iza forme, ali je to rečeno unaprijed** — uvodni ekran jasno kaže
da se na kraju ostavlja kontakt.

**Telefon nije obavezan.** Email je dovoljan, a obavezan telefon osjetno smanjuje
broj popunjenih formi.

**Plutajuća kontakt dugmad se ne prikazuju na stranici testa** — stajala su preko
ponuđenih odgovora i preko dugmeta „Dalje".

**`FloatingContact` namjerno NE koristi portal**, za razliku od modala. Prerender
upisuje gotov HTML u `dist/`, a `createRoot().render()` briše samo sadržaj `#root`
diva — portal u `<body>` bi ostao i dobili bi se dva kompleta dugmadi jedan preko
drugog. Komponenta se ionako renderuje na nivou stranice, gdje nema `transform`
pretka koji bi slomio `position: fixed`.

---

## Provjereno

Build prolazi sa nula TypeScript grešaka. Prerender upisuje svih devet ruta,
uključujući `/test-znanja`, a sitemap se generiše automatski iz `src/seo/pages.ts`.

U headless browseru na mobilnoj širini (390px) provjereno: modal se otvara i
vidljiv je, ESC ga zatvara, plutajuća dugmad prate skrol i nisu duplirana, sva tri
su 48×48 sa ikonicom 22×22, favicon i manifest se serviraju sa ispravnim tipom,
KONTAKT sa podstranice skroluje na kontakt sekciju, cijeli tok testa radi i kad
backend jeste i kad nije podešen, i nema horizontalnog scrolla.

## Preostalo

`goetheSecondary` nije u `imageManifest.json` — originalna slika je stajala na
`this.deakin.edu.au` i taj server odbija preuzimanje. Na stranici `/goethe` se i
dalje vuče sa tuđeg servera. Treba je zamijeniti vlastitom fotografijom.
