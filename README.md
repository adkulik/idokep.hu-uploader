# ⛅ Időkép Automata Feltöltő (Home Assistant Blueprint)

Ezzel a Home Assistant sablonnal (Blueprint) pofonegyszerűen, egy grafikus felületen keresztül bekötheted az okosotthonod meteorológiai szenzorait a magyar Időkép rendszerébe. 

Nincs többé bonyolult YAML kódolás, a sablon a háttérben automatikusan elvégzi a legnehezebb feladatokat:
* Az adatok Időkép-kompatibilis formázását (tizedesvesszők, pontos kerekítések).
* A szélsebesség és széllökés automatikus átváltását.
* A nem elérhető (unavailable/unknown) vagy épp frissítő szenzorok intelligens elrejtését, így a feltöltés sosem akad meg hamis nullákkal.

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

A feltöltéshez szükséged lesz egy saját, haladó szintű Időkép fiókra. Ha még nincs ilyened, **[kattints ide és regisztrálj egyet!](https://www.idokep.hu/regisztracio?szint=halado)**

Mivel a Blueprintek önmagukban nem tudnak közvetlenül a külső hálózatra kommunikálni, létre kell hoznunk egy apró, univerzális "hidat" a Home Assistant `configuration.yaml` fájljában.

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

> **Biztonsági megjegyzés:** a jelszó az URL-ben utazik, és ha a beküldés hibára fut, a Home Assistant a teljes URL-t, benne az állomás jelszavával, kiírhatja a rendszernaplóba. Ez bekerülhet a diagnosztikai csomagokba és a biztonsági mentésekbe is, ezért az Időkép-állomásodhoz használj külön, máshol nem használt jelszót.

4. Mentsd el a fájlt, majd **indítsd újra a Home Assistantot**.

---

## 🚀 2. Lépés: Telepítés

A Blueprint importálásához kattints az alábbi gombra, ami varázsütésre megnyitja a saját Home Assistant rendszeredet az importáló ablakkal:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/adkulik/idokep.hu-uploader/refs/heads/main/idokep.hu-uploader.yaml)

*(Ha a gomb valamiért nem működne, másold ki ezt a linket: `https://raw.githubusercontent.com/adkulik/idokep.hu-uploader/refs/heads/main/idokep.hu-uploader.yaml`, majd a Home Assistant **Sablon-automatizmusok** oldalán kattints a jobb alsó sarokban lévő **Sablon-automatizmus importálása** gombra. Illeszd be a linket, kattints az **Előnézet**, végül a **Sablon-automatizmus importálása** gombra!)*

---

## ⚙️ 3. Lépés: Beállítás a felületen

1. Menj a **Beállítások -> Automatizmusok és jelenetek -> Sablon-automatizmusok** oldalra (angol felületen: *Blueprints*).
2. Keresd meg a listában az *Időkép Automata Feltöltő (Univerzális)* sablont, és kattints az **Automatizmus létrehozása** gombra.
3. Töltsd ki a megjelenő egyszerű grafikus felületet:
   * **Felhasználónév és Jelszó:** Add meg az Időkép állomásod hivatalos azonosítóit.
   * **Ütemezés:** Válaszd ki, milyen sűrűn küldjön adatot (percenként, 2, 5, 10 vagy 15 percenként). Az alapértelmezés 5 perc, ami a legtöbb állomásnak bőven elég.
   * **Szenzorok:** Válaszd ki a legördülő menükből az okosotthonod megfelelő entitásait. Csak a Hőmérséklet megadása kötelező, az összes többi meteorológiai adat (páratartalom, légnyomás, beltéri hőmérséklet, szél, csapadék) opcionális!
4. Mentsd el az automatizmust.

Kész is vagy! Az automatizmus innentől kezdve a háttérben, észrevétlenül, a választott gyakorisággal frissíti az adataidat az Időkép hálózatában.

### 🌬️ Mértékegységek

* **Szélsebesség és széllökés:** a sablon a szenzor saját mértékegységét olvassa ki, és automatikusan m/s-ra váltja (km/h, m/s, mph és csomó támogatott).
* **Szélirány:** megadható fokban (0–360) vagy égtáj-rövidítésként, angolul (N, NNE, NE …) vagy magyarul (É, ÉK, K, DK, D, DNY, NY, ÉNY).

---

## 🔄 Frissítés

Új verzió megjelenésekor a Home Assistantban a **Sablon-automatizmusok** listájában kattints a sablon melletti három pöttyre, és válaszd a **Sablon-automatizmus újraimportálása** lehetőséget (angol felületen: *Re-import blueprint*). A meglévő automatizmusod beállításai megmaradnak. Hogy épp melyik verzió fut nálad, azt a sablon leírásának alján látod.

### Frissítés 1.x-ről 2.0-ra

A 2.0 a `shell_command` és a curl helyett a Home Assistant beépített `rest_command` integrációját használja, ezért egyszer a `configuration.yaml`-hoz is hozzá kell nyúlnod. Ebben a sorrendben haladj, így a feltöltés legfeljebb az újraindítások idejére áll le:

1. Vedd fel a `configuration.yaml`-ba a fenti, 1. lépésben látható `rest_command` blokkot. A régi `shell_command.idokep_get` sort egyelőre hagyd meg.
2. Indítsd újra a Home Assistantot.
3. Importáld újra a sablont a fent leírt módon.
4. Töröld a `configuration.yaml`-ból a régi `shell_command.idokep_get` sort, majd indítsd újra a Home Assistantot, vagy töltsd újra a shell parancsokat.

Az automatizmusod beállításait nem kell módosítani.

Ha maradnál a curl-os változatnál, az 1.0 továbbra is importálható erről a címről: `https://raw.githubusercontent.com/adkulik/idokep.hu-uploader/refs/tags/v1.0/idokep.hu-uploader.yaml`

---

## 📋 Változásnapló

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
