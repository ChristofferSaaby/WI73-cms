# CMS bygget med Node.js

### 1. Del.


### Indledningsvis vil vi starte med at demonstrere et simpelt API

#### Et meget simpelt API der viser hvordan routes kan opbygges i node.js uden brug af 3. parts moduler.

Dvs. dette eksempel benytter sig udelukkende af moduler der er en del af node.js installationen. Derfor er der ikke behov for at installere 3. parts moduler. Jeg vil dog anbefale at _nodemon_ modulet installeres. 

API'et har 2 forskellige routes.
* `/kat    (der svarer 'Miav' på en GET request)`
* `/hund   (der svarer 'Vov-vov på en GET request)`

 
Selve serveren er defineret i `server.js` og benytter sig af node modulet `http` samt et modul, `router.js` som vi selv skal skrive koden til.

server.js
```javascript
const 
    http = require('http'),
    router = require('./router');

http.createServer(router).listen(3003);
console.log('Server er startet. Venter tålmodigt ved port 3003');
```

Vi opretter en server med createServer() metoden. Samtidig sendes modulet `router` "ned i maven" på serveren. Derefter konfigureres serveren til at benytte port 3003.

Koden i modulet `router` der sendes "ned i maven" på serveren vil blive eksekveret hver gang serveren modtager en _request_ fra en browser på port 3003.

Men lad os kigge på koden i modulet `router`. I dette modul skal vi importere modulet `url`. Også dette modul er en del af Node.js installationen og behøver derfor ikke at installeres. 

router.js
```javascript
const url = require('url');

// Definition af vores API og API-handlere
const routes = {
    '/kat' : function(res) {
                 res.writeHead(200, {'Content-type' : 'application/json'});
                 res.end('Miav');
             },
             
    '/hund' : function(res){
                 res.writeHead(200, {'Content-type' : 'application/json'});
                 res.end('Vov-vov');                
              }            
};

module.exports = function(req, res){
    // Vi henter pathname fra request objektet.
    var pathname = url.parse(req.url).pathname;
    
    // Hvis vi har en route der matcher '/kat' eller '/hund'
    // henter vi den ind i 'handler' variablen
    var handler = routes[pathname];
    
    // Checker om vi har noget i 'handler'...
    if(handler){
        handler(res);   // ...hvis vi har en funktion i 'handler' skal den eksekveres...
        return; // ...derefter afsluttes funktionen.
    }
    
    // Hvis vi er her er der ikke fundet en route (eller endpointhandler)
    // Det meddeler vi i form af en tekst
    res.writeHead(404, {'Content-type' : 'text/plain; charset=utf-8'});
    res.end('Route ' + pathname + ' findes ikke');                    
}
```

Kigger vi på vores endpointhandlere ser vi, at de to funktioner ligner hinanden ret meget. Eneste forskel er at den ene sender tekststrengen 'Miauv' til browseren, mens den anden sender 'Vov-vov'.

Derfor vil jeg ændre koden lidt. Først opretter jeg en ny mappe, `endpointhandlers`. I denne mappe vil jeg oprette to filer, `kat.js` og `hund.js`.

I det to filer placeres funktionerne for henholdsvis `/kat` og `/hund` routene.

endpointhandlers/kat.js
```javascript

module.exports = function(res) {
    res.writeHead(200, {'Content-type' : 'application/json'});
    res.end('Miav');
};
```
endpointhandlers/hund.js
```javascript

module.exports = function(res) {
    res.writeHead(200, {'Content-type' : 'application/json'});
    res.end('Vov-vov');
};
```

Vi har nu fået adskilt selve endpointhandlerne fra `routes.js`, men stadig har vi to handlere der har en del til fælles.
Næste skridt vil være at oprette endnu en fil som jeg vil kalde `helpers.js`. Denne fil er tænkt til at indeholde hjælpefunktioner. I dette eksempel vil der dog kun være en enkelt hjælpefunktion, eller rettere sagt metode, nemlig `.respond()`. Tanken er at denne metode skal indeholde den kode der er fælles for endpointhandlerne. Koden for `respond()` metoden skal se sådan ud:


helpers.js
```javacsript
exports.respond = function(res, message, status = 200){
    res.writeHead(status, {'Content-type' : 'application/json'});
    res.end(message);
};
```

Det man skal lægge mærke til her er det sidste argument `status`. Dette argument får som default værdien 200, hvilket er http-statuskoden for at alt er gået godt.


Denne metode kan også benyttes i vores `router` modul. Lad os ændre koden i dette modul så den ser sådan ud:

router.js
```javascript
const url = require('url');
const helpers = require('./helpers');

// Definition af vores API og API-handlere
const routes = {
    '/kat' : require('./endpointhandlers/kat'),
             
    '/hund' : require('./endpointhandlers/hund')            
};

module.exports = function(req, res){
    var pathname = url.parse(req.url).pathname;
    
    var handler = routes[pathname];
    if(handler){
        handler(res);
        return;
    }
    // Hvis vi er her er der ikke fundet en route (eller endpointhandler)
    helpers.respond(res, `Status: 404. Route '${pathname}' findes ikke` , 404);
}
```

I sin nuværende form kan vores API ikke skelne mellem request metoder som fx `GET`, `POST`, `PUT` eller `DELETE`

Heldigvis er det enkelt at udvide API'et til også at kunne dette.

