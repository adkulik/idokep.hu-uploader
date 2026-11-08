# ⛅ Időkép Automata Feltöltő (Home Assistant sablon-automatizmus / Blueprint)

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20me%20a%20coffee-FFDD00?logo=buymeacoffee&logoColor=black)](https://buymeacoffee.com/adkulik)

Ezzel a Home Assistant sablon-automatizmussal (angolul: Blueprint) pofonegyszerűen, egy grafikus felületen keresztül bekötheted az okosotthonod meteorológiai szenzorait a magyar Időkép rendszerébe. 

Nincs többé bonyolult YAML kódolás, a sablon a háttérben automatikusan elvégzi a legnehezebb feladatokat:
* Az adatok Időkép-kompatibilis formázását (tizedesvesszők, pontos kerekítések).
* A szélsebesség és széllökés automatikus átváltását.
* A nem elérhető (unavailable/unknown) vagy épp frissítő szenzorok intelligens elrejtését, így a feltöltés sosem akad meg hamis nullákkal.

---

## 🔔 Ajánlott: értesítés az új verziókról (Blueprints Updater)

A Home Assistant magától nem szól, ha egy sablon-automatizmusnak új verziója jelenik meg, ilyenkor kézzel kellene újraimportálni. Ezt veszi át a [Blueprints Updater](https://github.com/luuquangvu/blueprints-updater) közösségi integráció:

* Minden telepített sablon-automatizmushoz frissítés-entitást hoz létre. Az Időkép Automata Feltöltő új verziója így ugyanott jelenik meg, ahol a Home Assistant többi frissítése, és egy kattintással telepíthető.
* A sablon erre a repóra hivatkozik, ezért az integráció külön beállítás nélkül követi.
* Frissítéskor az automatizmusod beállításai megmaradnak, a sablon korábbi változatairól pedig mentést tart.

**Telepítés:** Home Assistant 2026.11-től a **Beállítások → Marketplace** oldalon (korábbi verziókban a HACS-ban) keresd meg a *Blueprints Updater*-t, telepítsd, majd indítsd újra a Home Assistantot. Ezután a **Beállítások → Eszközök és szolgáltatások → Integráció hozzáadása** gombbal add hozzá.

**Tipp:** az automatikus frissítést hagyd kikapcsolva, és frissítés előtt nézd meg lent a Változásnaplót. Egy-egy új verzió (például a 2.0) a `configuration.yaml` módosítását is kérheti, ezt a frissítés nem végzi el helyetted.

---

## ❗ Fontos figyelmeztetés: Csak hiteles adatokat tölts fel!

Az Időkép térképeit és előrejelzéseit emberek százezrei használják nap mint nap. Kérjük, **csak és kizárólag akkor indíts adatfeltöltést, ha a szenzoraid elhelyezése megfelel a meteorológiai szabványoknak!** A rossz helyre tett műszerek (pl. napon lévő hőmérő, fal melletti szélmérő) hibás adatokat generálnak, amivel rontod a hálózat megbízhatóságát.

### 📐 Műszerelhelyezési útmutató
* **Hőmérő és páratartalom mérő:** 2 méter magasságban, árnyékos helyen (lehetőleg szabványos hőmérőházban vagy tányéros árnyékolóban) kell elhelyezni. Szigorúan tilos falhoz, betonhoz vagy olyan felülethez rögzíteni, ami hőt sugározhat magából!
* **Szélmérő:** Szabad, nyílt területen, a talajszinttől számítva ideálisan 10 méter magasan kell lennie. Ha erre nincs mód, akkor a környező akadályok (háztető, fák) fölé kell nyúlnia legalább 2-3 méterrel, hogy a légáramlás akadálytalanul érje.
* **Csapadékmérő:** Nyílt, vízszintes területen, a talajtól nagyjából 1-1,5 méter magasságban telepítve. Ügyelj rá, hogy épület, fa vagy tető ne takarja ki, és a szél ne tudjon belefújni faleveleket.
* **Légnyomásmérő:** Beltéren is elhelyezhető, de figyelj arra, hogy a Home Assistant mindig **tengerszintre átszámított (relatív) légnyomásértéket** küldjön, ne pedig a szenzor által mért abszolút értéket.

---

## ⚠️ 1. Lépés: Előfeltételek (Kötelező!)

A sablon **Home Assistant 2024.10 vagy újabb** verziót igényel.

A feltöltéshez szükséged lesz egy saját, haladó szintű Időkép fiókra. Ha még nincs ilyened, **[kattints ide és regisztrálj egyet!](https://www.idokep.hu/regisztracio?szint=halado)** Az így létrehozott fiók Normál típusú (lásd a 3. lépésben a Fióktípust).

Mivel a sablon-automatizmusok önmagukban nem tudnak közvetlenül a külső hálózatra kommunikálni, létre kell hoznunk egy apró, univerzális "hidat" a Home Assistant `configuration.yaml` fájljában.

1. Nyisd meg a Home Assistant `configuration.yaml` fájlját.
2. Keresd meg a `rest_command:` blokkot (ha még nincs, hozd létre a fájl végén).
3. Illeszd be az alábbi bejegyzést pontosan így, a behúzásokra figyelve:

```yaml
rest_command:
  idokep_get:
    url: "{{ url }}"
    method: get
    headers:
      User-Agent: "IdokepAutomata/500 (Windows)"
    timeout: 15
```

> **Mit csinál ez a blokk?** Létrehoz egy `rest_command.idokep_get` műveletet, amellyel a sablon a Home Assistant beépített HTTP-kliensén keresztül küldi el az adatokat, külön program futtatása nélkül. A `User-Agent` fejléc igazolja, hogy Időkép Feltöltőt használsz.

> **Hibakezelés:** ha az Időkép hibakóddal (4xx/5xx) válaszol, vagy nem válaszol időben, a Home Assistant ezt a rendszernaplóba írja. A válasz szövegét a sablon nem ellenőrzi.

> **Biztonsági megjegyzés:** a beküldés titkosított HTTPS-kapcsolaton megy, de a jelszó az URL része, és ha a beküldés hibára fut, a Home Assistant a teljes URL-t, benne az állomás jelszavával, kiírhatja a rendszernaplóba. Ez bekerülhet a diagnosztikai csomagokba és a biztonsági mentésekbe is, ezért az Időkép-állomásodhoz használj külön, máshol nem használt jelszót.

4. Mentsd el a fájlt, majd **indítsd újra a Home Assistantot**.

---

## 🚀 2. Lépés: Telepítés

A sablon-automatizmus importálásához kattints az alábbi gombra, ami varázsütésre megnyitja a saját Home Assistant rendszeredet az importáló ablakkal:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/adkulik/idokep.hu-uploader/refs/heads/main/idokep.hu-uploader.yaml)

*(Ha a gomb valamiért nem működne, másold ki ezt a linket: `https://raw.githubusercontent.com/adkulik/idokep.hu-uploader/refs/heads/main/idokep.hu-uploader.yaml`, majd a Home Assistant **Sablon-automatizmusok** oldalán kattints a jobb alsó sarokban lévő **Sablon-automatizmus importálása** gombra. Illeszd be a linket, kattints az **Előnézet**, végül a **Sablon-automatizmus importálása** gombra!)*

---

## ⚙️ 3. Lépés: Beállítás a felületen

1. Menj a **Beállítások -> Automatizmusok és jelenetek -> Sablon-automatizmusok** oldalra (angol felületen: *Blueprints*).
2. Keresd meg a listában az *Időkép Automata Feltöltő (Univerzális)* sablont, és kattints az **Automatizmus létrehozása** gombra.
3. Töltsd ki a megjelenő egyszerű grafikus felületet:
   * **Fióktípus:** Az Időképen kétféle fiók létezik, Normál és Pro. A legtöbb állomáshoz a Normál fiók megfelelő, Pro fiókra csak speciális esetekben van szükség. Ha a fiókodat az 1. lépésben linkelt haladó szintű regisztrációval hoztad létre, akkor Normál fiókod van. Az alapértelmezés a Normál.
   * **Felhasználónév és Jelszó:** Add meg az Időkép állomásod hivatalos azonosítóit.
   * **Ütemezés:** Válaszd ki, milyen sűrűn küldjön adatot (percenként, 2, 5, 10 vagy 15 percenként). Az alapértelmezés 5 perc, ami a legtöbb állomásnak bőven elég.
   * **Szenzorok:** Válaszd ki a legördülő menükből az okosotthonod megfelelő entitásait. Csak a Hőmérséklet megadása kötelező, az összes többi meteorológiai adat (páratartalom, légnyomás, beltéri hőmérséklet, szél, csapadék, napsugárzás, UV-index) opcionális!
4. Mentsd el az automatizmust.

Kész is vagy! Az automatizmus innentől kezdve a háttérben, észrevétlenül, a választott gyakorisággal frissíti az adataidat az Időkép hálózatában.

### 🌬️ Mértékegységek

* **Szélsebesség és széllökés:** a sablon a szenzor saját mértékegységét olvassa ki, és automatikusan m/s-ra váltja (km/h, m/s, mph és csomó támogatott).
* **Szélirány:** megadható fokban (0–360) vagy égtáj-rövidítésként, angolul (N, NNE, NE …) vagy magyarul (É, ÉK, K, DK, D, DNY, NY, ÉNY).
* **Csapadék:** mm-ben megy fel. Az 1 órás (`csap1h`) és a 24 órás (`csap`) érték gördülő összeg, az összes csapadék (`csaptotal`) a csapadékmérő folyamatosan növekvő számlálója.
* **Napsugárzás:** W/m²-ben megy fel (`napfeny`). Ha a szenzor luxban mér, a sablon közelítőleg W/m²-re váltja.
* **UV-index:** mértékegység nélküli szám, átváltás nélkül megy fel (`uv`).

---

## 🔄 Frissítés

Új verzió megjelenésekor a Home Assistantban a **Sablon-automatizmusok** listájában kattints a sablon melletti három pöttyre, és válaszd a **Sablon-automatizmus újraimportálása** lehetőséget (angol felületen: *Re-import blueprint*). A meglévő automatizmusod beállításai megmaradnak. Hogy épp melyik verzió fut nálad, azt a sablon leírásának alján látod.

### Frissítés 2.3-ról 2.5-re

Importáld újra a sablont a fent leírt módon, majd nyisd meg az automatizmusodat, és a **Csapadék** részben a mezők leírása alapján ellenőrizd a csapadékszenzorokat.

### Frissítés 2.2-ről 2.3-ra

Importáld újra a sablont a fent leírt módon. Sem a `configuration.yaml`-hoz, sem az automatizmusod beállításaihoz nem kell hozzányúlni.

### Frissítés 2.1-ről 2.2-re

Importáld újra a sablont a fent leírt módon. A `configuration.yaml`-hoz nem kell hozzányúlni. A napsugárzás és az UV-index beküldéséhez nyisd meg az automatizmusodat, és a **Napsugárzás és UV** részben válaszd ki a szenzoraidat. Ha üresen hagyod őket, minden úgy működik, mint eddig.

### Frissítés 2.0-ról 2.1-re

Importáld újra a sablont a fent leírt módon. A `configuration.yaml`-hoz nem kell hozzányúlni.

A 2.0 és a korábbi verziók mindig a Pro fiókok címére (`pro.idokep.hu`) küldtek, a 2.1 alapértelmezése viszont a Normál fiók. Ha Pro fiókod van, az újraimportálás után nyisd meg az automatizmusodat, és az **Időkép fiók** részben a **Fióktípusnál** válaszd a **Pro**-t. Normál fiók esetén nincs teendőd.

### Frissítés 1.x-ről 2.0-ra

A 2.0 a `shell_command` és a curl helyett a Home Assistant beépített `rest_command` integrációját használja, ezért egyszer a `configuration.yaml`-hoz is hozzá kell nyúlnod. Ebben a sorrendben haladj, így a feltöltés legfeljebb az újraindítások idejére áll le:

1. Vedd fel a `configuration.yaml`-ba a fenti, 1. lépésben látható `rest_command` blokkot. A régi `shell_command.idokep_get` sort egyelőre hagyd meg.
2. Indítsd újra a Home Assistantot.
3. Importáld újra a sablont a fent leírt módon.
4. Töröld a `configuration.yaml`-ból a régi `shell_command.idokep_get` sort, majd indítsd újra a Home Assistantot, vagy töltsd újra a shell parancsokat.

Az automatizmusod beállításait nem kell módosítani.

Ha maradnál a curl-os változatnál, az 1.0 továbbra is importálható erről a címről: `https://raw.githubusercontent.com/adkulik/idokep.hu-uploader/refs/tags/v1.0/idokep.hu-uploader.yaml`

---

## ☕ Támogatás

A sablon ingyenes, és az is marad. Ha hasznosnak találod, egy kávéval támogathatod a további fejlesztést:

<a href="https://buymeacoffee.com/adkulik"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me a Coffee" height="50"></a>

Rendszeres támogatásra a [GitHub Sponsors](https://github.com/sponsors/adkulik) is elérhető.

---

## 📋 Változásnapló

### 2.5 (2026-11-08)

* Csapadékfeltöltés, UV- és fényadatok optimalizálása.

### 2.3 (2026-11-07)

* A beküldés titkosított HTTPS-kapcsolaton megy (`https://idokep.hu`, illetve `https://pro.idokep.hu`), így a jelszó és a mérési adatok nem utaznak titkosítatlanul a hálózaton.
* A frissítéshez elég újraimportálni a sablont: a `configuration.yaml` és az automatizmus beállításai változatlanok.

### 2.2 (2026-11-07)

* Új, opcionális **Napsugárzás és UV** szekció: napsugárzás (`solarradiation`, W/m²) és UV-index (`UV`) beküldése.
* Lux mértékegységű fényerő-szenzor értékét a sablon közelítőleg W/m²-re váltja.
* Ha a két új mezőt üresen hagyod, a beküldés változatlan.

### 2.1 (2026-11-01)

* Új beállítás az **Időkép fiók** szekcióban: **Fióktípus** (Normál vagy Pro). Normál fióknál az adatok az `idokep.hu`, Pro fióknál a `pro.idokep.hu` címre mennek. Az alapértelmezés a Normál.
* A 2.0 és a korábbi verziók mindig a `pro.idokep.hu` címre küldtek. Ha Pro fiókod van, a frissítés után a Fióktípusnál válaszd a Pro-t. Lásd fent a *Frissítés 2.0-ról 2.1-re* részt.
* A beküldés típusjelölése (`tipus`) `egyeb` helyett `HA_Uploader`.

### 2.0 (2026-10-31)

**Törő változás:** a frissítéshez a `configuration.yaml`-ba új bejegyzés kell. Lásd fent a *Frissítés 1.x-ről 2.0-ra* részt.

* A beküldés a Home Assistant beépített `rest_command` integrációjával történik a `shell_command` és a curl helyett. Nem indul külön folyamat minden beküldésnél, és megszűnt a `shell_command` átirányításos csapdája.
* A 4xx/5xx HTTP-hibákat a Home Assistant figyelmeztetésként naplózza. Az 1.0 curl-os beállítása ezeket csendben elnyelte.
* Ismert korlát: az Időkép válasza nem UTF-8 kódolású, és a `rest_command` ezt nem tudja szövegként beolvasni. Ezért a sablon a válasz szövegét nem ellenőrzi, így például a hibás jelszót sem ismeri fel.

### 1.0 (2026-10-31)

Első számozott kiadás.

* Grafikus beállítófelület öt összecsukható szekcióval: Időkép fiók, Ütemezés, Hőmérséklet és Páratartalom, Szél, Csapadék.
* Választható feltöltési gyakoriság: percenként, 2, 5, 10 vagy 15 percenként (alapértelmezés: 5 perc).
* Beküldhető adatok: kültéri hőmérséklet, páratartalom, tengerszintre korrigált légnyomás, beltéri hőmérséklet, szélirány, szélsebesség, széllökés, valamint 10 perces, 1 órás és napi csapadék.
* A szélsebesség és a széllökés automatikus átváltása m/s-ra a szenzor mértékegysége alapján (km/h, m/s, mph, csomó).
* Szélirány fokban vagy angol, illetve magyar égtáj-rövidítéssel.
* A nem elérhető vagy nem számértékű szenzorok kimaradnak a beküldésből, így nem kerülnek fel hamis nullák.
* Kitöltetlen fiókadatok vagy hőmérséklet-szenzor esetén nem indul feltöltés.
* Műszerelhelyezési útmutató és biztonsági tudnivalók közvetlenül a beállítófelületen.
