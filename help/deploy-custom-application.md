---
title: Implementeren [!DNL Asset Compute Service] aangepaste toepassing
description: Implementeren [!DNL Asset Compute Service] aangepaste toepassing.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 50f69e16772cee7f79a812f2b86f0ef0221db369
workflow-type: tm+mt
source-wordcount: '190'
ht-degree: 0%

---

# Een aangepaste toepassing implementeren {#deploy-custom-application}

Om uw toepassing op te stellen, gebruik [Implementatie van een aio-app](https://github.com/adobe/aio-cli#aio-appdeploy) gebruiken. In de terminal geeft de opdracht een URL weer voor toegang tot de aangepaste toepassing. De URL heeft de notatie `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Als u dezelfde URL wilt ophalen zonder de toepassing opnieuw te implementeren, gebruikt u [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) gebruiken.

Gebruik de URL in een [Profiel verwerken in [!DNL Experience Manager] als [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) om uw toepassing te integreren met [!DNL Experience Manager] als [!DNL Cloud Service].

Zorg ervoor dat uw project en de werkruimte van de Bouwer App met het project en de [!DNL Experience Manager] als [!DNL Cloud Service] omgeving waarin u de handeling wilt gebruiken. Het heeft verschillende omgevingen voor ontwikkeling, staging en productie. U kunt de omgeving controleren door `AIO_runtime_*` referenties die in uw ENV-bestand zijn gedefinieerd in de hoofdmap van uw Adobe Developer App Builder-toepassing. Bijvoorbeeld, om aan `Stage` werkruimte, de `AIO_runtime_namespace` heeft de notatie `xxxxxx_xxxxxxxxx_stage`. Om te integreren met [!DNL Experience Manager] als [!DNL Cloud Service] Productieomgeving, toepassings-URL&#39;s uit uw Adobe Developer App Builder gebruiken `Production` werkruimte.

>[!CAUTION]
>
>Gebruik geen persoonlijke werkruimte voor kritieke toepassingen [!DNL Experience Manager] omgevingen.

>[!MORELIKETHIS]
>
>* [Omgevingen begrijpen en beheren in [!DNL Experience Manager] als [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

