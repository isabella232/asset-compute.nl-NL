---
title: Inleiding tot de [!DNL Asset Compute Service]
description: "[!DNL Asset Compute Service] is een service voor de verwerking van eigen middelen in de cloud die de complexiteit vermindert en de schaalbaarheid verbetert."
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 0%

---

# Overzicht van [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] is de scalable en verlengbare dienst van [!DNL Adobe Experience Cloud] om digitale elementen te verwerken. Het kan beeld, video, document, en andere dossierformaten in verschillende vertoningen met inbegrip van duimnagels, gehaalde tekst en meta-gegevens, en archieven omzetten.

Ontwikkelaars kunnen aangepaste middelentoepassingen (ook wel aangepaste workers genoemd) insluiten om aangepaste gebruiksgevallen aan te pakken. De service werkt op de [!DNL Adobe I/O] runtime. Het kan worden uitgebreid door [!DNL Adobe Developer App Builder] toepassingen zonder kop geschreven in Node.js. Deze kunnen aangepaste bewerkingen uitvoeren, zoals het aanroepen van externe API&#39;s voor het uitvoeren van afbeeldingsbewerkingen of het gebruiken van [!DNL Adobe Sensei] ondersteuning.

[!DNL Adobe Developer App Builder] is een framework voor het ontwikkelen en implementeren van aangepaste webtoepassingen op [!DNL Adobe I/O] om Adobe Experience Cloud-oplossingen uit te breiden. Om aangepaste toepassingen te maken, kunnen ontwikkelaars [!DNL React Spectrum] (UI toolkit van Adobe), creeer microservices, creeer douanegebeurtenissen, en orkest APIs. Zie [documentatie van Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>Op dit moment worden de [!DNL Asset Compute Service] kan alleen via [!DNL Experience Manager] als [!DNL Cloud Service]. Beheerders maken verwerkingsprofielen die de [!DNL Asset Compute Service] om elementen door te geven voor verwerking. Zie [gebruik microservices en verwerkingsprofielen voor bedrijfsmiddelen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## Ondersteund gebruik van [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] ondersteunt een aantal gangbare gevallen voor zakelijk gebruik, zoals eenvoudige beeldverwerking; specifieke omzettingen van Adobe-toepassingen; en de verwezenlijking van douanetoepassingen die complexe bedrijfsvereisten ordenen.

U kunt [!DNL Asset Compute] webservice voor het genereren van miniaturen voor verschillende bestandstypen, hoogwaardige afbeeldingsrenderingen voor de [ondersteunde bestandsindelingen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). Zie [gebruik gevallen die via aangepaste configuratie worden ondersteund](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>De service biedt geen opslag van bedrijfsmiddelen. Gebruikers geven deze informatie en geven verwijzingen naar de locatie van bron- en uitvoerbestanden in de cloudopslag.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Overzicht van de verwerking van bedrijfsmiddelen met assetmicroservices in [!DNL Adobe Experience Manager] als [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Documentatie over Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).
>* [Ondersteunde bestandsindelingen voor verwerking](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
