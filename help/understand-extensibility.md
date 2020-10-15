---
title: Begrijp over uitbreiden [!DNL Asset Compute Service].
description: Wanneer en hoe u [!DNL Asset Compute Service] de functionaliteit kunt uitbreiden voor aangepaste verwerking van elementen.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '275'
ht-degree: 0%

---


# Inleiding tot uitbreidbaarheid {#introduction-to-extensibilty}

Veel vereisten voor vertoningen, zoals het omzetten in indelingen en het vergroten of verkleinen van afbeeldingen, worden behandeld door Profielen [verwerken [!DNL Experience Manager] als een Cloud Service](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/asset-microservices-overview.html). De complexere bedrijfsvereisten kunnen een douane-gecreeerde oplossing vereisen die de behoeften van een organisatie aanpast. [!DNL Asset Compute Service] U kunt deze extensie uitbreiden door aangepaste toepassingen te maken die worden aangeroepen vanuit Procesprofielen in [!DNL Experience Manager]. Deze aangepaste toepassingen zijn gebaseerd op de [ondersteunde gebruiksgevallen](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met [!DNL Experience Manager] als Cloud Service.

De aangepaste toepassingen zijn toepassingen zonder hoofd voor [probleemloze](https://github.com/AdobeDocs/project-firefly) projecttoepassingen. Het uitbreiden [!DNL Asset Compute Service] met douanetoepassingen wordt eenvoudig gemaakt door de [Activa Compute SDK](https://github.com/adobe/asset-compute-sdk) en het de ontwikkelaars van het Project Frefly hulpmiddelen. Dit staat ontwikkelaars toe om zich op bedrijfslogica te concentreren. Het maken van aangepaste toepassingen is net zo eenvoudig als het maken van een Adobe I/O Runtime-handeling zonder normale serverless. Het is één JavaScript-functie Node.js. Het [eenvoudige aangepaste toepassingsvoorbeeld](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) illustreert dit.

## Vereisten voor vereisten en voorzieningen {#prerequisites-and-provisioning}

Zorg ervoor dat u aan de volgende voorwaarden voldoet:

* De projectprogramma&#39;s zijn geïnstalleerd op uw computer.
* Een [!DNL Experience Cloud] organisatie. Meer informatie [hier](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* De Experience Organisation moet [!DNL Experience Manager] als Cloud Service ingeschakeld zijn.
* [!DNL Adobe Experience Cloud] organisatie maakt deel uit van het voorvertoningsprogramma voor [!DNL Project Firefly] ontwikkelaars. Zie [hoe u toegang](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md)kunt aanvragen.
* Verzeker een ontwikkelaarrol of beheerdertoestemmingen in de organisatie voor de ontwikkelaar.
* Zorg ervoor dat [Adobe I/O CLI](https://github.com/adobe/aio-cli) lokaal is geïnstalleerd.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
