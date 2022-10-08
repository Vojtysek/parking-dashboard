# Parking Dashboard 

## 🚀 Obsah

- (✅) Získávání [dat](#📊-data) z aplikace
- (✅) [Algoritmus](#algoritmus) dostupných míst
- (✅) [Databáze](#📦-databáze) parkovišť
- (✅) [Webový výstup](#🖥-webový-výstup) dashboardu

# 📊 Data

Na AXIS kameře běží aplikace na klasifikaci aut. Tato aplikace vrací data ve formátu JSON. V tomto JSONu jsou informace o jednotlivých parkovacích místech. Hlavní hodnotou je ale `state` která je `true` pokud je místo obsazené a `false` pokud je místo volné.

```json
{
  "01": {
    "id": "01",
    "state": "true"
  }
}
```

Stačí pouze přes jednoduchý [fetch](https://javascript.info/fetch) získat data z aplikace. Je ale zapotřebí ověřit identitu pomocí Digest autentizace. Na to je použit speciální [digest-fetch](https://www.npmjs.com/package/digest-fetch) balíček.

```js
const digestFetch = require("digest-fetch");
const digclient = new digestFetch(username, password, { algorithm: "MD5" });

await digclient.fetch(url)
    .then ...
```

# Algoritmus

Aby se docílilo hodnoty kolik je obsazených míst, je potřeba vytvořit algoritmus. Stačí projet každý jeden objekt a zjistit, zda-li je jeho proměnná `state` rovno `true`. Pokud tomu tak je tak přidáme +1. Výsledná hodnota se poté uloží do pole parkovišť.

```js
for (const key in json) {
  if (json[key].state) {
    dostupnéMísta[x]++;
  }
}
```

Hodnota pro maximání počet míst se jednoduše zjistí pomocí délky JSON objektu.

```js
max[x] = Object.keys(json).length;
```

Ale taktéž je potřeba zjistit, kolik aut denně zaparkuje na celém parkovišti. Proto si naši `Aktuální` hodnotu obsazených míst uložíme do proměnné, kterou poté poměříme s novou `Vypočítanou` hodnoutou dle předchozího algoritmu. V případě ze `Aktuální` hodnota je menšá než `Vypočítaná` hodnota, tak se k dennímu počtu přičte rozdíl mezi `Aktuální` a `Vypočítanou` hodnotou. Každý den přesně ve 23:59:59 se tato hodnota odešle do databáze a zresetuje.

```js
if(Aktuální < Vypočítaná) {
  denníNárust[x] += Vypočítaná - Aktuální;
}
```

# 📦 Databáze

Jak je na konci [Algoritmizace](#algoritmus) rečeno, na konci každného dne se denní hodnota odešle do databáze. Databáze je vytvořena pomocí [MongoDB](https://www.mongodb.com/) a je uložena na [Atlas](https://www.mongodb.com/cloud/atlas). Pro zjednodušení práce, ale hlavně prěhlednosti, je využívána [Mongoose](https://mongoosejs.com/) knihovna. Do schématu je uloženo aktuální datum a hodnoty pro jednotlivé parkoviště. 

## Získávání dat z databáze

Datum poslaná do MongoDB musí být ve specifickém formátu. Proto je třeba ho před odesláním upravit. K tomu slouží funkce formatDate, která převede datum do formátu `YYYY-MM-DD`. V případě že je měsíc nebo den jednociferný, tak před něj přidá 0 (padTo2Digits funkce). Takto upravené datum se poté dám odeslat. 

Při zpětním získávání data z databáze, pro zobrazení hodnot v statistikách na [stránce](#🖥-webový-výstup) je ale přijde ošlivý formát, protože mongo přidá `čas` 00-00-00, v jaké jsme `časové zóně` atd. Proto je potřeba upravit formát tak, aby byl formát čitelný. Je znovu využit formatDate (vše je rozděleno pomocí "-"), ale aby nebyl první rok, je třeba datum otočit a spojený tečkama. Obdobně byla potřeba upravit jednotlivá data jednotlivých parkovišť.

# 🖥 Webový výstup

Pro zobrazení dat na stránce je využit [Express](https://expressjs.com/) framework. Jedná se velice jednoduchý framework, který umožňuje vytvářet endpointy, které vrací různé data, v tomto případě HTML strukturu. Jednoduše stačí na specifikém endpointu vracet hodnoty z pole na indexu. Např. parkoviště 04 vrací všechny data z polích na indexu 4. Stránka se obnovuje každou vteřinu a půl. Pro spestření stránky je využit [Font Awesome](https://fontawesome.com/) pro ikonky.

Na levé straně je list všech parkovišť vedle kterých je číslo reprezentující `maximální` počet parkovacích míst. Všechny hodnoty jsou `dynamické`, tudíž v případě přidání nového parkovacího místa na jakémkoliv parkovišti se maximální hodnota automaticky přepočítá.
