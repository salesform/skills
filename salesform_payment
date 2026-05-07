# SalesForm fizetési integrációs skill agenteknek

## Cél

Ez a skill azt írja le, hogyan kell olyan szoftvert készíteni vagy bővíteni, amely a SalesFormot használja fizetésre, előfizetésre, számlázásra és sikeres fizetés utáni jogosultság-kezelésre.

A dokumentum célja nem egy konkrét PHP, Laravel, CodeIgniter, Node.js vagy más framework implementáció megadása, hanem az integrációs logika, adatáramlás, mezőmapping és biztonságos fejlesztési szabályok rögzítése.

---

## Legfontosabb alapelv

A SalesForm-integráció nem API-alapú checkout session létrehozás.

A külső szoftver nem hoz létre SalesForm rendelést API-n keresztül, és nem kér le dinamikus fizetési linket API-val.

A működés alapja:

```text
1. A SalesForm felhasználó létrehoz egy fix terméklinket a SalesFormban.
2. A külső szoftver ezt a fix terméklinket használja.
3. A szoftver URL paraméterekkel átadja az ismert adatokat.
4. A vásárló a SalesForm fizetési oldalára kerül.
5. Sikeres vagy sikertelen fizetés után a SalesForm webhookot küld.
6. A külső szoftver a webhook alapján frissíti a belső státuszokat.
```

Mentális modell:

```text
Saját szoftver
  ↓
Fix SalesForm termék URL + URL paraméterek
  ↓
SalesForm checkout / banki fizetés
  ↓
SalesForm webhook
  ↓
Saját szoftver: fizetési státusz, jogosultság, előfizetés frissítése
```

---

## Mit szabad és mit nem szabad feltételezni

### Szabad feltételezni

```text
- Van egy előre létrehozott SalesForm terméklink.
- A terméklink URL paraméterezhető.
- A custom és custom2 mezők használhatók külső azonosítók átadására.
- Sikeres fizetéskor webhook érkezhet a külső szoftverbe.
- A webhook data nevű POST mezőben, JSON-kódolt formában érkezik.
- A trid és token értékeket előfizetésnél érdemes eltárolni.
```

### Nem szabad feltételezni

```text
- Ne feltételezd, hogy van SalesForm checkout API.
- Ne feltételezd, hogy a szoftver API-val hoz létre rendelést a SalesFormban.
- Ne feltételezd, hogy API-val lehet fizetési linket generálni.
- Ne feltételezd, hogy a success URL bizonyítja a sikeres fizetést.
- Ne feltételezd, hogy minden webhookban minden opcionális mező szerepel.
- Ne feltételezd, hogy a SalesForm domain mindig sf.salesform.hu.
- Ne feltételezd, hogy az email önmagában biztonságos belső azonosító.
- Ne írd át automatikusan a projekt meglévő payment/order/subscription architektúráját.
```

---

## Agent fejlesztési szabály

A kódoló agent feladata nem új fizetési architektúra kitalálása, hanem a meglévő projektlogikához való illesztés.

Mielőtt kódot ír:

```text
1. Keresse meg a meglévő user/customer modellt.
2. Keresse meg a meglévő order/payment/subscription/billing logikát.
3. Keresse meg, van-e már webhook fogadó mechanizmus.
4. Keresse meg, hogyan történik jogosultság, csomag vagy státusz aktiválás.
5. Csak ezekhez illessze hozzá a SalesForm integrációt.
```

Ha nincs meglévő fizetési logika, akkor minimális, különálló integrációs réteget hozzon létre, ne írja át a teljes alkalmazást.

---

## Kódpéldák használati szabálya

A skillben szereplő kódrészletek csak referencia célúak.

Az agent:

```text
- ne másolja vakon a példákat;
- ne hozzon létre nem létező service/metódus hívásokat;
- ne írja felül a projekt meglévő naming conventionjeit;
- ne cserélje le a meglévő fizetési architektúrát;
- a SalesForm mezőneveket tartsa meg pontosan;
- a belső változóneveket igazítsa a meglévő projekthez.
```

---

## Ajánlott integrációs adatáramlás

