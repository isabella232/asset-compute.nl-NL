---
title: Begrijp over het uitbreiden [!DNL Asset Compute Service].
description: Wanneer en hoe te om  [!DNL Asset Compute Service] functionaliteit uit te breiden om de verwerking van douaneactiva te doen.
translation-type: tm+mt
source-git-commit: c392b8588929f7b13db13e42a3f17bbc4f68a376
workflow-type: tm+mt
source-wordcount: '265'
ht-degree: 0%

---


# Inleiding rekbaarheid {#introduction-to-extensibilty}

Veel vereisten voor vertoningen, zoals het omzetten in indelingen en het wijzigen van het formaat van afbeeldingen, worden opgelost door [Profielen verwerken in [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). De complexere bedrijfsvereisten kunnen een douane-gecreeerde oplossing vereisen die de behoeften van een organisatie aanpast. [!DNL Asset Compute Service] U kunt deze extensie uitbreiden door aangepaste toepassingen te maken die worden aangeroepen vanuit Procesprofielen in  [!DNL Experience Manager]. Deze douanetoepassingen behandelen aan [gesteunde gebruiksgevallen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met  [!DNL Experience Manager] als  [!DNL Cloud Service]een.

De aangepaste toepassingen hebben geen kop [Project Firefly](https://github.com/AdobeDocs/project-firefly)-toepassingen. Het uitbreiden van [!DNL Asset Compute Service] met douanetoepassingen wordt eenvoudig gemaakt door [Asset compute SDK](https://github.com/adobe/asset-compute-sdk) en het Project van de projectontwikkelaar het Levensvatbare hulpmiddel. Dit staat ontwikkelaars toe om zich op bedrijfslogica te concentreren. Het maken van aangepaste toepassingen is net zo eenvoudig als het maken van een Adobe I/O Runtime-handeling zonder normale serverless. Het is één JavaScript-functie Node.js. Het [basis voorbeeld van de douanetoepassing](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) illustreert het.

## Vereisten voor vereisten en voorzieningen {#prerequisites-and-provisioning}

Zorg ervoor dat u aan de volgende voorwaarden voldoet:

* De projectprogramma&#39;s zijn geïnstalleerd op uw computer.
* Een [!DNL Experience Cloud]-organisatie. Meer informatie [hier](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* De ervaringsorganisatie moet [!DNL Experience Manager] hebben als [!DNL Cloud Service] toegelaten.
* [!DNL Adobe Experience Cloud] organisatie maakt deel uit van het voorvertoningsprogramma voor  [!DNL Project Firefly] ontwikkelaars. Zie [hoe te om voor toegang toe te passen](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md).
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
