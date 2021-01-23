---
ms.openlocfilehash: 73ba9c271c9c6675ffbb22ce4f800ffb652c715d
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942131"
---
<!-- markdownlint-disable MD002 MD041 -->

Este tutorial ensina a criar um aplicativo Web principal ASP.NET que usa a API do Microsoft Graph para recuperar informações de calendário para um usuário.

> [!TIP]
> Se preferir apenas baixar o tutorial concluído, você pode baixar ou clonar o repositório [do GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core) Consulte o arquivo README na pasta **de demonstração** para obter instruções sobre como configurar o aplicativo com uma ID e um segredo do aplicativo.

## <a name="prerequisites"></a>Pré-requisitos

Antes de começar este tutorial, você deve ter o [SDK do .NET Core](https://dotnet.microsoft.com/download) instalado em seu computador de desenvolvimento. Se você não tiver o SDK, visite o link anterior para opções de download.

Você também deve ter uma conta pessoal da Microsoft com uma caixa de correio no Outlook.com ou uma conta de trabalho ou de estudante da Microsoft. Se você não tiver uma conta da Microsoft, há algumas opções para obter uma conta gratuita:

- Você pode [se inscrever para uma nova conta pessoal da Microsoft.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Você pode se inscrever no Programa para Desenvolvedores do [Office 365](https://developer.microsoft.com/office/dev-program) para obter uma assinatura gratuita do Office 365.

> [!NOTE]
> Este tutorial foi escrito com o .NET Core SDK versão 5.0.102. As etapas neste guia podem funcionar com outras versões, mas que não foram testadas.

## <a name="feedback"></a>Comentários

Faça comentários sobre este tutorial no repositório [do GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core)