```text
1. Felhasználó rákattint egy fizetési / előfizetési / upgrade gombra.
2. A szoftver létrehoz vagy kiválaszt egy belső fizetési próbálkozás azonosítót.
3. A szoftver kiválasztja a megfelelő SalesForm terméklinket.
4. A szoftver hozzáadja az URL paramétereket.
5. A böngésző átirányítódik a SalesForm oldalra.
6. A vásárló fizet.
7. A SalesForm webhookot küld.
8. A szoftver webhook alapján fizetettre, sikertelenre vagy lemondottra állítja a belső rekordot.
9. A szoftver aktiválja, hosszabbítja, módosítja vagy lemondja a hozzáférést.
```

---

## SalesForm terméklink

Minden SalesForm terméknek van saját fix URL-je.

Általános forma:

```text
https://{salesform_domain}/t/r/{termek_slug}
```

Példák:

```text
https://sf.salesform.hu/t/r/havi-elofizetes
https://sf.salesform.hu/t/r/video-oktatas-pendrive-on
https://sf.salesform.hu/t/r/idopont-foglalas-neve
```

Saját domainnel:

```text
https://sf.ugyfel-domain.hu/t/r/havi-elofizetes
```

A `{salesform_domain}` nem fix. Ezt konfigurációból kell kezelni.

---

## Minimum szükséges külső konfiguráció

Egy szoftverben legalább ezekre van szükség:

```text
salesform_product_url
success_redirect_url
failed_redirect_url
webhook_url
```

Előfizetés vagy több csomag esetén:

```text
internal_plan_id → SalesForm terméklink mapping
```

Példa logikai mapping:

```text
free_trial → nincs SalesForm link
pro_monthly → https://sf.pelda.hu/t/r/pro-havi-elofizetes
pro_yearly → https://sf.pelda.hu/t/r/pro-eves-elofizetes
vip_package → https://sf.pelda.hu/t/r/vip-csomag
```

---

## Belső azonosítók átadása

A SalesForm két fő egyedi mezőt tud URL paraméterből fogadni:

```text
custom
custom2
```

Ajánlott használat:

```text
custom = belső fizetési próbálkozás azonosítója
custom2 = belső csomag / termék / szolgáltatás azonosítója
```

Példa:

```text
custom=pay_abc123
custom2=pro_monthly
```

Egyszerűbb integrációnál elfogadható:

```text
custom=user_42
custom2=pro_monthly
```

Jobb megoldás:

```text
custom=payment_attempt_id
custom2=plan_id
```

Miért jobb a payment_attempt_id?

```text
- egyértelműen egy konkrét fizetési indításhoz tartozik;
- nem csak a felhasználót, hanem a vásárlási szándékot is azonosítja;
- idempotens webhook feldolgozásnál biztonságosabb;
- több termék, több csomag, több próbálkozás esetén tisztább.
```

---

## URL paraméterek űrlap előtöltéshez

A SalesForm terméklink query paraméterekkel tölthető elő.

Példa:

```text
https://sf.pelda.hu/t/r/pro-havi-elofizetes?email=hello@example.com&name=Teszt%20Elek&custom=pay_abc123&custom2=pro_monthly
```

Szabály:

```text
- Az első query paraméter előtt ? jel van.
- A további paramétereket & jel választja el.
- Minden értéket URL-kódolni kell.
- Az üres vagy ismeretlen mezőket nem kötelező átadni.
```

---

## SalesForm URL paraméterek

### Megrendelő adatai

```text
name = megrendelő neve
telephone = telefonszám
email = email cím
```

Példa:

```text
name=Bártfai Balázs
telephone=06303550880
email=hello@salesform.hu
```

---

### Számlázási adatok

```text
invoice_name = számlázási név
invoice_vatnumber = adószám
invoice_zipcode = irányítószám
invoice_city = város
invoice_street = utca, házszám
```

Példa:

```text
invoice_name=Netlight Consulting Kft
invoice_vatnumber=14188127-2-42
invoice_zipcode=1148
invoice_city=Budapest
invoice_street=Bánki Donát utca 12/b
```

---

### Szállítási adatok

```text
ship_name = szállítási név
ship_zipcode = szállítási irányítószám
ship_city = szállítási város
ship_street = szállítási utca, házszám
```

---

### Kupon és mennyiségek

```text
coupon = kuponkód
qty = fő termék darabszáma
wqty = bump darabszáma
```

---

### Egyedi azonosítók

```text
custom = egyedi azonosító
custom2 = második egyedi azonosító
```

