---
title: Stel de ontwikkelomgeving in die vereist is voor [!DNL Asset Compute Service]
description: Ontwikkelomgeving ingesteld voor [!DNL Asset Compute Service] om aangepaste code te maken en te testen.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 9404ffcc66a3b6ba206155d1b1a5c16a43e22a39
workflow-type: tm+mt
source-wordcount: '370'
ht-degree: 0%

---

# Een ontwikkelomgeving instellen {#create-dev-environment}

Als u een installatie wilt maken waarmee u zich kunt ontwikkelen voor [!DNL Asset Compute Service], volgt u deze vereisten en instructies.

1. [Verkrijg toegang en ](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) geloofsbrieven voor  [!DNL Project Firefly].

1. [Stel de lokale ](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) omgeving en de vereiste gereedschappen in.

1. Hier volgen nog enkele gereedschappen die u helpen probleemloos aan de slag te gaan met het ontwikkelen van:

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (v10 tot v12 LTS, oneven versies worden niet aanbevolen) en  [NPM](https://www.npmjs.com). Gebruiker van OSX HomeBrew kan `brew install node` doen om beide te installeren. Anders, download het van [deJS downloadpagina](https://nodejs.org/en/).
   * winde die voor NodeJS goed is, adviseren wij [de Code van Visual Studio (de Code van VS)](https://code.visualstudio.com) aangezien het gesteunde winde voor debugger is. U kunt om het even welke andere winde als coderedacteur gebruiken, maar geavanceerd gebruik (b.v. debugger) wordt nog niet gesteund.
   * [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`) - installeren met  `npm install -g @adobe/aio-cli@7.1.0`.

1. Zorg ervoor dat u voldoet aan de [voorwaarden](/help/understand-extensibility.md#prerequisites-and-provisioning).

>[!NOTE]
>
>Gebruik momenteel [!DNL Adobe I/O] CLI v7.1.0 van en gebruik [!DNL Adobe I/O] CLI v8 niet.

## Een Firefly-project instellen {#create-firefly-project}

1. Zorgen voor de rol van systeembeheerder of ontwikkelaar in de organisatie [!DNL Experience Cloud]. Dit wordt opstelling door een systeembeheerder in [Admin Console](https://adminconsole.adobe.com/overview).

1. Log op [Adobe Developer Console](https://console.adobe.io/). Zorg ervoor dat u deel uitmaakt van dezelfde [!DNL Experience Cloud]-organisatie als de [!DNL Experience Manager] als een [!DNL Cloud Service]-integratie. Zie [Consoledocumentatie](https://www.adobe.io/apis/experienceplatform/console/docs.html) voor meer informatie over Adobe Developer Console.

1. [Maak een Firefly-project](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Klik **[!UICONTROL Nieuw project maken]** > **[!UICONTROL Project van sjabloon]**. Selecteer Firefly. Het leidt tot een nieuw Vuurwerk Project met twee werkruimten: `Production` en `Stage`. Voeg desgewenst extra werkruimten toe, bijvoorbeeld `Development`.

1. Selecteer in het project Firefly een werkruimte en abonneer de services die nodig zijn voor de Asset compute. Klik **Toevoegen aan project** > **API** en voeg `Asset Compute`-, `IO Events`- en `IO Events Management`-services toe. Wanneer de eerste API wordt toegevoegd, wordt gevraagd om een persoonlijke sleutel te maken. Sla deze gegevens op uw computer op omdat u deze sleutel nodig hebt om uw aangepaste toepassing met het ontwikkelaarsgereedschap te testen.

## Volgende stap {#next-step}

Nu uw milieu opstelling is, bent u klaar om [een douanetoepassing te creëren](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
