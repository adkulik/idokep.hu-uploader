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

A feltöltéshez szükséged lesz egy saját, haladó szintű Időkép fiókra. Ha még nincs ilyened, **[kattints ide és regisztrálj egyet!](https://www.idokep.hu/regisztracio?szint=halado)**

Mivel a Blueprintek önmagukban nem tudnak közvetlenül a külső hálózatra kommunikálni, létre kell hoznunk egy apró, univerzális "hidat" a Home Assistant `configuration.yaml` fájljában.

1. Nyisd meg a Home Assistant `configuration.yaml` fájlját.
2. Keresd meg a `shell_command:` blokkot (ha még nincs, hozd létre a fájl végén).
3. Illeszd be az alábbi sort pontosan így, az idézőjelekre figyelve:

```yaml
shell_command:
  idokep_get: 'curl -s -o /dev/null -A "IdokepAutomata/500 (Windows)" "{{ url }}"'
```

> **Miért van erre szükség?** Az `-A` kapcsolóval megadott azonosító elhiteti az Időkép szervereivel, hogy egy hivatalos meteorológiai szoftverből érkezik az adat, így elkerülheted a kérések blokkolását. Az `-o /dev/null` eldobja a szerver válaszát, hogy ne kerüljön feleslegesen a Home Assistantba.

> **Ne használj `> /dev/null 2>&1` átirányítást!** A Home Assistant a `shell_command`-ot nem shellben futtatja, így az átirányítást a curl külön argumentumként kapná meg, és `URL_MALFORMAT` (3-as) hibakóddal leállna. Ezért szerepel fent a curl saját `-o /dev/null` kapcsolója.

> **Biztonsági megjegyzés:** ha a parancs hibára fut, a Home Assistant a teljes parancssort, benne az állomás jelszavával, kiírja a rendszernaplóba. Ez bekerülhet a diagnosztikai csomagokba és a biztonsági mentésekbe is, ezért az Időkép-állomásodhoz használj külön, máshol nem használt jelszót.

4. Mentsd el a fájlt, majd **indítsd újra a Home Assistantot**, vagy a *Fejlesztői eszközök* menüpontban végezz egy YAML újratöltést!

---

## 🚀 2. Lépés: Telepítés

A Blueprint importálásához kattints az alábbi gombra, ami varázsütésre megnyitja a saját Home Assistant rendszeredet az importáló ablakkal:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/adkulik/idokep.hu-uploader/refs/heads/main/idokep.hu-uploader.yaml)

*(Ha a gomb valamiért nem működne, másold ki ezt a linket: `https://raw.githubusercontent.com/adkulik/idokep.hu-uploader/refs/heads/main/idokep.hu-uploader.yaml` és a Home Assistant **Tervezetek** menüpontjában, a jobb alsó sarokban lévő **Importálás** gombnál illeszd be!)*

---

## ⚙️ 3. Lépés: Beállítás a felületen

1. Menj a **Beállítások -> Automatizációk és jelenetek -> Tervezetek (Blueprints)** menüpontba.
2. Keresd meg a listában az *Időkép Automata Feltöltő (Univerzális)* sablont, és kattints az **Automatizálás létrehozása** gombra.
3. Töltsd ki a megjelenő egyszerű grafikus felületet:
   * **Felhasználónév és Jelszó:** Add meg az Időkép állomásod hivatalos azonosítóit.
   * **Ütemezés:** Válaszd ki, milyen sűrűn küldjön adatot (percenként, 2, 5, 10 vagy 15 percenként). Az alapértelmezés 5 perc, ami a legtöbb állomásnak bőven elég.
   * **Szenzorok:** Válaszd ki a legördülő menükből az okosotthonod megfelelő entitásait. Csak a Hőmérséklet megadása kötelező, az összes többi meteorológiai adat (páratartalom, légnyomás, beltéri hőmérséklet, szél, csapadék) opcionális!
4. Mentsd el az automatizációt.

Kész is vagy! Az automatizáció innentől kezdve a háttérben, észrevétlenül, a választott gyakorisággal frissíti az adataidat az Időkép hálózatában.

### 🌬️ Mértékegységek

* **Szélsebesség és széllökés:** a sablon a szenzor saját mértékegységét olvassa ki, és automatikusan m/s-ra váltja (km/h, m/s, mph és csomó támogatott).
* **Szélirány:** megadható fokban (0–360) vagy égtáj-rövidítésként, angolul (N, NNE, NE …) vagy magyarul (É, ÉK, K, DK, D, DNY, NY, ÉNY).