SaaS integrációnál ajánlott:

```text
custom = payment_attempt_id
custom2 = plan_id / product_id / service_id / tenant_id
```

---

### Automatikus űrlapbeküldés

```text
sub=1
```

A `sub=1` automatikusan elküldi a SalesForm űrlapot, így a vásárló közvetlenül a banki fizetési oldalra kerülhet.

Csak akkor használd, ha minden kötelező adat ismert és a saját szoftver már validálta.

Minimum ajánlott adatok `sub=1` használatához:

```text
name
telephone
email
invoice_name
invoice_zipcode
invoice_city
invoice_street
```

Céges vásárlásnál:

```text
invoice_vatnumber
```

Fontos:

```text
sub=1 esetén a vásárló nem ellenőrzi kézzel az űrlapot.
Ha hibás adatot adsz át, hibás rendelés és hibás számla készülhet.
```

---

## Teljes URL példa űrlap előtöltéssel

```text
https://sf.pelda.hu/t/r/pro-havi-elofizetes?name=B%C3%A1rtfai%20Bal%C3%A1zs&telephone=06303550880&email=hello%40salesform.hu&invoice_name=Netlight%20Consulting%20Kft&invoice_vatnumber=14188127-2-42&invoice_zipcode=1148&invoice_city=Budapest&invoice_street=B%C3%A1nki%20Don%C3%A1t%20utca%2012%2Fb&custom=pay_abc123&custom2=pro_monthly
```

## Teljes URL példa közvetlen banki fizetéshez

```text
https://sf.pelda.hu/t/r/pro-havi-elofizetes?name=B%C3%A1rtfai%20Bal%C3%A1zs&telephone=06303550880&email=hello%40salesform.hu&invoice_name=Netlight%20Consulting%20Kft&invoice_vatnumber=14188127-2-42&invoice_zipcode=1148&invoice_city=Budapest&invoice_street=B%C3%A1nki%20Don%C3%A1t%20utca%2012%2Fb&custom=pay_abc123&custom2=pro_monthly&sub=1
```

---

## Korábbi rendelési adatokkal előtöltött űrlap

Ha ismert a korábbi SalesForm tranzakciós azonosító, akkor a terméklink végére tehető a `trid`.

Forma:

```text
https://{salesform_domain}/t/r/{termek_slug}/{trid}
```

Példa:

```text
https://sf.pelda.hu/t/r/havi-elofizetes/9464235405361497
```

Automatikus posztolással:

```text
https://sf.pelda.hu/t/r/havi-elofizetes/9464235405361497?sub=1
```

---

## Előfizetés váltás: upgrade / downgrade

Előfizetés váltásnál az új SalesForm terméklinket kell használni, de a régi előfizetés `trid` és `token` értékét is át kell adni.

Forma:

```text
https://{salesform_domain}/t/r/{uj_termek_slug}/{korabbi_trid}/{korabbi_token}
```

Példa:

```text
https://sf.pelda.hu/t/r/eves-elofizetes/9464235405361497/12345678
```

Jelentés:

```text
uj_termek_slug = az új csomag SalesForm terméklinkje
korabbi_trid = a korábbi előfizetés tranzakciós azonosítója
korabbi_token = a korábbi előfizetés tokenje
```

Sikeres új fizetés után a SalesForm a korábbi előfizetést lecseréli az újra.

SaaS oldalon a csomagváltást webhook alapján kell véglegesíteni.

Ajánlott paraméterezés upgrade esetén:

```text
https://sf.pelda.hu/t/r/eves-elofizetes/{old_trid}/{old_token}?custom=pay_upgrade_123&custom2=pro_yearly
```

---

## Bankkártya csere link

Ha az előfizető bankkártyát szeretne cserélni, a szoftver a SalesForm kártyacsere linket mutathatja meg.

Forma:

```text
https://{salesform_domain}/t/r/repay/{trid}/{token}
```

Példa:

```text
https://sf.pelda.hu/t/r/repay/9464235405361497/12345678
```

A `trid` és `token` értékeket a sikeres előfizetés webhookjából kell eltárolni.

---

## Előfizetés lemondási link

Forma:

```text
https://{salesform_domain}/index/cancel/{token}
```

Példa:

```text
https://sf.pelda.hu/index/cancel/12345678
```

