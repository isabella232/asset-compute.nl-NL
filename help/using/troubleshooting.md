---
title: Problemen oplossen [!DNL Asset Compute Service]
description: Aangepaste toepassingen oplossen en fouten opsporen met [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# Problemen oplossen {#troubleshoot}

Enkele algemene tips voor het oplossen van problemen die u kunnen helpen problemen op te lossen met de service Asset compute zijn:

* Zorg ervoor dat de JavaScript-toepassing niet vastloopt bij het opstarten. Dergelijke neerstortingen zijn gewoonlijk verwant aan een ontbrekende bibliotheek of een gebiedsdeel.
* Zorg ervoor dat in de toepassing wordt verwezen naar alle afhankelijkheden die moeten worden geïnstalleerd `package.json` bestand.
* Verzeker om het even welke fouten die uit schoonmaakbeurt bij mislukking kunnen voortkomen niet hun eigen fouten produceren die het originele probleem verbergen.

* Wanneer u het ontwikkelaarsgereedschap voor het eerst start met een nieuwe [!DNL Asset Compute Service] het eerste verwerkingsverzoek mislukt als het Asset compute Events Journal niet volledig is ingesteld. Wacht enige tijd op het dagboek aan opstelling alvorens een ander verzoek te verzenden.
* Als er fouten optreden bij het verzenden van Asset compute `/register` of `/process` aanvragen, zorg ervoor dat alle noodzakelijke API&#39;s worden toegevoegd aan de [!DNL Adobe I/O] Project en werkruimte: Asset compute, [!DNL Adobe I/O] gebeurtenissen, [!DNL Adobe I/O] Events Management en [!DNL Adobe I/O] Runtime.

## Aanmeldingsproblemen via [!DNL Adobe I/O] CLI {#login-via-aio-cli}

Als u problemen hebt met het aanmelden bij de [!DNL Adobe Developer Console] [via de [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)Voeg vervolgens handmatig de vereiste gegevens toe voor het ontwikkelen, testen en implementeren van uw aangepaste toepassing:

1. Ga naar uw Adobe Developer App Builder-project en -werkruimte op het tabblad [Adobe Developer Console](https://console.adobe.io/) en drukken **[!UICONTROL Downloaden]** in de rechterbovenhoek. Open dit bestand en sla deze JSON op een veilige plaats op uw computer op.

1. Navigeer naar het ENV-bestand in de Adobe Developer App Builder-toepassing.

1. Voeg de [!DNL Adobe I/O] Runtimereferenties. Krijg de [!DNL Adobe I/O] A Runtime geloofsbrieven van gedownloade JSON. De referenties zijn onder `project.workspace.services.runtime`. Voeg de [!DNL Adobe I/O] Runtimegegevens in het dialoogvenster `AIO_runtime_XXX` variabelen:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Voeg het absolute pad toe aan de gedownloade JSON in Stap 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. De rest van de [vereiste referenties](develop-custom-application.md) nodig is voor het ontwikkelaarsgereedschap.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
