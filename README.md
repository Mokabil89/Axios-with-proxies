# Einrichten eines Proxys in Axios

[![Promo](https://github.com/luminati-io/Rotating-Residential-Proxies/blob/main/50%25%20off%20promo.png)](https://brightdata.de/proxy-types/residential-proxies) 

Dieser Axios-Proxy-Leitfaden behandelt die folgenden Themen:

1. [Axios und Proxies](#axios-und-proxies)
2. [Verwendung eines Proxys in Axios](#using-a-proxy-in-axios)
   - [HTTP/HTTPS Proxies](#httphttps-proxies)
   - [SOCKS Proxies](#socks-proxies)
3. [Axios Proxy: Fortgeschrittene Anwendungsfälle](#axios-proxy-advanced-use-cases)
   - [Einen Proxy global festlegen](#setting-a-proxy-globally)
   - [Umgang mit Proxy-Authentifizierung in Axios](#dealing-with-proxy-authentication-in-axios)
   - [Proxies über Umgebungsvariablen festlegen](#setting-proxies-via-environment-variables)
   - [Rotierende Proxies implementieren](#implementing-rotating-proxies)
4. [Fazit](#conclusion)

## Axios und Proxies

[Axios](https://axios-http.com/) ist einer der am weitesten verbreiteten HTTP-Clients im JavaScript-Ökosystem. Es bietet eine Promise-basierte, einfach zu verwendende, intuitive API zum Ausführen von HTTP-Anfragen und zum Umgang mit benutzerdefinierten Headern, Konfigurationen und Cookies.

Indem Sie Ihre Axios-Anfragen über einen Proxy leiten, können Sie Ihre IP-Adresse maskieren, wodurch es für den Zielserver schwieriger wird, Sie zu identifizieren und zu blockieren.

## Using a Proxy in Axios

Richten wir einen HTTP-, HTTPS- oder SOCKS-Proxy in Axios ein. Installieren Sie das `axios` npm-Paket:

```bash
npm install axios
```

In Node.js unterstützt Axios HTTP- und HTTPS-Proxies nativ über die [`proxy`](https://github.com/axios/axios#request-config)-Konfiguration. Wenn Sie also HTTP/HTTPS Proxies mit Axios in einer Node.js-Anwendung verwenden möchten, gibt es hier nichts Weiteres zu tun.

Wenn Sie stattdessen einen Nicht-HTTP/S-Proxy verwenden möchten, müssen Sie auf das Projekt [Proxy Agents](https://github.com/TooTallNate/proxy-agents) zurückgreifen. Dieses stellt `http.Agent`-Implementierungen bereit, um Axios mit Proxies in unterschiedlichen Protokollen zu integrieren:

- HTTP und HTTPS Proxies: [`https-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/https-proxy-agent)
- SOCKS, SOCKS5 und SOCKS4: [`socks-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/socks-proxy-agent)
- PAC-\*: [`pac-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/pac-proxy-agent)

### HTTP/HTTPS Proxies

Die URL Ihres HTTP/HTTPS Proxys sollte wie folgt aussehen:

```
"<PROXY_PROTOCOL>://<PROXY_HOST>:<PROXY_PORT>"
```

- `<PROXY_PROTOCOL>` ist „http“ für HTTP Proxies und „https“ für HTTPS Proxies.
- `<PROXY_HOST>` ist in der Regel eine rohe IP.
- `<PROXY_PORT>` ist der Port, auf dem der Proxy-Server lauscht.

Angenommen, dies ist die URL Ihres HTTP Proxys:

```
"http://47.88.62.42:80"
```

Sie können diesen Proxy in Axios wie folgt festlegen:

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "47.88.62.42",

        port: 80

    }

})
```

Um zu überprüfen, dass der oben genannte Axios-Proxy-Ansatz funktioniert, rufen Sie die URL eines kostenlosen HTTP- oder HTTPS-Proxy-Servers ab. Probieren Sie dieses Beispiel:

```
Protocol: HTTP; IP Address: 52.117.157.155; Port: 8002
```

Die vollständige Proxy-URL lautet `http://52.117.157.155:8002`.

Um zu überprüfen, dass der Proxy wie erwartet funktioniert, zielen Sie auf den Endpunkt [/ip](https://httpbin.io/ip) aus dem HTTPBin-Projekt. Diese öffentliche API gibt die IP der eingehenden Anfrage zurück, sodass sie die IP des Proxy-Servers zurückgeben sollte.

Der Ausschnitt des Node.js-Skripts lautet:

```js
import axios from "axios"

async function testProxy() {

    // perform the desired request through the HTTP proxy

const response = await axios.get("https://httpbin.io/ip", {
    proxy: {  
        protocol: "http",  
        host: "52.117.157.155",
        port: 8002
    }
});

    // print the result

    console.log(response.data)

}

testProxy()
```

Führen Sie das Skript aus, und es sollte Folgendes protokollieren:

```js
{ "origin": "52.117.157.155" }
```

> **Warnung**:\
> Sie erhalten nicht dasselbe Ergebnis, wenn Sie das Skript ausführen, da kostenlose Proxy-Dienste unzuverlässig, langsam, fehleranfällig, datengierig und kurzlebig sind.

### SOCKS Proxies

Wenn Sie versuchen, die Zeichenkette „socks“ im Feld `protocol` des `proxy`-Konfigurationsobjekts festzulegen, erhalten Sie den folgenden Fehler:

```js
AssertionError [ERR_ASSERTION]: protocol mismatch

  // ...

 {

  generatedMessage: false,

  code: 'ERR_ASSERTION',

  actual: 'dada:',

  expected: 'http:',

  operator: '=='

}
```

Das liegt daran, dass Axios SOCKS Proxies nicht nativ unterstützt. Fügen Sie die npm-Bibliothek `socks-proxy-agent` zu den Abhängigkeiten Ihres Projekts hinzu:

```bash
npm install socks-proxy-agent
```

Dieses Paket ermöglicht Ihnen, beim Ausführen von HTTP- oder HTTPS-Anfragen in Axios eine Verbindung zu einem SOCKS-Proxy-Server herzustellen.

Importieren Sie dann die SOCKS-Proxy-Agent-Implementierung aus der Bibliothek:

```js
const SocksProxyAgent = require("socks-proxy-agent")
```

Oder wenn Sie ESM verwenden:

```js
import { SocksProxyAgent } from "socks-proxy-agent"
```

Angenommen, dies ist die URL Ihres SOCKS Proxys:

```
"socks://183.88.74.73:4153"
```

> **Hinweis**:\
> Das Proxy-Protokoll kann entweder „socks“, „socks5“ oder „socks4“ sein.

Speichern Sie sie in einer Variable und übergeben Sie sie an den `SocksProxyAgent`-Konstruktor:

```js
const proxyURL = "socks://183.88.74.73:4153"

const proxyAgent = new SocksProxyAgent(proxyURL)
```

`SocksProxyAgent()` initialisiert eine `http.Agent`-Instanz, um HTTP/HTTPS-Anfragen über die Proxy-URL auszuführen.

Sie können nun einen SOCKS Proxy mit Axios wie folgt verwenden:

```js
axios.get(targetURL, { 

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

`httpAgent` und `httpsAgent` definieren den benutzerdefinierten Agent, der beim Ausführen von HTTP- bzw. HTTPS-Anfragen verwendet werden soll. Mit anderen Worten: Die von Axios ausgeführte HTTP- oder HTTPS-Anfrage wird über den angegebenen SOCKS Proxy geleitet. Auf ähnliche Weise können Sie das npm-Paket [`https-proxy-agent`](https://www.npmjs.com/package/https-proxy-agent) als alternative Möglichkeit verwenden, HTTP/HTTPS Proxies in Axios festzulegen.

Fügen wir alles zusammen:

```js
import axios from "axios"

import { SocksProxyAgent } from "socks-proxy-agent"

async function testProxy() {

    // replace with the URL of your SOCKS proxy 

    const proxyURL = "socks://183.88.74.73:4153"

    // define the HTTP/HTTPS proxy agent

    const proxyAgent = new SocksProxyAgent(proxyURL)

    // perform the request via the SOCKS proxy

    const response = await axios.get("https://httpbin.io/ip", { 

        httpAgent: proxyAgent,     

        httpsAgent: proxyAgent 

    })

    // print the result

    console.log(response.data) // { "origin": "183.88.74.73" }

}

testProxy()
```

Folgen Sie dem Link für weitere Beispiele dazu, [wie Sie einen SOCKS Proxy in Axios konfigurieren](https://writech.run/blog/how-to-use-a-socks-proxy-in-axios-6c0355a2e013/).

## Axios Proxy: Fortgeschrittene Anwendungsfälle

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.de/proxy-types/residential-proxies) 

### Einen Proxy global festlegen

Sie können einen Proxy global festlegen, indem Sie ihn direkt in einer Axios-Instanz angeben:

```js
const axiosInstance = axios.create({

    proxy: { 

        protocol: "<PROXY_PROTOCOL>", 

        host: "<PROXY_HOST>",

        port: "<PROXY_PORT>" 

    },

    // other configs...

})
```

Oder wenn Sie Proxy Agents verwenden:

```js
// proxy Agent definition ...

const axiosInstance = axios.create({

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

So konfigurieren Sie Axios, um global einen SOCKS Proxy zu verwenden:

```js
import { SocksProxyAgent } from "socks-proxy-agent";

const proxyURL = "socks://183.88.74.73:4153";

// Create a SOCKS proxy agent
const proxyAgent = new SocksProxyAgent(proxyURL);

// Create an Axios instance with the SOCKS proxy
const axiosInstance = axios.create({
    httpAgent: proxyAgent, // for HTTP requests
    httpsAgent: proxyAgent, // for HTTPS requests
    // other configs...
});
```

Alle Anfragen, die mit `axiosInstance` ausgeführt werden, laufen nun automatisch über den angegebenen Proxy.

### Umgang mit Proxy-Authentifizierung in Axios

Um ausschließlich zahlenden Nutzern Zugriff auf Premium-Proxies zu gewähren, schützen Proxy-Anbieter diese mit Authentifizierung. Der Versuch, sich ohne Benutzername und Passwort mit einem authentifizierten Proxy zu verbinden, führt zu einem Fehler [407 Proxy Authentication Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/407).

Insbesondere lautet die Syntax der URL eines authentifizierten Proxys:

```
[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]
```

Beispielsweise könnte eine URL aus der Praxis für die Verbindung zu einem authentifizierten Proxy wie folgt aussehen:

```
http://admin:lK4w90MEe45YIkOpk@156.127.0.192:8391
```

In diesem Fall wären die Felder der Proxy-URL:

- `<PROTOCOL>:HTTP`
- `<HOST>:156.127.0.192`
- `<PORT>:8391`
- `<USERNAME>:admin`
- `<PASSWORD>:lK4w90MEe45YIkOpk`

Um Proxy-Authentifizierung in Axios zu handhaben, geben Sie den Benutzernamen und das Passwort im `authfield` von `proxy` an:

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "156.127.0.192",

        port: "8381",

        auth: {

            username: "admin",

            password: "lK4w90MEe45YIkOpk"

        }

    }

})
```

Wenn Sie stattdessen Proxy Agents verwenden, haben Sie zwei Möglichkeiten, die Authentifizierung zu handhaben:

1. Fügen Sie die Zugangsdaten direkt in die Proxy-URL ein:

```js
var proxyAgent = new SocksProxyAgent("http://admin:[email protected]:8391")
```

2. Legen Sie die Optionen `username` und `password` in einem [URL](https://nodejs.org/api/url.html)-Objekt fest:

```js
const proxyOpts = new URL("http://156.127.0.192:8391")

proxyOpts.username = "admin"

proxyOpts.password = "lK4w90MEe45YIkOpk"

const proxyAgent = new SocksProxyAgent(proxyOpts)
```

Die gleichen Ansätze funktionieren auch mit HttpsProxyAgent.

### Proxies über Umgebungsvariablen festlegen

Eine weitere Möglichkeit, einen Proxy global in Axios zu konfigurieren, besteht darin, die folgenden Umgebungsvariablen zu setzen:

- `HTTP_PROXY`: Die URL des Proxy-Servers, der für HTTP-Anfragen verwendet werden soll.
- `HTTPS_PROXY`: Die URL des Proxy-Servers, der für HTTPS-Anfragen verwendet werden soll.

Unter Linux oder macOS können Sie sie wie folgt setzen:

```bash
export HTTP_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"

export HTTPS_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"
```

Wenn Axios diese Umgebungsvariablen erkennt, liest es daraus die Proxy-Einstellungen, einschließlich der Zugangsdaten für die Authentifizierung. Setzen Sie das Feld `proxy` auf `false`, damit Axios diese Umgebungsvariablen ignoriert. Beachten Sie, dass Sie auch eine `NO_PROXY`-Umgebungsvariable als kommagetrennte Liste von Domains definieren können, die nicht über einen Proxy geleitet werden sollen.

Beachten Sie, dass derselbe Mechanismus auch funktioniert, wenn Sie [Proxies in cURL verwenden](https://brightdata.de/blog/proxy-101/curl-with-proxies).

### Rotierende Proxies implementieren

Um zu verhindern, dass die Zielseite die IP-Adresse Ihres Proxys blockiert, stellen Sie sicher, dass jede Anfrage, die Sie ausführen, von einem anderen Proxy-Server stammt:

1. Definieren Sie eine Liste von Objekten, die jeweils die Informationen enthalten, um sich mit einem anderen Proxy zu verbinden.
2. Wählen Sie vor jeder Anfrage zufällig ein Proxy-Objekt aus.
3. Konfigurieren Sie den ausgewählten Proxy in Axios.

Der oben skizzierte Ansatz setzt voraus, dass Sie Zugriff auf einen Pool zuverlässiger Proxy-Server haben, wie z. B. die [Rotating Proxies](https://brightdata.de/solutions/rotating-proxies), die Bright Data anbietet.

## Conclusion

Bright Data kontrolliert die besten Proxy-Server der Welt und bedient Fortune-500-Unternehmen sowie über 20.000 Kunden. Sein weltweites Proxy-Netzwerk umfasst:

*   [Datacenter proxies](https://brightdata.de/proxy-types/datacenter-proxies) – Über 770.000 Rechenzentrums-IPs.
*   [Residential proxies](https://brightdata.de/proxy-types/residential-proxies) – Über 72M Residential IPs in mehr als 195 Ländern.
*   [ISP proxies](https://brightdata.de/proxy-types/isp-proxies) – Über 700.000 ISP-IPs.
*   [Mobile proxies](https://brightdata.de/proxy-types/mobile-proxies) – Über 7M mobile IPs.

[Erstellen Sie noch heute ein kostenloses Bright Data-Konto](https://brightdata.de/#popup-155639), um unsere Proxy-Server auszuprobieren.