A saját szoftverben a lemondás státuszát ne a link megnyitása alapján állítsd át, hanem a SalesForm webhook alapján.

---

## Sikeres és sikertelen vásárlás utáni böngészős redirect

A SalesFormban beállítható sikeres és sikertelen vásárlás utáni URL.

Ezek célja:

```text
- vásárlói élmény;
- sikeres oldal megjelenítése;
- sikertelen oldal megjelenítése;
- belső rendszerben státusz ellenőrzése;
- nem fizetési bizonyíték.
```

A redirect URL paraméterezhető SalesForm behelyettesítőkkel.

Használható változók:

```text
[nev]
[email]
[cim]
[tel]
[termek]
[bump]
[ar]
[source]
[trid]
[egyedi]
[egyedi2]
[allp]
```

Jelentés:

```text
[nev] = név
[email] = email
[cim] = cím
[tel] = telefonszám
[termek] = termék neve
[bump] = bump neve
[ar] = teljes ár
[source] = forrás / UTM source
[trid] = tranzakciós azonosító
[egyedi] = custom értéke
[egyedi2] = custom2 értéke
[allp] = minden eredeti URL paraméter a kezdő ? nélkül, például fbclid esetén
```

Példa sikeres redirect URL:

```text
https://app.pelda.hu/payment/success?payment_ref=[egyedi]&plan=[egyedi2]&trid=[trid]&email=[email]&price=[ar]
```

Példa sikertelen redirect URL:

```text
https://app.pelda.hu/payment/failed?payment_ref=[egyedi]&plan=[egyedi2]&email=[email]
```

Fontos:

```text
A redirect URL-re érkezés nem bizonyítja, hogy a fizetés sikeres volt.
Jogosultságot csak webhook alapján szabad aktiválni.
```

---

## Webhook működés

A SalesForm webhook POST kérést küld a beállított webhook URL-re.

Az adatok nem raw JSON bodyként, hanem `data` nevű POST mezőben érkeznek JSON-kódoltan.

Logikai forma:

```text
POST /salesform-webhook
Content-Type: application/x-www-form-urlencoded vagy multipart/form-data jellegű POST

data={JSON-kódolt SalesForm payload}
```

Referencia PHP beolvasás:

```php
$payloadJson = $_POST['data'] ?? null;
$payload = json_decode($payloadJson, true);
```

Ez csak a SalesForm payload fogadásának módját mutatja. Az agent a projekt meglévő request-kezeléséhez igazítsa.

---

## Webhook status mező

A `status` mező jelentése:

```text
true = sikeres fizetés
false = sikertelen fizetés vagy átutalásos rendelés
cancel = előfizetés lemondása
```

Pszeudókód:

```text
ha status === true:
    fizetés sikeres
    belső payment/order státusz paid
    jogosultság aktiválása vagy hosszabbítása
    trid/token/recdate eltárolása, ha van

ha status === "cancel":
    előfizetés lemondva
    belső subscription státusz cancelled

ha status === false:
    fizetés sikertelen vagy nem azonnali fizetés
    jogosultságot nem aktiválunk
    payment/order státusz failed vagy pending_payment
```

---

## Fontos webhook mezők

### Vevőadatok

```text
name
telephone
email
invoice_name
invoice_zipcode
invoice_city
invoice_street
ship_name
ship_zipcode
ship_city
ship_street
ship_country
```

### Mérési és egyedi mezők

```text
seller
utm_all
custom
custom2
custominput1
custominput2
```

### Termékek

```text
product
```

A `product` tömbben:

```text
product[0] = fő termék
product[1] = bump, ha van
```

Termékmezők:

```text
pid = SalesForm termékazonosító
product = terméknév
qty = mennyiség
vat = ÁFA kulcs
price = bruttó ár
```

### Kupon

```text
coupon
```

A `coupon` csak akkor létezik, ha használtak kupont.

Kuponmezők:

```text
code = kuponkód
value = kedvezmény értéke
type = kupon típusa, 1 fix összeg, 2 százalékos
```

### Fizetési és előfizetéses mezők

```text
price = rendelés bruttó végösszege
status = fizetési státusz
trid = tranzakciós azonosító
token = ismétlődő fizetés tokenje
day = ismétlődés gyakorisága napokban
time = részletfizetés részleteinek száma
recdate = következő terhelés dátuma
ipncc.token = token
ipncc.mask = maszkolt kártyaszám
ipncc.exp = bankkártya lejárata
```

