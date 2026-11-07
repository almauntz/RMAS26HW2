# Zadaća 2 – Jetpack Compose: stanje, unos i računanje

**Predmet:** Razvoj mobilnih aplikacija i servisa (RMAS), 2026/27
**Objava:** subota, 07.11.2026. u 23:00
**Rok za predaju:** subota, 14.11.2026. u 23:00 (zadnji commit na `main` granu)
**Odbrana:** uživo, u posebnom terminu van laba (termin se objavljuje na Google Classroomu)

Birate **jednu od dvije verzije zadaće**:

| Verzija | Tema | Bodovi |
|---|---|---|
| **A** | Složi svoju pizzu | 6 (+2 za bonus, poglavlje 4.3) |
| **B** | Podijeli račun | 8 |

Odabranu verziju upišite u `README.md` (polje `Verzija zadaće:`).

Uz upute dobijate **snimke ekrana referentnih aplikacija za obje verzije** (na Google Classroomu). Snimci pokazuju šta aplikacija treba raditi. **Izgled ne mora biti isti** (boje, raspored, ikone, gornja i donja traka su primjer, ne zahtjev), ali sva pravila iz uputa moraju raditi.

---

## 1. Pravila o AI-ju

- **AI smijete koristiti za izradu** zadaće.
- **Odbrana je bez AI-ja.** Morate objasniti svaki dio svog koda: gdje živi stanje, ko ga mijenja, zašto se ekran ponovo iscrta, i kako ste riješili pravila iz zadatka. Na odbrani dobijate i **dodatni zadatak**: promjenu jednog pravila, koju radite sami, uživo.
- Ako ne znate objasniti kod koji ste predali, zadaća se ne priznaje, bez obzira na to radi li aplikacija.

> Savjet: AI lako napravi ekran koji *izgleda* tačno. Teži dio su pravila koja utiču jedno na drugo. Zato provjerite svoju aplikaciju na **referentnim primjerima** na kraju svake verzije: vaš rezultat mora biti tačno isti, do feninga.

## 2. Ono što nismo radili na predavanjima

### 2.1 Komponente za unos

Komponente za unos nismo radili. **Dio zadaće je da ih sami pronađete i naučite iz dokumentacije:**

- `TextField` / `OutlinedTextField`
- `Checkbox`, `RadioButton`, `Switch`
- `FilterChip` (ili `SegmentedButton`)

Dokumentacija: https://developer.android.com/develop/ui/compose/components i https://developer.android.com/develop/ui/compose/text/user-input

Sve ove komponente rade po istom principu kao naše `WindowControls`: **dobijaju vrijednost i javljaju događaj** (`value` + `onValueChange`, `checked` + `onCheckedChange`, `selected` + `onClick`). Same ne pamte ništa: ako na događaj ne promijenite stanje, na ekranu se ništa ne promijeni. Na odbrani to morate znati objasniti.

Compose nema gotovu „grupu" RadioButtona. Grupu pravite sami, od više `RadioButton`-a.

### 2.2 Postavke projekta (Gradle)

