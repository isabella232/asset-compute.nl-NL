---
title: Testen en fouten opsporen [!DNL Asset Compute Service] aangepaste toepassing
description: Testen en fouten opsporen [!DNL Asset Compute Service] aangepaste toepassing.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: ebc0d717b3f6fc4518f4a79cd44ebe8fdcf9ec6a
workflow-type: tm+mt
source-wordcount: '811'
ht-degree: 0%

---

# Een aangepaste toepassing testen en fouten opsporen {#test-debug-custom-worker}

## Eenheidstests uitvoeren voor een aangepaste toepassing {#test-custom-worker}

Installeer [Docker Desktop](https://www.docker.com/get-started) op uw computer. Als u een aangepaste worker wilt testen, voert u de volgende opdracht uit in de hoofdmap van de toepassing:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Dit stelt een kader van de douanetest van de eenheid voor de acties van de Asset compute toepassing in het project in werking zoals hieronder beschreven. Het wordt omhoog vastgemaakt door een configuratie in het `package.json` dossier. Het is ook mogelijk om JavaScript eenheidstests zoals Jest te hebben. `aio app test` voert beide uit.

De [air-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency)-plug-in is ingesloten als ontwikkelingsafhankelijkheid in de aangepaste toepassing, zodat deze niet hoeft te worden geïnstalleerd op build-/testsystemen.

### Testframework toepassingseenheid {#unit-test-framework}

Met het testframework voor de eenheid van de Asset compute-toepassing kunt u toepassingen testen zonder code te schrijven. Het is afhankelijk van het principe van de bron naar het weergavebestand van toepassingen. Er moet een bepaalde bestands- en mapstructuur worden ingesteld om testcase te definiëren met testbronbestanden, optionele parameters, verwachte uitvoeringen en aangepaste validatiescripts. Standaard worden de uitvoeringen vergeleken voor de gelijkheid van bytes. Bovendien kunnen externe HTTP-services eenvoudig worden gecontroleerd met behulp van eenvoudige JSON-bestanden.

### Tests toevoegen {#add-tests}

Tests worden verwacht binnen de `test` omslag op het wortelniveau van het [!DNL Adobe I/O] project. De testgevallen voor elke toepassing moeten zich in het pad `test/asset-compute/<worker-name>` bevinden, met één map voor elk testgeval:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Bekijk [voorbeeld douanetoepassingen](https://github.com/adobe/asset-compute-example-workers/) voor sommige voorbeelden. Hieronder vindt u een gedetailleerde referentie.

### Uitvoer {#test-output} testen

De gedetailleerde testuitvoer, inclusief de logboeken van de aangepaste toepassing, wordt beschikbaar gesteld in de map `build` in de hoofdmap van de Firefly-app, zoals wordt getoond in de `aio app test`-uitvoer.

### Externe services koppelen {#mock-external-services}

Het is mogelijk om externe de dienstvraag in uw acties te controleren door `mock-<HOST_NAME>.json` dossiers in uw testgevallen te bepalen, waar HOST_NAME de gastheer is u zou willen controleren. Een geval van het voorbeeldgebruik is een toepassing die een afzonderlijke vraag aan S3 maakt. De nieuwe teststructuur ziet er als volgt uit:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

Het mock-bestand is een http-reactie met JSON-indeling. Zie [deze documentatie](https://www.mock-server.com/mock_server/creating_expectations.html) voor meer informatie. Als er veelvoudige gastheernamen aan mock zijn, bepaal veelvoudige `mock-<mocked-host>.json` dossiers. Hieronder ziet u een voorbeeldmodelbestand voor `google.com` met de naam `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

Het voorbeeld `worker-animal-pictures` bevat een [mock-bestand](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) voor de Wikimedia-service waarmee het communiceert.

#### Bestanden delen in testgevallen {#share-files-across-test-cases}

Het wordt aangeraden relatieve symmetrieën te gebruiken als u scripts `file.*`, `params.json` of `validate` over meerdere tests deelt. Ze worden ondersteund met git. Geef uw gedeelde bestanden een unieke naam, omdat u mogelijk andere bestanden hebt. In het onderstaande voorbeeld mengen en vergelijken de tests een paar gedeelde bestanden en hun eigen bestanden:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Verwachte fouten testen {#test-unexpected-errors}

Fouttestgevallen mogen geen verwacht `rendition.*`-bestand bevatten en moeten de verwachte `errorReason` in het `params.json`-bestand definiëren.

>[!NOTE]
>
>Als een testcase geen verwacht `rendition.*`-bestand bevat en de verwachte `errorReason` in het `params.json`-bestand niet definieert, wordt aangenomen dat dit een foutmelding is met een `errorReason`.

Fout bij testen hoofdletterstructuur:

```json
<error_test_case>/
    file.jpg
    params.json
```

Parameterbestand met reden van fout:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Zie volledige lijst en beschrijving van [Asset compute foutenredenen](https://github.com/adobe/asset-compute-commons#error-reasons).

## Fouten opsporen in een aangepaste toepassing {#debug-custom-worker}

De volgende stappen tonen hoe u uw douanetoepassing kunt zuiveren gebruikend de Code van Visual Studio. Het staat voor het zien van levende logboeken, raakbreekpunten en stap door code evenals het levende opnieuw laden van lokale codeveranderingen na elke activering toe.

Veel van deze stappen worden gewoonlijk geautomatiseerd door `aio` uit de doos, zie sectie het Zuiveren van de Toepassing in [Firefly documentatie](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Tot nu toe bevatten de onderstaande stappen een tijdelijke oplossing.

1. Installeer de nieuwste [wskdebug](https://github.com/apache/openwhisk-wskdebug) van GitHub en de optionele [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Voeg aan uw gebruikersinstellingen JSON-bestand toe. Het blijft het oude debugger van de Code van VS gebruiken, nieuw heeft [sommige kwesties](https://github.com/apache/openwhisk-wskdebug/issues/74) met wskdebug: `"debug.javascript.usePreview": false`.
1. Sluit alle instanties van toepassingen die zijn geopend via `aio app run`.
1. Implementeer de nieuwste code met `aio app deploy`.
1. Alleen de Asset compute Ontwikkelen uitvoeren met `aio asset-compute devtool`. Houd het open.
1. In de Redacteur van de Code van VS, voeg de volgende zuivert configuratie aan uw `launch.json` toe:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Haal `ACTION NAME` uit de uitvoer van `aio app deploy`.

1. Selecteer `wskdebug worker` van de looppas/zuivert configuratie en druk het playbackpictogram. Wacht tot het om te beginnen tot het **[!UICONTROL Klaar voor activering]** in **[!UICONTROL Debug Console]** venster toont.

1. Klik **[!UICONTROL run]** in Devtool. U kunt de acties zien die in de redacteur van de Code van VS worden uitgevoerd en het logboeken begint te tonen.

1. Stel een onderbrekingspunt in de code in, voer het opnieuw uit en druk op.

Eventuele codewijzigingen worden in real-time geladen en worden van kracht zodra de volgende activering plaatsvindt.

>[!NOTE]
>
>Er zijn twee activeringen voor elke aanvraag in aangepaste toepassingen. Het eerste verzoek is een webactie die zichzelf asynchroon aanroept in de SDK-code. De tweede activering is de activering die raakt aan uw code.