---

## Webhook minta payload szerkezet

Referencia szerkezet, nem kötelezően minden mezővel:

```text
name
telephone
email
invoice_name
invoice_zipcode
invoice_city
invoice_street
seller
utm_all
custom
custom2
ship_name
ship_zipcode
ship_city
ship_street
ship_country
ship_price
product[]
coupon
price
status
trid
token
custominput1
custominput2
day
time
recdate
ipncc
```

Az agent minden opcionális mezőt ellenőrizzen létezés előtt.

---

## Ajánlott mezőmapping SaaS rendszerhez

### SalesForm → saját szoftver

```text
custom → payment_attempt_id / external_reference
custom2 → plan_id / product_id / service_id / tenant_id
email → customer email
status → fizetési esemény eredménye
trid → SalesForm tranzakciós azonosító
token → SalesForm előfizetési token
price → bruttó fizetett összeg
recdate → következő terhelés / periódus vége
product[0].pid → SalesForm termékazonosító
product[0].product → SalesForm terméknév
ipncc.mask → maszkolt kártyaszám
ipncc.exp → kártya lejárat
```

### Saját szoftver → SalesForm URL

```text
payment_attempt_id → custom
plan_id / service_id → custom2
user.name → name
user.email → email
user.telephone → telephone
billing.name → invoice_name
billing.tax_number → invoice_vatnumber
billing.zip → invoice_zipcode
billing.city → invoice_city
billing.street → invoice_street
coupon_code → coupon
quantity → qty
```

---

## Belső adatmodell elvárás

A skill nem ír elő konkrét táblaneveket, de a projektben legyen valamilyen megfelelője ezeknek:

### Fizetési próbálkozás

Tárolandó logikai mezők:

```text
belső azonosító
user/customer azonosító
belső csomag/termék azonosító
státusz: pending / paid / failed / cancelled
SalesForm trid
SalesForm token
bruttó összeg
nyers webhook payload
létrehozás dátuma
fizetés dátuma
```

### Előfizetés

Előfizetéses terméknél tárolandó:

```text
user/customer azonosító
plan_id
státusz: active / cancelled / expired / past_due
SalesForm trid
SalesForm token
következő terhelés dátuma vagy periódus vége
kártya maszkolt száma, ha szükséges
kártya lejárata, ha szükséges
```

### Webhook eseménynapló

Ajánlott tárolni:

```text
egyedi esemény hash vagy deduplikációs kulcs
trid
custom
custom2
status
nyers payload
feldolgozás státusza
feldolgozás dátuma
```

---

## Idempotencia szabály

A SalesForm webhookot úgy kell kezelni, hogy ugyanaz az esemény többször is megérkezhet.

Az agent készítsen vagy használjon deduplikációt.

Lehetséges deduplikációs kulcsok:

```text
trid + status + custom + price
vagy
hash(nyers payload)
vagy
projektben meglévő webhook event azonosítási módszer
```

Szabály:

```text
Ha ugyanazt a webhookot már feldolgoztuk, HTTP 200 jellegű választ kell adni, de nem szabad újra aktiválni a jogosultságot.
```

---

## Webhook feldolgozási pszeudókód

```text
1. Olvasd be a data POST mezőt.
2. JSON dekódolás.
3. Ha hibás a payload, naplózd és adj hibás választ.
4. Készíts deduplikációs kulcsot.
5. Ha már feldolgozott esemény, állj meg sikeres válasszal.
6. Naplózd a nyers webhook payloadot.
7. Olvasd ki: status, custom, custom2, trid, token, price, recdate.
8. custom alapján keresd meg a belső fizetési próbálkozást vagy rendelést.
9. Ha nincs találat, jelöld kézi ellenőrzésre, ne aktiválj automatikusan.
10. status=true esetén:
    - fizetés paid
    - jogosultság aktiválása / hosszabbítása
    - subscription adatok frissítése
    - trid/token/recdate mentése
11. status=cancel esetén:
    - subscription keresése token vagy trid alapján
    - subscription cancelled
12. status=false esetén:
    - ne aktiválj jogosultságot
    - státusz failed vagy pending_payment
13. Jelöld feldolgozottnak a webhook eseményt.
```

---