Lad os kigge på en af endpointhandlerne, `kat`. Hvis vi i dette modul ændrer koden til at se sådan ud:

endpointhandlers/kat.js
```javascript

module.exports = {
    'GET' : function(res) {
                res.writeHead(200, {'Content-type' : 'application/json'});
                res.end('GET: Miav');
            },
    'POST' : function(res) {
                res.writeHead(200, {'Content-type' : 'application/json'});
                res.end('POST: Miav');
            }            
};
```

På tilsvarende vis skal `hund` ændres til at se sådan ud:

endpointhandlers/kat.js
```javascript

module.exports = {
    'GET' : function(res) {
                res.writeHead(200, {'Content-type' : 'application/json'});
                res.end('GET: Vov-vov');
            },
    'POST' : function(res) {
                res.writeHead(200, {'Content-type' : 'application/json'});
                res.end('POST: Vov-vov');
            }            
};
```

Men for at få den ændrede kode til at virke, er det nødvendigt at ændre i `router` modulet.

Koden i dette modul ændres så den ser sådan ud:

router.js
```javascript
const url = require('url');
const helpers = require('./helpers');

// Definition af vores API og API-handlere
const routes = {
    '/kat' : require('./endpointhandlers/kat'),
             
    '/hund' : require('./endpointhandlers/hund')            
};

module.exports = function(req, res){
    var pathname = url.parse(req.url).pathname;
    
    var handler = routes[pathname];
    if(handler){
        var action = handler[req.method];
        if(action){
            action(res);
            return;
        }
        helpers.respond(res, `Status: 404. Metode '${req.method}' ikke understøttet.`, 404);
        return;
    }
    // Hvis vi er her er der ikke fundet en route (eller endpointhandler)
    helpers.respond(res, `Status: 404. Route '${pathname}' findes ikke` , 404);
}
```

Vi har nu et simpelt API der er i stand til at svare på både `GET` og `POST` request. Det er nu op til dig at udvide dette API til også at kunne håndtere andre requesttyper fx. `PUT` og `DELETE`. Prøv også at tilføje en route, fx. `/and` der svarer `Rap-rap` på indkommende request.

### 2. del. Statiske filer

Vores API kan ikke levere statiske filer. Det får vi brug for, så vi skal til at lave de nødvendige tilføjelser til koden for at det kan lade sig gøre. 

Vi skal tilføje en hjælpefunktion til vores `helpers.js`. Funktionen skal kunne læse en fil fra filsystemet og sende indholdet i filen til en browser ved hjælp af `response` objektet. Derfor får vi brug for filsystem-modulet `fs`. 

Vi får også brug for at kunne detektere hvilken mimetype vi har med at gøre. Vi skal derfor oprette et objekt til at indholde definitionerne på de mimetyper vi ønsker at kunne håndtere.

Vi opretter et json-objekt i `helpers` filen. Objektet indeholder en række navn/værdi par, hvor navnene svarer til ekstensionen på de filer vi ønsker at håndtere, og værdierne svarer til mimetyperne.

Koden for mimetype-objektet.
```javascript
const fs = require('fs');  // Importer filsystem-modulet
const path = require('path');

// De foreløbige mimetypes. Vi kan tilføje flere når behovet opstår
const mimetypes = {
    '.html' :  'text/html',               // mimetype for html
    '.css'  :  'text/css',                // mimetype for css
    '.js'   :  'application/javascript',  // mimetype for javascript
    '.png'  :  'image/png'                // mimetype for png 
};

´´´

Vi får også brug for en funktion der kan læse filer fra serverens filsystem. Derfor får vi brug for at importere filsystem modulet `fs` der også er en del af node installationen.

Det gør vi i starten af filen `helpers.js` med denne linje `var fs = require('fs');` 

Koden til funktionen der skal læse en fil fra filsystemet sende den til browseren placeres også i `helpers.js` 

Her er koden
```javascript
exports.fileRespond = funktion(res, fileName){
    fs.readFile(fileName, function(err, fileContent){
        if(err){
            // Hvis der opstod en fejl, er vi her.
            exports.respond(res, `Filen '${fileName}' findes ikke.`, 404);
            return;
        }
        
        // Hvis vi er her, er der fundet en fil der kan læses. Indholdet skal så sendes til browseren,
        // men først skal vi detektere hvilken mimetype det handler om.
        
        var ext = path.extname(fileName); // hent fil-ekstension
        var mime = mimetypes[ext] // brug ekstensionen til at hente mimetype
        res.writeHead(200, {'Content-type' : mime });
        res.end(fileContent);
    }
}

```

Funktionen tager to parametre, response objektet og stien til filen der skal sendes.

Men inden vi ændrer mere i koden skal vi oprette en mappe til vores statiske filer. Jeg har valgt at kalde mappen for `public`. Inde i denne mappe vil jeg, til at begynde med, placere en html-fil samt tre undermapper `js`, `css` og `img`. Mapperne er tænkt til at indeholde henholdsvis javascript-, stylesheet- og billedfiler. Efter at have oprettet disse filer, ser mappestrukturen således ud:


```
├── endpointhandlers
│   ├── cat.js
│   └── dog.js
│
├── public
│   ├── css
│   │   └── style.css
│   │
│   ├── img
│   │   ├── logo.png
│   │   └── anonymprofil.png
│   │
│   ├── js
│   │   └── script.js
│   │
│   └── index.html
│
├── helpers.js
├── router.js
└── server.js
```


Fortsættes...
