---
title: Begrijp over uitbreiden [!DNL Asset Compute Service]
description: Wanneer en hoe breidt u de [!DNL Asset Compute Service] functionaliteit voor aangepaste middelenverwerking.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 0%

---

# Inleiding tot uitbreidbaarheid {#introduction-to-extensibilty}

Aan veel vereisten voor vertoningen, zoals het omzetten in indelingen en het vergroten of verkleinen van afbeeldingen, wordt voldaan door [Profielen verwerken in [!DNL Experience Manager] als [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). De complexere bedrijfsvereisten kunnen een douane-gecreeerde oplossing vereisen die de behoeften van een organisatie aanpast. [!DNL Asset Compute Service] kan worden uitgebreid door aangepaste toepassingen te maken die worden aangeroepen vanuit Process Profiles in [!DNL Experience Manager]. Deze aangepaste toepassingen kunnen [ondersteunde gebruiksgevallen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met [!DNL Experience Manager] als [!DNL Cloud Service].

De aangepaste toepassingen zijn zonder kop [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) apps. Uitbreiden [!DNL Asset Compute Service] met aangepaste toepassingen eenvoudig wordt gemaakt via de [asset compute SDK](https://github.com/adobe/asset-compute-sdk) en Adobe Developer App Builder. Dit staat ontwikkelaars toe om zich op bedrijfslogica te concentreren. Aangepaste toepassingen maken is net zo eenvoudig als het maken van een normale server [!DNL Adobe I/O] Runtime, actie. Het is één JavaScript-functie Node.js. De [basis, aangepast toepassingsvoorbeeld](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) illustreert dit.

## Vereisten voor vereisten en voorzieningen {#prerequisites-and-provisioning}

Zorg ervoor dat u aan de volgende voorwaarden voldoet:

* De Adobe Developer App Builder-gereedschappen zijn op uw computer geïnstalleerd.
* An [!DNL Experience Cloud] organisatie. Meer informatie [hier](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* De Ervaringsorganisatie moet beschikken over [!DNL Experience Manager] als [!DNL Cloud Service] ingeschakeld.
* [!DNL Adobe Experience Cloud] organisatie maakt deel uit van de [!DNL Adobe Developer App Builder] voorvertoningsprogramma voor ontwikkelaars. Zie [toegangsaanvraag](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Verzeker een ontwikkelaarrol of beheerdertoestemmingen in de organisatie voor de ontwikkelaar.
* Zorg ervoor dat [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) is lokaal geïnstalleerd.

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