## Sikeroldal javasolt működése

A sikeroldal ne aktiválja a hozzáférést közvetlenül.

Pszeudókód:

```text
1. Vásárló megérkezik a success URL-re.
2. A szoftver kiolvassa a payment_ref / custom / trid paramétert, ha van.
3. Megnézi a belső payment_attempt státuszát.
4. Ha már paid: sikeres hozzáférés oldal.
5. Ha még pending: fizetés feldolgozás alatt oldal.
6. Ha failed: sikertelen fizetés oldal.
```

Ajánlott pending szöveg:

```text
A fizetés feldolgozása folyamatban van. Ha a fizetés sikeres volt, a hozzáférés rövidesen aktiválódik.
```

---

## Tipikus integrációs esetek

### Egyszeri vásárlás

```text
1. Fizetés indítása gomb.
2. payment_attempt létrehozása.
3. SalesForm URL építése custom és custom2 paraméterrel.
4. Redirect SalesFormra.
5. Webhook status=true.
6. Belső rendelés paid.
7. Termék / szolgáltatás aktiválása.
```

### Előfizetés indítása

```text
1. Csomag kiválasztása.
2. SalesForm előfizetés terméklink használata.
3. custom = payment_attempt_id.
4. custom2 = plan_id.
5. Webhook status=true.
6. trid/token/recdate mentése.
7. Subscription active.
```

### Előfizetés megújulása

```text
1. SalesForm kezeli az ismétlődő terhelést.
2. Webhook érkezik sikeres újraterhelésről.
3. Szoftver azonosítja az előfizetést trid/token/custom alapján.
4. Hozzáférés hosszabbítása.
5. recdate frissítése.
```

### Upgrade / downgrade

```text
1. Szoftver ismeri a régi subscription trid/token értékét.
2. Új SalesForm terméklinket épít.
3. URL forma: /uj-termek/{old_trid}/{old_token}
4. custom = új payment_attempt_id.
5. custom2 = új plan_id.
6. Webhook után csomag frissítése.
```

### Lemondás

```text
1. User SalesForm cancel linken lemondja.
2. SalesForm webhook status=cancel.
3. Szoftver token/trid alapján megkeresi az előfizetést.
4. Subscription cancelled.
```

### Bankkártya csere

```text
1. Szoftver megjeleníti a repay linket.
2. User SalesFormon kártyát cserél.
3. Következő releváns webhookból a kártyaadatok frissíthetők, ha érkeznek.
```

---

## URL builder elvárás

Az agent ne string konkatenációval, hanem biztonságos URL építő logikával dolgozzon.

Elvárt viselkedés:

```text
- base SalesForm product URL megtartása;
- opcionális trid hozzáfűzése path elemként;
- opcionális token hozzáfűzése path elemként;
- query paraméterek URL-kódolása;
- üres paraméterek kihagyása;
- meglévő projekt URL helperének használata, ha van.
```

Pszeudókód:

```text
function buildSalesFormUrl(productUrl, params, trid = null, token = null):
    url = productUrl végéről / eltávolítása

    ha trid van:
        url += /urlencode(trid)

    ha token van:
        url += /urlencode(token)

    query = nem üres params URL-kódolva

    ha query nem üres:
        url += ? + query

    return url
```

---

## Minimum működő integráció

A legkisebb helyes SalesForm integráció:

```text
1. Konfigurált SalesForm terméklink.
2. Fizetési gomb vagy checkout indító endpoint.
3. Belső fizetési referencia létrehozása vagy kiválasztása.
4. custom paraméter átadása.
5. custom2 paraméter átadása, ha több csomag/termék van.
6. SalesForm webhook endpoint.
7. Webhook data mező JSON dekódolása.
8. status=true esetén belső fizetés paid.
9. status=false esetén nincs aktiválás.
10. status=cancel esetén előfizetés cancelled.
```

---

## Biztonsági szabályok

```text
- Fizetést csak webhook alapján tekints sikeresnek.
- Redirect URL alapján soha ne aktiválj jogosultságot.
- custom mezőben ne érzékeny adatot adj át.
- Ne tedd az URL-be jelszót, API kulcsot, titkos tokent.
- Webhook endpoint legyen nehezen kitalálható vagy védett.
- Minden webhook payloadot naplózz.
- Feldolgozás legyen idempotens.
- Ismeretlen custom értéket ne aktiválj automatikusan.
- sub=1 előtt validáld a számlázási adatokat.
- trid/token értékeket előfizetési azonosítóként kezeld.
```

