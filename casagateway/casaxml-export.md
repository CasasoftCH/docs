---
title: SwissRETS export per HTTP
keywords: SwissRETS
summary: "Diese Schnittstelle ermöglicht das transferieren von Objektdaten eines Veröffentlichungs-Portal auf CASAGATEWAY (normalerweise publisher genant) mittels dem SwissRETS Standart. Dies wird vollständig über HTTP ermöglich und benötigt sommit keine FTP abhängigkeiten. Das XML kann beliebig von dem CASAGATEWAY API jederzeit abgeholt werden. CASAGATEWAY generiert und liefert per response dan direkt ein XML body. Ebenfalls wird CASAGATEWAY pokes/hooks ausführen sobald jegliche änderungen oder mutationen zu den Daten vorgenommen wurden. Diese werden per GET an einem vorkonfigurierten URI ausgelöst."
sidebar: casagateway_sidebar
permalink: /swissrets-export/
folder: casagateway
---

These Documentations have migrated. Diese Dokumentationen wurden verschoben.
------------------------

<a href="https://casasoftag.atlassian.net/wiki/spaces/DOCS/pages/146636828/SwissRETS+export+per+HTTP" class="btn btn-primary">https://casasoftag.atlassian.net/wiki/spaces/DOCS/pages/146636828/SwissRETS+export+per+HTTP</a>


## Wie erhalte ich die API und Private Keys?

Casasoft AG kann Ihnen diese keys gennerieren und liefern. Jeder Schlüssel dient zum authentifizieren einer `publisher` Schnittstelle die die Objekte und/oder Projekte von einem oder mehreren `provider`s ermöglicht. Meistens werden nur Schnitstellen von individuellen providers erstellt.

## Wie kann ich ein XML generieren und fetchen?

Der request läuft mittels gewöhnlichem `http` und einem `HMAC handshake` der durch ein `API Key` und einem `Private Key` generiert wird ab. Dieser wird von `CASAGATEWAY` zur verfügung gestellt. Bitte beachten Sie das Sie den `Private Key` nie Öffentlich zugänglich machen.

[SwissRETS](https://github.com/qualipool/swissrets)

Dieses XML kann dan beliebig mit Ihrer Infrastruktur beglichen werden, oder Sie nutzen das XML gleich als Speicher-Medium. Das `parsen` und `persistieren` dieser XML liegt in den Händen des Konsumenten. Allerdings bieten wir einige PHP Klassen an die das interpretieren von Werten vereinfacht. [github.com/casamodules/CasasoftStandards](https://github.com/CasasoftCH/casamodules/tree/master/src/CasasoftStandards)

### HMAC Formula

Hier wird abstrahiert wie der request hmac key generiert wird und an einem request string angehängt wird. Achten Sie das der `Private Key` niemals im schlussendlichem request angegeben wird. Anstelle wird ein `hmac key` und das öffentliche `api key` mitgeliefert. Die `+` Zeichen dienen nur zur visuellen trennung. 

```
checkstring = key1+value1+key2+value2+key3+value3+PRIVATE_KEY+TIMESTAMP

hmac = hash checkstring with sha256

xml_response = [GET] "http://casagateway.ch/rest/publisher-properties?hmac=abcd1234&apikey=abcd1234&timestamp=12345678&key1=value1&key2=value2&key3=value3"

```

### PHP Beispiel

Dieses Beispiel zeigt eine einfache PHP Funktion die mittels CURL diese Abfrage generiert und ausführt.

```php
<?php
    function getSwissRETS($apikey, $privateKey, $options){
        //specify the current UnixTimeStamp
        $timestamp = time();

        //specify your api key
        $apikey = $apikey;

        //specify your private key
        $privatekey = $privateKey;

        //sort the options alphabeticaly and combine it into the checkstring
        ksort($options);
        $checkstring = '';
        foreach ($options as $key => $value) {
            $checkstring .= $key . $value;
        }
        
        //add private key at end of the checkstring
        $checkstring .= $privatekey;

        //add the timestamp at the end of the checkstring
        $checkstring .= $timestamp;

        //hash it to specify the hmac
        $hmac = hash('sha256', $checkstring, false);

        //combine the query (DONT INCLUDE THE PRIVATE KEY!!!)
        $query = array(
            'hmac' => $hmac,
            'apikey' => $apikey,
            'timestamp' => $timestamp
        ) + $options;

        //build url
        $url = 'https://casagateway.ch/rest/publisher-properties?' . http_build_query($query, '', '&');
        $url = urldecode($url);
		
		//execute request with CURL
        $response = false;
        try {
            $ch = curl_init(); 
            curl_setopt($ch, CURLOPT_URL, $url); 
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1); 
            curl_setopt($ch, CURLOPT_IPRESOLVE, CURL_IPRESOLVE_V4);
            $response = curl_exec($ch); 
            $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
            if($httpCode == 404) {
                $response = $httpCode;
            }
            curl_close($ch); 
        } catch (Exception $e) {
            $response =  $e->getMessage() ;
        }

        return $response;
    }

    $options = array(
        'format' => 'casa-xml'
        //further options might be added later
    );
    $xml_string = getSwissRETS('API_KEY_GOES_HERE', 'PRIVATE_KEY_GOES_HERE', $options);
    
    //use the $xml_string as you see fit from here on
    
?>
```

## Welche optionen kann ich nutzten

*company*: company_slug um nach Firmen zu filtern. Dies ist sinvol für grössere Portal/Publishers wo die imports Unterteilen möchten.

## Welche Firmen wollen für mich (publisher) Objekte versenden und wie erhalte ich diesen company_slug

Mit dem gleichen API Schlüssel und Verfahren kann unter dem Link 'https://casagateway.ch/rest/publisher-companies' eine JSON Liste aller Firmen bezogen werden die auf diese Platform Objekte zum veröffentlichen bereit haben.

## Wie weiss ich wenn sich etwas aktualisiert hat?

`CASAGATEWAY` wird bei jeglichen Änderungen einen `GET poke/hook` auf eine vordefinierte URI ausführen. Diese kann z.B. wie folgt aussehen:

```
http://your-new-website.ch/casagateway-change-alert?timestamp=1234567890&a_super_secret_key=12k34k5j2hkj34k2j3bkj2b3kj3
```

Hier könnte ein Script sein welches ein Import bei der nächsten Gelegenheit ausführt. Allerdings sollten die imports nur asynchron angestossen werden oder zukünftig vermerkt werden. Während diesem Aufrufs sollte kein Import ausgeführt werden da der CASAGATEWAY denn request nach einer kurzen Zeit abbrechen wird.

## Glossar

**publisher**: Eine Veröffentlichungsinstanz vom CASAGATEWAY. Normalerweise assoziiert mit Portal- oder Webseitenbereiche.

**provider**: Eine Import- oder Erfassungsschnittstelle. Normalerweise sind das Makler oder Maklergruppen mit einer Immobilien-Software die an die CASAGATEWAY liefern.

<a target="_blank" class="noCrossRef" href="/pdf/mydoc.pdf"><button type="button" class="btn btn-default" aria-label="Left Align"><span class="glyphicon glyphicon-download-alt" aria-hidden="true"></span> PDF Download</button></a>
