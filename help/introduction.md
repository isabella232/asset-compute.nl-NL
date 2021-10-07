---
title: Inleiding aan  [!DNL Asset Compute Service]
description: '[!DNL Asset Compute Service] is een service voor de verwerking van eigen middelen in de cloud die de complexiteit vermindert en de schaalbaarheid verbetert.'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: a2460a0719f8c585ed72e44c904aa0df33301032
workflow-type: tm+mt
source-wordcount: '307'
ht-degree: 0%

---

# Overzicht van [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] is een schaalbare en uitbreidbare service voor  [!DNL Adobe Experience Cloud] het verwerken van digitale elementen. Het kan beeld, video, document, en andere dossierformaten in verschillende vertoningen met inbegrip van duimnagels, gehaalde tekst en meta-gegevens, en archieven omzetten.

Ontwikkelaars kunnen aangepaste middelentoepassingen (ook wel aangepaste workers genoemd) insluiten om aangepaste gebruiksgevallen aan te pakken. De service werkt bij de [!DNL Adobe I/O]-runtime. De functie kan worden uitgebreid via [!DNL Project Firefly] toepassingen zonder kop die zijn geschreven in Node.js. Deze kunnen aangepaste bewerkingen uitvoeren, zoals het aanroepen van externe API&#39;s om afbeeldingsbewerkingen uit te voeren of ondersteuning voor [!DNL Adobe Sensei] benutten.

[!DNL Project Firefly] is een raamwerk voor het ontwikkelen en implementeren van aangepaste webtoepassingen bij  [!DNL Adobe I/O] uitvoering voor het uitbreiden van Adobe Experience Cloud-oplossingen. Om douanetoepassingen tot stand te brengen, kunnen de ontwikkelaars hefboomwerking [!DNL React Spectrum] (de toolkit van UI van Adobe), microdiensten tot stand brengen, douanegebeurtenissen tot stand brengen, en APIs orchestreren. Zie [documentatie van Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

>[!NOTE]
>
>Momenteel kan [!DNL Asset Compute Service] alleen via [!DNL Experience Manager] als [!DNL Cloud Service] worden gebruikt. Beheerders maken verwerkingsprofielen die de [!DNL Asset Compute Service] kunnen aanroepen om elementen door te geven voor verwerking. Zie [Elementmicroservices en verwerkingsprofielen gebruiken](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## Ondersteund gebruik van [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] ondersteunt een aantal gangbare gevallen voor zakelijk gebruik, zoals eenvoudige beeldverwerking; specifieke omzettingen van Adobe-toepassingen; en de verwezenlijking van douanetoepassingen die complexe bedrijfsvereisten ordenen.

Met de webservice [!DNL Asset Compute] kunt u miniaturen genereren voor verschillende bestandstypen en kwalitatief hoogstaande afbeeldingsrenderingen voor de [ondersteunde bestandsindelingen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). Zie [gebruik gevallen die door douaneconfiguratie](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) worden gesteund.

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
>* [Overzicht van de verwerking van bedrijfsmiddelen met  [!DNL Adobe Experience Manager] onder andere [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html) assetmicroservices.
>* [Documentatie van het project zonder problemen](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).
>* [Ondersteunde bestandsindelingen voor verwerking](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
