---
title: Laboration 9
code: "da344a-da355a"
---

# Laboration: extra - Mediahantering med server-API

Syftet med denna laboration är att vi ska lära oss att ladda upp media genom ett webbgränssnitt. Vi vill sedan spara denna media någonstans (vilket kommer att vara i en databas på er dvwebb), för att sedan kunna visa upp detta för användaren igen. Vi kommer således att fokusera på:

- Bilder
- Video
- Ljud

För att kunna spara bilderna i er databas (som finns inbyggt i dvwebb) är det några steg vi måste göra innan vi kan bygga vårt gränssnitt.

Tanken är att vi ska ha en databas som håller information om den uppladdade median (bilder/ljud/video). Det som ska sparas är t.ex. et `id`, `sökvägen`, `titeln`, `typ av media`, `uppladdningstid/dag`. Så att vi har koll på den uppladdade median.
{:.info}

## 1. Aktivera dvwebb/ddwap

`dvwebb` och `ddwap` är **samma sak** och refererar till samma plats, som dumt nog har två olika namn. Så när ni läser/hör detta så syftar vi på den webbpubliseringstjänst som Malmö universitet erbjuder som har stöd för databas (MySQL). Ni hittar tjänsten [här](http://ddwap.mah.se/dotnet/index.aspx). Så börja med följande:

1. [Aktivera dvwebb här](https://ddwap.mah.se/dotnet/create/)

Se till att komma ihåg det lösenord som ni använder för databasen - vi kommer att behöva det strax igen!
{:.info}

## 2. Skapa din databas

1. Logga in [här](http://ddwap.mah.se/mysqlbrowser) och ange ditt datorid (samma som till Canvas) och det lösenord som du precis skapade.
2. Välj uppe till vänster i gränssnittet (klicka på den) den databas som har samma namn som ert datorid.
3. Skapa den tabell som vi behöver för att hålla reda på vår media, enligt följande:

```sql
CREATE TABLE `media` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `path` varchar(200) NOT NULL,
  `title` varchar(200) NULL,
  `type` varchar(100) NOT NULL,
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;
```

- Testa att ställa en SQL-fråga så att ni ser att det fungerar, t.ex. `SELECT * FROM media`.

Tänk på att du är inne på rätt databas när du ska hitta tabellen `media`, det är alltså den databas med samma namn som ditt användarnamn som gäller.
{:.info}

## 3. PHP-skript för att ladda upp filer

Vi kommer i denna labb att använda oss utav ett php-skript (en php-fil) som agerar mellanhand mellan vår klient (webbsidan) och vår databas - alltså som ett "mini-API" mot vår databas. För att använda detta API så ladda ner följande [php-fil](14/server.php) (högerklicka > Spara som...) och ladda upp den på er dvwebb.

- Öppna filen "server.php" och byt på rad 1 & 2 ut *användarnamn* och *lösenord* till era personliga uppgifter.

För att ladda upp filen på dvwebb, så behöver vi först ansluta till dvwebb. Detta fungerar på samma sätt, genom FTP som när ni laddat upp filer på webshare. Så öppna valfritt FTP-program (t.ex. FileZilla) och anslut med följande uppgifter:

- Host: `ftpes://dvwebb.mah.se`
- Username: `Ditt Canvas-användarnamn`
- Password: `Ditt Canvas-lösenord`

Ladda sedan upp filen `server.php` i root-mappen för din dvwebb.

- I samma mapp som ni placerat filen "server.php" skapa mapparna "audio", "photo", "video". Det är i dessa mappar som era mediafiler kommer att bli uppladdade. Det borde se ut något i stil med:

![FTP](13/ftp.png)

## 4. Att använda vårt "mini-API"


### 4.1. Ladda upp medie-objekt

Ni skickar enklast ett anrop genom att skicka iväg ett formulär, genom ajax - som vi tittade på vid [föreläsningen (ej publicerad än)](#). Det som krävs från servern är minimum (ska skicks med metoden `POST`):

- "media" (filen som ska laddas upp)
- "type" (vilken typ är objektet? (audio/video/photo)

Så ett formulär med tillhörande JavaScript-kod skulle kunna se ut såhär (HTML):

```html
<form action="server.php" enctype="multipart/form-data" method="post" id="myForm">
  <select name="type">
  <option value="">Välj typ här</option>
  <option value="photo">Foto</option>
  <option value="video">Video</option>
  <option value="audio">Ljud</option>
  </select>
  <input type="file" name="media" class="btn">
  <input type="submit" value="Spara media" class="btn">
</form>
```

och såhär (JavaScript):

```js
// När man skickar iväg formuläret (klickat på knappen "Spara media")
$("#myForm").submit(function(e) {
  // Förhindrar att vi skickas iväg till en ny sida (standardhändelsen)
  e.preventDefault();
  // Hämtar formulärsdata (värde från drop-down menyn, samt filen)
  var formData = new FormData(this);

  // Gör ett ajax-anrop
  $.ajax({
    url: $(this).attr("action"), // Till adressen "server.php"
    type: $(this).attr("method"), // Med metoden "post"
    data: formData, // Vår data vi skickar med
    dataType: "JSON", // Hur vi ska tolka den data vi får tillbaka (som JSON)
    cache: false, // Vi tillåter inte att webbläsaren att cacha några resultat
    contentType: false, // Vi vill inte att jQuery ska bestämma hur vårt innehåll ska tolkas
    processData: false // Vi tillåter inte att jQuery att processa vår data (som strängar)
  }).done(function (data) {
    // Om vi får ett lyckat svar
    console.log(data);
  }).fail(function(data){
    // Om vi får ett misslyckat svar
    console.log(data);});
});
```

Exempelsvar från ett lyckat API-anrop (objektet har laddats upp):

```json
{
  "path":"path-to-file",
  "success":true,
  "message":"File successfully uploaded"
}
```

### 4.2. Hämta medie-objekt

För att hämta information om sina media-objekt som man laddat upp, kan man använda följande anrop:

`server.php?action=getMedia&type=photo`

Det krävs som ni ser att man skickar med två parametrar i sitt anrop (GET-anrop):

- "action" - den funktion vi vill köra, i detta fall "getMedia"
- "type" - vilken typ av objekt vi vill hämta ("video"/"photo"/"auido"/"")

Ett exempelsvar från anropet ovan kan se ut såhär:

```json
{
  "files":[
    {
      "path":"photo\/bild3.jpg",
      "type":"photo",
      "title": "title-of-media",
      "timestamp": "2017-02-19 16:17:18"
    },
    {
      "path":"photo\/bild2.jpg",
      "type":"photo",
      "title": "title-of-media",
      "timestamp": "2017-02-19 16:17:18"
    },
    {
      "path":"photo\/bild1.jpg",
      "type":"photo",
      "title": "title-of-media",
      "timestamp": "2017-02-19 16:17:18"
    }
  ]
}
```

## 5. Övningar för att skapa ert media-bibliotek

### 5.1. Titel till medie-objekten

Börja med att lägga till ett fält i ert formulär som tillåter användaren att lägga till en titel till det objekt som ska laddas upp. Det behövs skickas från ert formulär med nyckeln "name" (t.ex. `name="title"`, lämpligen vid ett text-fält).

### 5.2. Visa media

Ni ska nu skapa en funktion som - när sida laddas - ladda in all den media som finns sparad på er server (och i er databas). Vi kan visa medie-objekten på valfritt sätt, men tänk på att videoklipp ska visas med elementet `<video>`, ljudklipp med elementet `<audio>` och bilder ska visas med elementet `<img>`. Dokumentation för ett anrop till ert API finns under punkt [4.2. Hämta medie-objekt](#42-h%C3%A4mta-medie-objekt
### 5.3. Förbättra medie-uppladdningen

Vi ska nu förbättra (i så stor möjlighet som går) vår medie-uppladdning genom att försöka tvinga webbläsare att öppna rätt app för att välja rätt typ av fil. Alltså har jag i drop-down menyn valt "foto" vill vi att när vi klickar på knappen "Välj fil" ska vi komma direkt till kamerna, har jag valt "video" ska vi komma direkt till filmkameran och har jag valt "ljud" ska vi komma direkt till ljudinspelaren för telefonen/surfplattan. Hur man gör detta kan ni läsa mer om [här](http://mobilehtml5.org/ts/?id=23).

### 5.4. Förbättra visningen av uppladdad media

Ni ska nu implementera en funktion som gör det möjligt för användaren att välja om man vill visa alla media-objekt (som nu), eller bara bilder/video/ljud. Ni får **inte** bara dölja de olika sektionerna, då vi vill optimera laddningstiderna för vår webbapp. Väljer vi då bilder ska således inga videos/ljudklipp hämtas. **Bonus**: Spara användarens val genom i localStorage så att sidan laddas in med korrekt inställningar.

### 5.5. Visa uppladdningsprocessen

Just nu så ser ju användaren inte hur det går för uppladdningen av våra medie-objekt. Vi vill förbättra detta först genom att:

Visa en laddningsikon under tiden som bilden laddas upp, ni kan [hämta inspiration här](https://www.w3schools.com/howto/howto_css_loader.asp).
... och sedan förbättra laddningsrutan ytterliggare genom att antingen:

- Visa antal % som är klar av uppladningen, alternativ
- Visa en progress-bar som illustrerar uppladdningsprocessen

Bra ställe att inspireras för att göra detta är [här på gituhub](https://github.com/englercj/jquery-ajax-progress)

## 5. Nu då?

Nu får ni i uppgift att vidare-utveckla skriptet efter eget intresse. Fråga gärna om ni undrar något!
