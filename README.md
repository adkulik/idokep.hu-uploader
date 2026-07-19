# ⛅ Időkép Automata Feltöltő (Home Assistant Blueprint)

Ezzel a Home Assistant sablonnal (Blueprint) pofonegyszerűen, egy grafikus felületen keresztül bekötheted az okosotthonod meteorológiai szenzorait a magyar Időkép rendszerébe. 

Nincs többé bonyolult YAML kódolás, a sablon a háttérben automatikusan elvégzi a legnehezebb feladatokat:
* Az adatok Időkép-kompatibilis formázását (tizedesvesszők, pontos kerekítések).
* A szélsebesség és széllökés automatikus átváltását.
* A nem elérhető (unavailable/unknown) vagy épp frissítő szenzorok intelligens elrejtését, így a feltöltés sosem akad meg hamis nullákkal.

---

## ⚠️ 1. Lépés: Előfeltételek (Kötelező!)

Mivel a Blueprintek önmagukban nem tudnak közvetlenül a külső hálózatra kommunikálni, létre kell hoznunk egy apró, univerzális "hidat" a Home Assistant `configuration.yaml` fájljában.

1. Nyisd meg a Home Assistant `configuration.yaml` fájlját.
2. Keresd meg a `shell_command:` blokkot (ha még nincs, hozd létre a fájl végén).
3. Illeszd be az alábbi sort pontosan így, az idézőjelekre figyelve:

```yaml
shell_command:
  idokep_get: 'curl -s -A "IdokepAutomata/500 (Windows)" -X GET "{{ url }}"'
```

> **Miért van erre szükség?** Ez a biztonságos hivatkozás elhiteti az Időkép szervereivel, hogy egy hivatalos meteorológiai szoftverből érkezik az adat, így elkerülheted a kérések blokkolását.

4. Mentsd el a fájlt, majd **indítsd újra a Home Assistantot**, vagy a *Fejlesztői eszközök* menüpontban végezz egy YAML újratöltést!

---

## 🚀 2. Lépés: Telepítés

A Blueprint importálásához kattints az alábbi gombra, ami varázsütésre megnyitja a saját Home Assistant rendszeredet az importáló ablakkal:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/adam-kulik/idokep.hu-uploader/refs/heads/main/blueprint)

*(Ha a gomb valamiért nem működne, másold ki ezt a linket: `https://raw.githubusercontent.com/adam-kulik/idokep.hu-uploader/refs/heads/main/blueprint` és a Home Assistant **Tervezetek** menüpontjában, a jobb alsó sarokban lévő **Importálás** gombnál illeszd be!)*

---

## ⚙️ 3. Lépés: Beállítás a felületen

1. Menj a **Beállítások -> Automatizációk és jelenetek -> Tervezetek (Blueprints)** menüpontba.
2. Keresd meg a listában az *Időkép Automata Feltöltő (Univerzális)* sablont, és kattints az **Automatizálás létrehozása** gombra.
3. Töltsd ki a megjelenő egyszerű grafikus felületet:
   * **Felhasználónév és Jelszó:** Add meg az Időkép állomásod hivatalos azonosítóit.
   * **Szenzorok:** Válaszd ki a legördülő menükből az okosotthonod megfelelő entitásait. Csak a Hőmérséklet megadása kötelező, az összes többi meteorológiai adat opcionális!
4. Mentsd el az automatizációt.

Kész is vagy! Az automatizáció innentől kezdve a háttérben, észrevétlenül, **5 percenként** frissíti az adataidat az Időkép hálózatában.