- **Projekat pravite unutar svog repozitorija:** prvo klonirajte repozitorij (npr. **File → New → Project from Version Control** u Android Studiju ili `git clone`), a zatim **New Project → Empty Activity**, i kao **Save location** izaberite folder kloniranog repozitorija. Android Studio će upozoriti da folder nije prazan, i to je u redu. Fajlovi `README.md`, `UPUTE.md` i `INSTRUCTIONS.md` trebaju ostati u korijenu, pored foldera `app`.
- Jezik Kotlin, Minimum SDK kako je ponuđeno.
- **Ne treba nijedna dodatna biblioteka.** Template već uključuje sve što vam treba: Compose, Material 3, `@Preview` i JUnit za unit testove. Ne dodajte biblioteke koje vam AI predloži, osim ako znate zašto.
- **Ne nadograđujte verzije** usred zadaće. Žuta upozorenja „A newer version of … is available" možete ignorisati.
- **Ikone:** najlakše je desni klik na `res` → **New → Vector Asset → Clip art**. Ikona postaje fajl u projektu, bez ikakve biblioteke. Ako ipak koristite `Icons.Default...`, u `app/build.gradle.kts` dodajte `implementation("androidx.compose.material:material-icons-core")` (bez verzije, verziju daje Compose BOM).
- **`TopAppBar`** traži `@OptIn(ExperimentalMaterial3Api::class)` iznad funkcije koja ga koristi.
- **Provjera računanja:** `main` funkcija se u Android projektu ne može pokrenuti (greška „SourceSet with name 'main' not found"). Računanje provjerite **unit testom** u `src/test/...` (`ExampleUnitTest`), zelenom strelicom pored testa.
- **Rotacija ekrana nije dio zadaće.** Ne trebaju vam `ViewModel`, `@Parcelize` ni `Saver`.

## 3. Zajednički zahtjevi (obje verzije)

Ovo se boduje kod obje verzije.

### 3.1 Logika odvojena od UI-ja
- Model podataka i računanje su u **posebnom fajlu** koji **nema nijedan `androidx.compose` import**. To je čist Kotlin, kao u Zadaći 1.
- Računanje je **čista funkcija**: prima stanje, vraća rezultat, ništa ne mijenja.
- **Novac se računa u feninzima kao `Int`** (12,50 KM = 1250). Ne koristite `Double` za novac.
- Iznosi se prikazuju kao npr. „12,50 KM" (decimalni zarez, uvijek dvije decimale).
- Zaokruživanje procenata: **na najbliži fening, a pola feninga ide naviše** (npr. 1,875 KM → 1,88 KM).

### 3.2 Stanje

U aplikaciji razlikujemo **tri stvari**: podatke, stanje prikaza i rezultat. Svaka ima svoje mjesto.

**1. Podaci aplikacije** (šta je korisnik izabrao i unio: pizze, dodaci, kupon, osobe, stavke…)
- Hoistani su u **jedan** composable na vrhu ekrana, kao na predavanju o stanju. Komponente ispod njega su **stateless**.

**2. Stanje prikaza** (ovo nismo detaljno radili)

Neke stvari na ekranu moraju nešto pamtiti, a to nisu podaci aplikacije. Najčešći primjer je **scroll**.

- `Column` sam po sebi ne scrolla. Ako sadržaj ne stane na ekran, ono što je ispod se jednostavno ne vidi.
- Da bi se scrollao, dodaje mu se modifier za vertikalni scroll: `Modifier.verticalScroll(...)`. Taj modifier mora dobiti **stanje scrolla**, objekat koji pamti dokle je korisnik scrollao.
- To stanje se pravi s `rememberScrollState()`. Već u imenu je `remember`: ono je zapamćeno između iscrtavanja, isto kao `remember { mutableStateOf(...) }`.

```kotlin
Column(
    modifier = Modifier.verticalScroll(rememberScrollState())
) {
    // sadržaj ekrana
}
```

- **Zašto mora biti zapamćeno:** kad korisnik klikne checkbox, ekran se ponovo iscrta. Da se stanje scrolla pravi iznova pri svakom iscrtavanju, pozicija bi se vratila na 0 i ekran bi **skočio na vrh** nakon svakog klika.
- **Gdje živi:** u composableu koji scrolla, a ne na vrhu s podacima. Pozicija scrolla ne zanima nikog drugog: ni račun, ni kartice, ni računanje. Zato je ovo izuzetak od pravila „komponente nemaju `remember`". To pravilo važi za **podatke aplikacije**, a ne za stanje prikaza.

**3. Rezultat** (račun, iznosi)
- **Ne čuva se kao posebno stanje.** Računa se iz podataka svaki put kad se ekran iscrta. Na odbrani morate objasniti zašto je to bolje nego ga pamtiti i ručno ažurirati.

### 3.3 Liste
- Obje verzije imaju liste koje korisnik mijenja (dodaje i briše stavke).
- Prikaz liste: **`Column` + `forEach`**, sa scrollanjem (vidi 3.2). `LazyColumn` nije potreban.
- Liste u stanju se ne mijenjaju „na mjestu".

### 3.4 Kod i izgled
- Svaki composable koji crta nešto na ekranu ima `modifier: Modifier = Modifier` parametar.
- **Boje samo iz teme**, bez `Color.Red` i sličnih.
- **`@Preview` za svaku komponentu** s fiksnim podacima.
- Uz to, **dva `@Preview`-a cijelog ekrana** napunjena podacima iz dva referentna primjera (po vašem izboru).
- Projekat se otvara i pokreće u Android Studiju bez grešaka (Empty Activity, Kotlin, Jetpack Compose), u korijenu repozitorija.

---

## 4. Verzija A – Složi svoju pizzu

Aplikacija u kojoj korisnik sam slaže pizzu (nema gotovih vrsta poput Margarite) i pravi narudžbu od **jedne ili više pizza**, a račun se ažurira uživo dok mijenja izbor.

### 4.1 Cijene

**Osnovna cijena pizze.** Svaka pizza u osnovnoj cijeni ima **paradajz sos i sir**.

| Veličina | Cijena |
|---|---|
| S (mala) | 8,00 KM |
| M (srednja) | 11,00 KM |
| L (velika) | 14,00 KM |

**Dodaci.** Cijena dodatka zavisi od veličine pizze:

| Dodatak | Vrsta | S | M | L |
|---|---|---|---|---|
| gljive, masline, extra sir, pileći parizer, pileća salama | obični | 1,00 | 1,50 | 2,00 |
| suho meso, sudžuka, tuna | premium | 2,00 | 3,00 | 4,00 |

### 4.2 Pravila za jednu pizzu

1. **Veličina** S / M / L. Početna vrijednost je M.
2. **Maksimalan broj dodataka:** S do 2, M do 3, L do 5. Kad je limit dostignut, checkboxovi za ostale dodatke su **onemogućeni**.
3. **Smanjenje veličine:** ako pizza ima više dodataka nego što nova veličina dozvoljava, **uklanjaju se dodaci koji su zadnji dodani**, dok broj ne bude u limitu. Ispod pizze se prikaže poruka, npr. „Uklonjeno: tuna, sudžuka".
4. **Količina** s dugmadima − i +, od 1 do 10. Dugme − je onemogućeno na 1, a + na 10.

Cijena stavke = (osnovna cijena + dodaci) × količina.

### 4.3 Bonus (+2 boda): pola-pola

**Pola-pola** (Switch) postoji **samo za L**:
- Switch je onemogućen kad veličina nije L.
- Dok je pola-pola uključeno, S i M su onemogućeni.
- Svaka polovina ima svoje dodatke, **najviše 3 po polovini**.
- Dodatak na polovini košta **50% cijene za L** (obični 1,00 KM, premium 2,00 KM).
- Kad se pola-pola **uključi**, dosadašnji dodaci postaju dodaci lijeve polovine (najviše 3; višak se uklanja i prikaže poruka, kao u pravilu 4.2.3).
- Kad se pola-pola **isključi**, dodaci lijeve polovine postaju dodaci cijele pizze, a dodaci desne polovine se brišu.

### 4.4 Pravila za narudžbu

1. **Lista pizza:** dugme „Dodaj pizzu" dodaje novu pizzu (M, bez dodataka, količina 1). Svaka pizza ima dugme za brisanje, koje je onemogućeno kad je u narudžbi samo jedna pizza.
2. **Kupon** (TextField). Velika i mala slova nisu bitna, a razmaci na početku i kraju se ignorišu. Može se upisati samo jedan kupon:

   | Kod | Djelovanje |
   |---|---|
   | `STUDENT15` | 15% popusta na zbir pizza |
   | `MINUS5` | 5,00 KM popusta, **samo ako je zbir pizza najmanje 25,00 KM**. Inače se prikaže poruka „MINUS5 vrijedi od 25,00 KM" i popusta nema. |
   | `DOSTAVA0` | besplatna dostava |
   | prazno | bez kupona |
   | bilo šta drugo | poruka „Nepoznat kupon" **ispod polja**, a račun se računa bez kupona |

   Vrste kupona predstavite kao **sealed tip**.
3. **Dostava** (Switch) košta 3,00 KM. Dostava je **besplatna ako je iznos nakon popusta najmanje 30,00 KM**, ili ako je upisan `DOSTAVA0`. Pažnja: **popust se oduzima prije provjere praga od 30 KM.**

### 4.5 Prikaz računa

Račun stavku po stavku, uživo:
- za svaku pizzu: opis (veličina i dodaci, npr. „Srednja: gljive, tuna" ili „Srednja (sos, sir)" bez dodataka), količina, cijena stavke;
- zbir pizza;
- popust (ako ga ima);
- dostava (0,00 KM ako je besplatna, uz napomenu zašto);
- **ukupno**.

### 4.6 Referentni primjeri (verzija A)

Vaša aplikacija mora za ove ulaze dati tačno ove iznose.

| # | Narudžba | Dostava | Kupon | Zbir pizza | Popust | Dostava | **Ukupno** |
|---|---|---|---|---|---|---|---|
| A1 | 1× M: extra sir, tuna | ne | – | 15,50 | 0,00 | 0,00 | **15,50 KM** |
| A2 | 2× L: extra sir, gljive, sudžuka | da | `student15` | 44,00 | 6,60 | 0,00 | **37,40 KM** |
| A3 | 1× L: extra sir, tuna + 1× M: extra sir, gljive | da | `" MINUS5 "` | 34,00 | 5,00 | 3,00 | **32,00 KM** |
| A4 | 2× S: extra sir | da | `MINUS5` | 18,00 | 0,00 (poruka) | 3,00 | **21,00 KM** |
| A5 | 1× M: extra sir | da | `STUDENT15` | 12,50 | 1,88 | 3,00 | **13,62 KM** |
| A6 | 1× M bez dodataka | da | `DOSTAVA0` | 11,00 | 0,00 | 0,00 | **11,00 KM** |
| A7 | 1× S: suho meso | da | `PIZZA50` | 10,00 | 0,00 („Nepoznat kupon") | 3,00 | **13,00 KM** |
| A8 (bonus) | 1× L pola-pola (lijevo: extra sir, tuna · desno: gljive, masline, pileći parizer) + 1× M: extra sir, gljive | da | `MINUS5` | 34,00 | 5,00 | 3,00 | **32,00 KM** |

Pojedinačne cijene stavki: A1 15,50 · A2 22,00 × 2 · A3 20,00 i 14,00 · A4 9,00 × 2 · A5 12,50 · A6 11,00 · A7 10,00 · A8 20,00 i 14,00.

---

## 5. Verzija B – Podijeli račun

Društvo je bilo zajedno na ćevapima, pizzi ili kafi. Aplikacija dijeli račun: svaka osoba plaća ono što je naručila, zajedničke stavke se dijele, a na kraju svako vidi koliko plaća.

### 5.1 Osobe

1. Lista osoba, **od 1 do 8**. Na početku postoji jedna osoba („Osoba 1"). Dugme „Dodaj osobu" i dugme za brisanje kod svake osobe. Brisanje je onemogućeno kad je ostala samo jedna osoba, a dodavanje kad ih je 8.
2. Ime je TextField. Ime **ne smije biti prazno** (razmaci se ne računaju) i **ne smije se ponavljati** (velika i mala slova nisu bitna, „amar" i „Amar" su isto ime).
3. Kad se osoba obriše, briše se i iz svih stavki na kojima je bila.

### 5.2 Stavke računa

Svaka stavka ima:
- **naziv** (TextField, ne smije biti prazan);
- **cijenu po komadu** (TextField);
- **količinu** s dugmadima − i +, od 1 do 20;
- **ko dijeli stavku**: izbor za svaku osobu (npr. checkbox ili chip). Mora biti izabrana **barem jedna osoba**.

Dugme „Dodaj stavku" dodaje novu stavku (prazna, količina 1, nijedna osoba izabrana) i dugme za brisanje kod svake stavke.

**Cijena po komadu** je ispravna ako:
- ima samo cifre i najviše jedan decimalni separator (prihvata se **i zarez i tačka**: `12,5`, `12.50`, `3`);
- ima najviše dvije decimale;
- veća je od 0 i najviše 1000,00 KM.

Neispravna cijena prikazuje grešku **ispod tog polja** (npr. za `12,5a`: „Neispravna cijena"). Čitanje cijene iz teksta je posebna funkcija u logici, a rezultat (ispravna cijena ili greška) je **sealed tip**.

### 5.3 Opcije

- **Napojnica**: izbor 0%, 5%, 10%, 15% ili 20%. Početna vrijednost je 10%.
- **Zaokruži naviše** (Switch): iznos svake osobe se zaokružuje naviše na najbližih 0,50 KM.

### 5.4 Pravila računanja

Za svaku stavku:
1. `iznos stavke = cijena × količina`.
2. Iznos se dijeli **na jednake dijelove** među osobama koje dijele stavku. Ako se ne može podijeliti tačno na feninge, **preostali feninzi idu po jedan osobama redom kojim su u listi osoba** (prvoj, pa drugoj…). Primjer: 25,00 KM na troje daje 8,34 + 8,33 + 8,33.

Za svaku osobu:
3. `osnovica` = zbir njenih dijelova svih stavki.
4. `napojnica = osnovica × procenat` (zaokruženo na fening, pola naviše).
5. `ukupno = osnovica + napojnica`.
6. Ako je „Zaokruži naviše" uključeno: `konačno` = `ukupno` zaokruženo naviše na prvi višekratnik od 0,50 KM (ako je već višekratnik, ostaje isto). Inače je `konačno = ukupno`.

Zbirno:
- račun = zbir svih iznosa stavki;
- napojnica = **zbir napojnica osoba** (ne računa se posebno na cijeli račun);
- zaokruživanje = zbir (`konačno` − `ukupno`) svih osoba;
- **ukupno za platiti = zbir `konačno` svih osoba.**

### 5.5 Greške

Ako postoji **bilo koja greška** (prazno ili ponovljeno ime, prazan naziv stavke, neispravna cijena, stavka bez osobe), **rezultat se ne prikazuje**. Umjesto njega prikazuje se lista onoga što treba popraviti, npr.:
- „Osoba 3: ime se ponavlja"
- „Stavka 2 (Kola): neispravna cijena"
- „Stavka 3: nije izabrana nijedna osoba"

Greška se prikazuje i kod samog polja.

### 5.6 Prikaz rezultata

- Za svaku osobu: osnovica, napojnica, ukupno, i konačno (ako je uključeno zaokruživanje).
- Zbirno: račun, napojnica, zaokruživanje, **ukupno za platiti**.

### 5.7 Referentni primjeri (verzija B)

Osobe su u svim primjerima, ovim redom: **Amar, Lejla, Dino.**

**Stavke za B1 i B2:**

| Stavka | Cijena | Kol. | Ko dijeli |
|---|---|---|---|
| Ćevapi 10 | 12,00 | 1 | Amar |
| Pljeskavica | 10,50 | 1 | Lejla |
| Kafa | 2,00 | 3 | Amar, Lejla, Dino |
| Kola | 3,50 | 1 | Dino |

**B1:** napojnica 10%, zaokruživanje **ne**

| Osoba | Osnovica | Napojnica | Ukupno |
|---|---|---|---|
| Amar | 14,00 | 1,40 | 15,40 |
| Lejla | 12,50 | 1,25 | 13,75 |
| Dino | 5,50 | 0,55 | 6,05 |

Zbirno: račun 32,00 · napojnica 3,20 · zaokruživanje 0,00 · **ukupno 35,20 KM**

**B2:** napojnica 10%, zaokruživanje **da**

| Osoba | Osnovica | Napojnica | Ukupno | Konačno |
|---|---|---|---|---|
| Amar | 14,00 | 1,40 | 15,40 | 15,50 |
| Lejla | 12,50 | 1,25 | 13,75 | 14,00 |
| Dino | 5,50 | 0,55 | 6,05 | 6,50 |

Zbirno: račun 32,00 · napojnica 3,20 · zaokruživanje 0,80 · **ukupno 36,00 KM**

**Stavke za B3 i B4:**

| Stavka | Cijena | Kol. | Ko dijeli |
|---|---|---|---|
| Pizza | 25,00 | 1 | Amar, Lejla, Dino |
| Sok | 3,10 | 1 | Lejla, Dino |

**B3:** napojnica 15%, zaokruživanje **ne**

| Osoba | Osnovica | Napojnica | Ukupno |
|---|---|---|---|
| Amar | 8,34 | 1,25 | 9,59 |
| Lejla | 9,88 | 1,48 | 11,36 |
| Dino | 9,88 | 1,48 | 11,36 |

Zbirno: račun 28,10 · napojnica 4,21 · zaokruživanje 0,00 · **ukupno 32,31 KM**

**B4:** napojnica 15%, zaokruživanje **da**

| Osoba | Osnovica | Napojnica | Ukupno | Konačno |
|---|---|---|---|---|
| Amar | 8,34 | 1,25 | 9,59 | 10,00 |
| Lejla | 9,88 | 1,48 | 11,36 | 11,50 |
| Dino | 9,88 | 1,48 | 11,36 | 11,50 |

Zbirno: račun 28,10 · napojnica 4,21 · zaokruživanje 0,69 · **ukupno 33,00 KM**

**B5 (greške):** iste osobe i stavke kao B1, ali cijena za Kolu je `3,5a`, kod Kafe nije izabrana nijedna osoba, a treća osoba se zove `amar`. Rezultat se ne prikazuje. Lista grešaka sadrži: ponovljeno ime (kod obje osobe s tim imenom), stavku bez osobe (Kafa) i neispravnu cijenu (Kola).

---

## 6. Šta se predaje

U repozitoriju (na `main` grani, do roka):
- Android Studio projekat u korijenu repozitorija;
- `README.md` s upisanom verzijom zadaće, imenom i prezimenom, i **odgovorima na pitanja** iz README-a. Odgovori se pišu vlastitim riječima, po 2–5 rečenica. Na odbrani se o njima pita.

## 7. Bodovanje

- **Verzija A** nosi **6 bodova**. Bonus pola-pola, opisan u poglavlju **4.3**, nosi dodatna **2 boda** (ukupno najviše 8).
- **Verzija B** nosi **8 bodova**.

Bodovi se dodjeljuju **tek nakon uspješne odbrane**. Odbrana uključuje pitanja o kodu i dodatni zadatak koji radite sami, bez AI-ja.