---

## Hibakezelési szabályok

### Hiányzó `data`

```text
- Hibás webhook kérés.
- Naplózd.
- Ne aktiválj semmit.
```

### Hibás JSON

```text
- Naplózd.
- Ne aktiválj semmit.
```

### Hiányzó `custom`

```text
- Ne aktiválj automatikusan.
- Próbálhatsz email + termék alapján egyeztetni, de csak óvatosan.
- Jelöld kézi ellenőrzésre.
```

### Ismeretlen payment_attempt

```text
- Naplózd unmatched státusszal.
- Ne aktiválj jogosultságot automatikusan.
```

### Dupla webhook

```text
- Adj sikeres választ.
- Ne dolgozd fel újra.
```

### status=false

```text
- Ne aktiválj hozzáférést.
- Jelöld sikertelen vagy függő fizetésnek.
```

### status=cancel

```text
- Keresd meg az előfizetést token vagy trid alapján.
- Állítsd cancelled státuszra.
```

---

## Tesztelési ellenőrzőlista

Az agent készítsen vagy javasoljon tesztet ezekre:

```text
- SalesForm URL helyesen épül custom/custom2 paraméterrel.
- URL paraméterek URL-kódolva vannak.
- sub=1 csak akkor kerül bele, ha engedélyezett.
- Webhook fogadja a data mezőt.
- Hibás JSON nem aktivál jogosultságot.
- status=true fizetettre állít.
- status=false nem aktivál.
- status=cancel lemondja az előfizetést.
- Dupla webhook nem aktivál kétszer.
- Hiányzó custom kézi ellenőrzésre kerül.
- trid/token mentődik előfizetésnél.
- upgrade URL helyesen tartalmazza a régi trid/token értéket.
```

---

## Agent feladatvégzési sorrend

Ha az agent SalesForm integrációt kap feladatként, ezt kövesse:

```text
1. Olvasd fel a meglévő projekt fizetési, rendelési, user és jogosultsági logikáját.
2. Azonosítsd, hol kell fizetési linket indítani.
3. Azonosítsd, hol kell státuszt frissíteni.
4. Készíts vagy használj belső payment_attempt jellegű rekordot.
5. Építs SalesForm URL-t a fix terméklinkből.
6. Add át custom/custom2 paraméterben a belső referenciákat.
7. Készíts webhook fogadó végpontot a projekt szabályai szerint.
8. A webhookból csak a szükséges mezőket mappeld.
9. A meglévő jogosultságkezelést hívd, ne találj ki újat, ha már van.
10. Írj tesztet vagy tesztelési útmutatót.
```

---

## Rövid agent prompt minta

```text
Készíts SalesForm fizetési integrációt a meglévő projektbe.

Fontos: a SalesForm nem API-val hoz létre checkout sessiont. Előre létrehozott fix SalesForm terméklinket kell használni, amelyet URL paraméterekkel egészítünk ki.

A custom mezőbe kerüljön a belső payment_attempt_id vagy ennek projektbeli megfelelője.
A custom2 mezőbe kerüljön a plan_id / product_id / service_id.

A SalesForm webhook POST kérésben, data mezőben, JSON-kódolt payloadot küld. Csak webhook alapján szabad fizetettre állítani a rendelést vagy aktiválni a jogosultságot. A success redirect csak UX célú.

Ne írj új architektúrát, ha a projektben már van payment/order/subscription/jogosultság logika. Először keresd meg a meglévő megoldást, és ahhoz illeszd a SalesForm integrációt.

Készíts:
- SalesForm URL buildert vagy meglévő helper használatát;
- checkout indító logikát;
- webhook fogadó logikát;
- idempotens feldolgozást;
- status=true / false / cancel kezelést;
- trid/token/recdate mentést előfizetéshez;
- tesztelési útmutatót.
```

---

## Lényeg egy mondatban

A SalesForm fizetési integráció lényege: fix SalesForm terméklinket paraméterezünk, a vásárlót a SalesForm fizetésre küldjük, majd a külső szoftverben kizárólag webhook alapján állítjuk át a fizetési, előfizetési és jogosultsági státuszokat.
