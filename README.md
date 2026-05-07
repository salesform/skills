# SalesForm Payment Integration Skill

Agenteknek készült fejlesztői skill, amely megmutatja, hogyan lehet egy külső szoftverben a SalesFormot fizetési, előfizetés-kezelési és számlázási rétegként használni.

A skill célja, hogy egy fejlesztő agent biztonságosan tudjon SalesForm-integrációt készíteni anélkül, hogy tévesen API-alapú checkout sessiont, dinamikus fizetési link generálást vagy nem létező SalesForm API működést feltételezne.

---

## Mire való?

Ezzel a skillel egy agent meg tudja érteni, hogy a SalesForm-integráció alapja:

```text
Fix SalesForm terméklink
  + URL paraméterek
  + SalesForm checkout
  + webhook
  = külső szoftverben fizetési / előfizetési státusz frissítés
```

A külső szoftver nem hoz létre SalesForm rendelést API-n keresztül. Ehelyett egy előre létrehozott SalesForm termék URL-t használ, amit paraméterezve nyit meg a vásárló számára.

---

## Mit tud a skill?

* SalesForm fix terméklinkes fizetési logika magyarázata
* URL paraméterek használata űrlap előtöltéshez
* `custom` és `custom2` mezők használata belső azonosítók átadására
* `sub=1` automatikus űrlapbeküldés szabályai
* sikeres és sikertelen vásárlás utáni redirect működése
* webhook payload fogadása és feldolgozása
* `status=true`, `status=false`, `status=cancel` kezelése
* előfizetéshez szükséges `trid`, `token`, `recdate` mentése
* upgrade / downgrade URL logika
* bankkártya csere és lemondási linkek kezelése
* agent-biztos fejlesztési szabályok

---

## Mit nem feltételezhet az agent?

A skill kifejezetten rögzíti, hogy az agent ne feltételezze ezeket:

```text
- nincs SalesForm checkout session API
- nincs API-val generált fizetési link
- nincs API-val létrehozott SalesForm rendelés
- a success redirect nem fizetési bizonyíték
- a SalesForm domain nem fix
- a webhook opcionális mezői nem mindig léteznek
```

---

## Alap integrációs modell

```text
1. A SalesForm felhasználó létrehoz egy terméket a SalesFormban.
2. A termékhez tartozik egy fix SalesForm URL.
3. A külső szoftver fizetésindításkor ezt az URL-t paraméterezi.
4. A vásárló a SalesForm checkout oldalra kerül.
5. A SalesForm kezeli a fizetést, számlázást, előfizetést.
6. Sikeres / sikertelen fizetés vagy lemondás után webhook érkezik.
7. A külső szoftver webhook alapján módosítja a belső jogosultságot vagy státuszt.
```

---

## Fontos SalesForm URL paraméterek

```text
name              Megrendelő neve
telephone         Telefonszám
email             Email cím
invoice_name      Számlázási név
invoice_vatnumber Adószám
invoice_zipcode   Számlázási irányítószám
invoice_city      Számlázási város
invoice_street    Számlázási cím
coupon            Kuponkód
qty               Fő termék darabszáma
wqty              Bump darabszáma
custom            Egyedi belső azonosító
custom2           Második egyedi belső azonosító
sub               Automatikus űrlapbeküldés, például sub=1
```

Ajánlott SaaS használat:

```text
custom  = payment_attempt_id
custom2 = plan_id / product_id / service_id / tenant_id
```

---

## Webhook működés

A SalesForm webhook a `data` nevű POST mezőben küldi az adatokat JSON-kódolt formában.

Referencia PHP beolvasás:

```php
$payloadJson = $_POST['data'] ?? null;
$payload = json_decode($payloadJson, true);
```

A skillben szereplő kódpéldák csak referencia célúak. Az agentnek mindig a meglévő projekt architektúrájához kell illesztenie a megoldást.

---

## Webhook státuszok

```text
status=true     sikeres fizetés
status=false    sikertelen fizetés vagy átutalásos rendelés
status=cancel   előfizetés lemondása
```

Fontos szabály:

```text
Fizetést vagy jogosultságot csak webhook alapján szabad aktiválni.
A sikeres redirect URL önmagában nem bizonyítja a fizetést.
```

---

## Előfizetéses mezők

```text
trid     SalesForm tranzakciós azonosító
token    ismétlődő fizetéshez tartozó token
recdate  következő terhelés dátuma
day      ismétlődés gyakorisága napokban
time     részletfizetés részleteinek száma
```

Ezeket előfizetéses rendszerben érdemes eltárolni, mert ezek kellenek többek között:

```text
- csomagváltáshoz
- bankkártya cseréhez
- lemondási linkhez
- előfizetési státusz kezeléséhez
```

---

## Előfizetés váltás URL

Upgrade vagy downgrade esetén az új SalesForm terméklinkhez hozzá kell fűzni a korábbi előfizetés `trid` és `token` értékét.

```text
https://{salesform_domain}/t/r/{uj_termek_slug}/{korabbi_trid}/{korabbi_token}
```

Példa:

```text
https://sf.pelda.hu/t/r/eves-elofizetes/9464235405361497/12345678
```

---

## Bankkártya csere link

```text
https://{salesform_domain}/t/r/repay/{trid}/{token}
```

Példa:

