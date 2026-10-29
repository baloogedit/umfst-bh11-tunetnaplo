# Tünetnapló, ami csak egérrel működik – hibariport

Repó: https://github.com/baloogedit/umfst-bh11-tunetnaplo
Készítette: Balog Edit, Info III

Cél: az űrlap feleljen meg a WCAG 2.2 AA szintnek.
Tesztkörnyezet: Windows, Google Chrome (DevTools: Elements → Accessibility fül, Rendering → látáshiba-emuláció, színválasztó kontrasztmérője, Lighthouse), csak billentyűzetes bejárás (Tab, Shift+Tab, Szóköz, Enter), helyi szerver: `php -S localhost:8000`.
Kiinduló állapot: [914dd87 – az eredeti, hibás változat](https://github.com/baloogedit/umfst-bh11-tunetnaplo/commit/914dd8791a6cbae6b470db47e4500b2f955b223f) (a VS Code Prettier mentéskor újraformázta a fájlt, a tartalom változatlan).

## 1. hiba – A nagyítás le van tiltva

- **Hiba leírása és reprodukálása (milyen segédtechnológiával, milyen lépésekkel):** Chrome eszköznézetben (F12 → Ctrl+Shift+M) / telefonon két ujjal nem lehet ránagyítani az oldalra, ami a nagyítóval dolgozó, gyengén látó felhasználót kizárja. A Lighthouse figyelmeztet: „[user-scalable="no"] is used in the meta viewport element or the [maximum-scale] attribute is less than 5”.
- **Gyökérok + megsértett WCAG 2.2 sikerkritérium:** A `viewport` meta tagben a `maximum-scale=1` és a `user-scalable=no` megtiltja a böngészőnek a nagyítást. Sérül: **WCAG 1.4.4 Szöveg átméretezése (AA)**.
- **Javítás (commit link, előtte/utána kód):** [1ef7ece](https://github.com/baloogedit/umfst-bh11-tunetnaplo/commit/1ef7ece0bb2178910cb1bb65352bc9f30875c235)
  - Előtte: `<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />`
  - Utána: `<meta name="viewport" content="width=device-width, initial-scale=1" />`
- **Hogyan tesztelted:** Lighthouse Accessibility: a viewport-figyelmeztetés eltűnt. Eszköznézetben a nagyítás engedélyezett.
- **Tanulság / érintett alapelv:** A felhasználótól soha nem szabad elvenni a nagyítást. A viewport csak a kezdő méretet állítsa be.

## 2. hiba – Div-ből készült „jelölőnégyzetek”

- **Hiba leírása és reprodukálása:** Csak billentyűzettel a Fejfájás / Hányinger / Szédülés „gombokra” Tab-bal nem lehet rálépni, Szóközzel nem kapcsolhatók. A DevTools Accessibility fülön a szerepük _generic_, nincs állapotuk, ezért a képernyőolvasó sima szövegként olvassa őket, és nem mondja meg, hogy ki vannak-e választva. A kiválasztást csak a háttérszín jelzi.
- **Gyökérok + megsértett WCAG 2.2 sikerkritérium:** Sima `div` elemek, amelyeket csak egy JS kattintáskezelő (`classList.toggle('on')`) tesz „kattinthatóvá”: nem fókuszálhatók, nincs checkbox szerepük és bejelölt állapotuk. Sérül: **WCAG 2.1.1 Billentyűzet (A)**, **4.1.2 Név, szerep, érték (A)**.
- **Javítás (commit link, előtte/utána kód):** [a85c9b8](https://github.com/baloogedit/umfst-bh11-tunetnaplo/commit/a85c9b8d0c65ce56dd8e6f64a866d9891d19bbec)
  - Előtte: `<h3>Tünetek</h3>` + `<div id="tunetek"><div class="chk" data-v="fejfajas">Fejfájás</div> …</div>`, valamint a `.chk` elemekre tett `click` kezelő.
  - Utána: `<fieldset id="tunetek"><legend>Tünetek</legend><label class="chk"><input type="checkbox" name="tunet" value="fejfajas" />Fejfájás</label> …</fieldset>`
  - A „pill” kinézet CSS-ből jön a natív elemre: `.chk:has(input:checked)` színezi a bejelöltet, `.chk:has(input:focus-visible)` jól látható fókuszkeretet ad. A JS kattintáskezelő törölve, mert a natív checkbox magától kapcsol.
- **Hogyan tesztelted:** Csak billentyűzet: Tab-bal a tünetekre lépek, kék fókuszkeret látszik, Szóközre be- és kikapcsolnak. DevTools Accessibility fül: Role: _checkbox_, Name: _Fejfájás_, Checked: _true/false_. Lighthouse Accessibility.
- **Tanulság / érintett alapelv:** **Az ARIA első szabálya: natív elem előbb.** A natív checkbox ingyen adja a fókuszt, a billentyűkezelést, a szerepet és az állapotot. A kinézetet CSS-sel kell a natív elemre tenni, nem `div`-et kell checkboxnak álcázni.

## 3. hiba – Címkék helyett címsorok

- **Hiba leírása és reprodukálása:** A képernyőolvasó a mezőknél csak „szerkesztés, szám” / „szerkesztés, több soros” szöveget mond, címkét nem. A DevTools Accessibility fülön a mezők neve üres, és a feliratra kattintva a kurzor nem kerül a mezőbe. Címsoronként navigálva a `h1` után rögtön `h3` jön.
- **Gyökérok + megsértett WCAG 2.2 sikerkritérium:** Az „Erősség (0–10)” és a „Megjegyzés” felirat `h3` címsor, nem `label`, így programozottan semmi nem köti a mezőhöz, a mezőknek nincs akadálymentes nevük. A címsor-hierarchia átugorja a `h2`-t. Sérül: **WCAG 1.3.1 Információ és kapcsolatok (A)**, **3.3.2 Címkék vagy utasítások (A)**, **4.1.2 Név, szerep, érték (A)**.
- **Javítás (commit link, előtte/utána kód):** [edd0cfa](https://github.com/baloogedit/umfst-bh11-tunetnaplo/commit/edd0cfa7652ffce015319bce916417cbf071dd08)
  - Előtte: `<h3>Erősség (0–10)</h3>` és `<h3>Megjegyzés</h3>`
  - Utána: `<label for="eros">Erősség (0–10)</label>` és `<label for="megj">Megjegyzés</label>`, plusz CSS: `label[for] { display: block; font-weight: bold; margin: 16px 0 4px; }`, hogy a felirat ugyanott és ugyanúgy jelenjen meg. A „Tünetek” felirat már a 2. hibánál `legend` lett, így `h3` nem maradt az oldalon.
- **Hogyan tesztelted:** A feliratra kattintva a kurzor a mezőbe ugrik (működik a `for`–`id` kapcsolat). DevTools Accessibility fül: a mezők neve „Erősség (0–10)” és „Megjegyzés”, a névadó a `label`. Lighthouse: a „Form elements do not have associated labels” jelzés eltűnt.
- **Tanulság / érintett alapelv:** A címsor a dokumentum szerkezetét adja, a `label` a mező nevét. Ami vizuálisan összetartozik, azt kódban is össze kell kapcsolni (`for`–`id`). Címsorszintet nem szabad átugorni.

## 4. hiba – Pozitív tabindex, felborult fókuszsorrend

- **Hiba leírása és reprodukálása:** Csak billentyűzettel az oldal betöltése után az első Tab a Megjegyzés mezőre, a második az Erősségre ugrik, és csak utána jön az oldal eleje, a vizuális sorrenddel ellentétesen.
- **Gyökérok + megsértett WCAG 2.2 sikerkritérium:** `tabindex="2"` az Erősség és `tabindex="1"` a Megjegyzés mezőn. A pozitív tabindexű elemek minden más elem elé kerülnek a fókuszsorrendben, számsorrendben. Sérül: **WCAG 2.4.3 Fókuszsorrend (A)**.
- **Javítás (commit link, előtte/utána kód):** [bb340f2](https://github.com/baloogedit/umfst-bh11-tunetnaplo/commit/bb340f2b0aba2aab554f9623e2e855d39b421d6d)
  - Előtte: `<input id="eros" type="number" min="0" max="10" tabindex="2" />`, `<textarea id="megj" tabindex="1"></textarea>`
  - Utána: `<input id="eros" type="number" min="0" max="10" />`, `<textarea id="megj"></textarea>`
- **Hogyan tesztelted:** Tab-bejárás: Fejfájás → Hányinger → Szédülés → Erősség → Megjegyzés, Shift+Tab-bal pontosan fordítva. Lighthouse: a „Some elements have a [tabindex] value greater than 0” jelzés eltűnt.
- **Tanulság / érintett alapelv:** Pozitív tabindexet nem szabad használni. A fókuszsorrendet a DOM-sorrend adja, ezért a HTML sorrendje legyen a logikai sorrend. Csak a `0` és a `-1` érték megengedett.

## 5. hiba – Div-ből készült Mentés „gomb”

- **Hiba leírása és reprodukálása:** Csak billentyűzettel a Mentés feliratra Tab-bal nem lehet rálépni, Enterre (sem a gombon, sem a mezőben) nem történik semmi, csak egérrel lehet menteni. A DevTools Accessibility fülön a szerepe _generic_, nem gomb.
- **Gyökérok + megsértett WCAG 2.2 sikerkritérium:** A „gomb” egy `div class="btn"` egy `click` kezelővel, `form` nélkül: nem fókuszálható, nincs gomb szerepe, nem reagál az Enterre és a Szóközre. Sérül: **WCAG 2.1.1 Billentyűzet (A)**, **4.1.2 Név, szerep, érték (A)**.
- **Javítás (commit link, előtte/utána kód):** [ab6d07c](https://github.com/baloogedit/umfst-bh11-tunetnaplo/commit/ab6d07ca40653419e9c22e8bfc9b81dae2129339)
  - Előtte: `<div class="btn" id="ment">Mentés</div>` és `document.getElementById('ment').addEventListener('click', function () { … })`
  - Utána: az egész űrlap egy `<form id="naplo">`-ban, `<button type="submit" class="btn" id="ment">Mentés</button>`, és `document.getElementById('naplo').addEventListener('submit', function (e) { e.preventDefault(); … })`. CSS: a `.btn` kapott `border: 0; font: inherit;` szabályt és egy `.btn:focus-visible` fókuszkeretet.
- **Hogyan tesztelted:** Tab-bal a gombra lépek (látható fókuszkeret), Enterre és Szóközre megjelenik a „Mentve!”. Az Erősség mezőben nyomott Enter is ment. Az oldal nem töltődik újra. DevTools Accessibility fül: Role: _button_, Name: _Mentés_.
- **Tanulság / érintett alapelv:** Az ARIA első szabálya, natív elem előbb: a `button` ingyen adja a fókuszt, az Enter/Szóköz kezelést és a szerepet. Űrlapnál a `form` `submit` eseményét kell figyelni (`preventDefault`-tal), nem a gomb kattintását, így minden beküldési mód működik.

## 6. hiba – A hibát csak a piros keret jelzi

- **Hiba leírása és reprodukálása:** Üres vagy 0–10 közé nem eső erősségnél csak a mező kerete lesz piros. Szürkeárnyalatos emulációban (DevTools → Rendering → Emulate vision deficiencies → Achromatopsia) a hiba nem vehető észre, a képernyőolvasó semmit nem mond, és sehol nincs leírva, mi a hiba.
- **Gyökérok + megsértett WCAG 2.2 sikerkritérium:** A hibát a JS csak a `hibas` CSS osztállyal (piros keret) jelzi: nincs szöveges hibaüzenet, nincs `aria-invalid` állapot, és a fókusz sem kerül a hibás mezőre. Sérül: **WCAG 1.4.1 Szín használata (A)**, **3.3.1 Hiba azonosítása (A)**.
- **Javítás (commit link, előtte/utána kód):** [3ef6ebb](https://github.com/baloogedit/umfst-bh11-tunetnaplo/commit/3ef6ebba5a6951a5f4a73fd5e9ea86bbb861c651)
  - Előtte: `if (eros.value === '' || v < 0 || v > 10) { eros.classList.add('hibas'); return; }`
  - Utána: `<form id="naplo" novalidate>`, `<input id="eros" … aria-describedby="eros-hiba" />` + `<p id="eros-hiba" class="hibauzenet" hidden></p>`. Hiba esetén a JS beállítja az `aria-invalid="true"`-t, kiírja a „⚠ Hiba: Az erősség 0 és 10 közötti szám kell legyen” szöveget, megjeleníti, és a fókuszt a mezőre teszi. Helyes értéknél mindezt visszaállítja. CSS: `.hibauzenet { color: #b3261e; font-weight: bold; }` (6,5:1 kontraszt).
- **Hogyan tesztelted:** Üres és 11-es értékkel beküldve megjelenik a szöveges hibaüzenet, és a fókusz az Erősség mezőre ugrik. DevTools Accessibility fül: _Invalid user entry: true_, a Description a hibaüzenet szövege. Szürkeárnyalatos emulációban a hiba a szöveg miatt egyértelmű. 5-ös értékkel a hiba eltűnik.
- **Tanulság / érintett alapelv:** **Ne csak színnel közölj információt.** A hibát szövegesen kell kiírni, programozottan a mezőhöz kötni (`aria-describedby`, `aria-invalid`), és a fókusszal oda kell vinni a felhasználót.

## 7. hiba – A „Mentve!” üzenet nincs felolvasva, és alig olvasható

- **Hiba leírása és reprodukálása:** Sikeres mentés után a képernyőolvasó nem olvassa fel a „Mentve!” szöveget, a vak felhasználó nem tudja, sikerült-e. A DevTools színválasztójának kontrasztmérője a `#2ecc71` szövegre fehér háttéren 2,1:1-et mutat.
- **Gyökérok + megsértett WCAG 2.2 sikerkritérium:** A `p id="ok"` nem élő régió, így a tartalma változását a képernyőolvasó nem jelzi. A `#2ecc71` kontrasztja fehéren 2,1:1, normál szövegnél legalább 4,5:1 kellene. Sérül: **WCAG 4.1.3 Állapotüzenetek (AA)**, **1.4.3 Kontraszt (minimum) (AA)**.
- **Javítás (commit link, előtte/utána kód):** [1b66b4c](https://github.com/baloogedit/umfst-bh11-tunetnaplo/commit/1b66b4cd7e36d36ad7f03f1779e691628a90efcd)
  - Előtte: `<p id="ok"></p>`, `#ok { color: #2ecc71; }`, `textContent = 'Mentve!'`
  - Utána: `<p id="ok" role="status"></p>`, `#ok { color: #085623; }` (8,9:1 kontraszt), `textContent = '✓ Mentve!'` (a sikert az ikon is jelzi, nem csak a szín).
- **Hogyan tesztelted:** DevTools kontrasztmérő: 8,9:1, mindkét pipa zöld (AA és AAA). DevTools Accessibility fül: Role: _status_, Live region: _polite_. Lighthouse: a kontraszt-figyelmeztetés eltűnt.
- **Tanulság / érintett alapelv:** A dinamikusan megjelenő állapotüzenetet élő régióba kell tenni (`role="status"`), különben a képernyőolvasó nem tud róla. A szövegszínt mindig meg kell mérni, és a „zöld = siker” önmagában se nem elég kontrasztos, se nem elég információ (Ne csak színnel közölj információt).