```text
https://sf.pelda.hu/t/r/repay/9464235405361497/12345678
```

---

## Előfizetés lemondási link

```text
https://{salesform_domain}/index/cancel/{token}
```

Példa:

```text
https://sf.pelda.hu/index/cancel/12345678
```

---

## A skill fájl helye

A tényleges agent skill dokumentáció a repo gyökerében található:

```text
salesform_payment_skill.md
```

Ez tartalmazza a teljes SalesForm fizetési integrációs logikát, mezőmappinget, webhook szabályokat, URL paramétereket, előfizetéses működést és az agenteknek szóló fejlesztési utasításokat.

A `README.md` csak áttekintő dokumentáció. A kódoló agentnek mindig a `salesform_payment_skill.md` fájlt kell elsődleges forrásként használnia.

---

## Használat különböző kódoló rendszerekben

### Claude / Claude Code / Claude Cowork

Add meg a projekt mellé a `salesform_payment_skill.md` fájlt, majd ezt írd az agentnek:

```text
Olvasd be teljes egészében a salesform_payment_skill.md fájlt, és annak szabályai alapján készíts SalesForm fizetési integrációt a meglévő projektbe.

Ne feltételezz SalesForm checkout API-t. Fix SalesForm terméklinkeket kell használni URL paraméterezéssel. Fizetési státuszt csak webhook alapján módosíts.
```

### Antigravity

Tedd a fájlt a projekt gyökerébe, majd az első feladat előtt add ezt az utasítást:

```text
Mielőtt kódot írsz, olvasd be a salesform_payment_skill.md fájlt. A SalesForm integrációt kizárólag az abban leírt működés szerint készítsd el.

Először keresd meg a meglévő payment/order/subscription/user logikát, és ahhoz illeszd hozzá az integrációt. Ne írj új fizetési architektúrát, ha már van meglévő.
```

### Lovable

Lovable-ben érdemes a skill lényegét a projekt promptba vagy knowledge részbe tenni:

```text
Use the salesform_payment_skill.md rules as the payment integration specification.

SalesForm does not create checkout sessions through an API. The app must redirect users to a fixed SalesForm product URL with URL parameters. Payment status must only be updated from the SalesForm webhook.
```

### VS Code / Cursor / Windsurf jellegű AI fejlesztőkörnyezet

Tedd a fájlt a repo gyökerébe, majd hivatkozz rá a chatben:

```text
A SalesForm integrációhoz használd a salesform_payment_skill.md fájlt elsődleges specifikációként.

Először olvasd be, majd nézd át a meglévő billing/payment/subscription kódot, és csak utána javasolj módosítást.
```

### GitHub Copilot

Copilot esetén a fájl legyen a repo része, és a releváns fájlok szerkesztésekor hivatkozz rá kommentben vagy chatben:

```text
Follow the integration rules from salesform_payment_skill.md.
Do not assume a SalesForm checkout API.
Use fixed SalesForm product URLs with URL parameters.
Activate access only from webhook processing.
```

---

## Agent használati szabály

A skillt használó agent először mindig keresse meg a meglévő projektben:

```text
- user / customer logikát
- payment / order logikát
- subscription / entitlement logikát
- webhook fogadó mechanizmust
- jogosultság aktiválási szabályokat
```

Csak ezután illessze hozzá a SalesForm integrációt.

A skill nem ír elő konkrét frameworköt, adatbázis-táblát, controller nevet vagy service struktúrát.

---

## Ajánlott agent prompt

```text
Készíts SalesForm fizetési integrációt a meglévő projektbe.

Fontos: a SalesForm nem API-val hoz létre checkout sessiont. Előre létrehozott fix SalesForm terméklinket kell használni, amelyet URL paraméterekkel egészítünk ki.

A custom mezőbe kerüljön a belső payment_attempt_id vagy ennek projektbeli megfelelője.
A custom2 mezőbe kerüljön a plan_id / product_id / service_id.

A SalesForm webhook POST kérésben, data mezőben, JSON-kódolt payloadot küld. Csak webhook alapján szabad fizetettre állítani a rendelést vagy aktiválni a jogosultságot. A success redirect csak UX célú.

Ne írj új architektúrát, ha a projektben már van payment/order/subscription/jogosultság logika. Először keresd meg a meglévő megoldást, és ahhoz illeszd a SalesForm integrációt.
```

---

## Tesztelési ellenőrzőlista

```text
- SalesForm URL helyesen épül custom/custom2 paraméterrel
- URL paraméterek URL-kódolva vannak
- sub=1 csak validált adatokkal kerül használatra
- webhook fogadja a data mezőt
- hibás JSON nem aktivál jogosultságot
- status=true fizetettre állít
- status=false nem aktivál
- status=cancel lemondja az előfizetést
- dupla webhook nem aktivál kétszer
- trid/token mentődik előfizetésnél
- upgrade URL tartalmazza a régi trid/token értéket
```

---

## Licenc

A repo használható SalesForm-integrációk készítéséhez, belső dokumentációként, agent skillként vagy fejlesztői útmutatóként.

---

## Röviden

A SalesForm fizetési integráció lényege:

```text
Fix SalesForm terméklinket paraméterezünk.
A vásárlót a SalesForm fizetésre küldjük.
A külső szoftverben csak webhook alapján frissítjük a fizetési, előfizetési és jogosultsági státuszokat.
```
