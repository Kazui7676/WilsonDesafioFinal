# WilsonDesafioFinal
# 🏗️ Wilson Sons

> **Desafio 1 (Power Developers) - Wilson Sons** > *Sistema Inteligente de Conciliação de Planilhas (Jobbook) e Desenhos Técnicos de Tubulação via Inteligência Artificial Multimodal.*

---

## 📝 Visão Geral do Projeto

Na logística de suprimentos e engenharia da Wilson Sons, a conferência manual entre os desenhos técnicos de tubulação e a planilha oficial de envios (**Jobbook**) é um processo demorado e sujeito a falhas humanas. 

O **Smart Piping Reconciliation** automatiza completamente este processo. Através de uma interface amigável desenvolvida no **Builder.io**, o usuário envia o PDF de um desenho técnico. Uma esteira automatizada no **Make.com** aciona a API de IA Multimodal do **Google Gemini 3.5 Flash**, que realiza a visão computacional e extração de dados da lista de materiais do desenho, confronta as informações com o banco de dados no **Google Sheets** e gera alertas automáticos em tempo real na tela e relatórios executivos via **E-mail**.

---

## 🛠️ Stack Tecnológica (Arquitetura No-Code & GenAI)

| Camada | Tecnologia | Função |
| :--- | :--- | :--- |
| **Front-end (Interface)** | Builder.io | Dashboard interativo e campo de upload do PDF |
| **Mensageria (Gatilho)** | Webhooks (Make) | Recebimento instantâneo do payload (PDF + E-mail) |
| **Armazenamento de Arquivos**| Google Drive | Armazenamento e geração de link limpo para a IA |
| **Cérebro de IA** | Google Gemini 3.5 Flash | Visão computacional e extração estruturada (JSON) |
| **Banco de Dados** | Google Sheets | Armazenamento do Jobbook Oficial e logs de extração |
| **Notificação** | Gmail / Outlook | Disparo automatizado de relatórios e alertas de divergência |

---

## 📐 Fluxo da Arquitetura de Dados (Data Pipeline)

A nossa esteira de automatização foi desenhada para ser rápida e à prova de falhas de formato binário, seguindo o fluxo:

`[Tela Builder.io] ➔ [Webhook do Make] ➔ [Upload Google Drive] ➔ [Get Share Link] ➔ [Leitura Gemini 3.5] ➔ [Loop Iterator / Google Sheets] ➔ [Text Aggregator HTML] ➔ [E-mail]`

1. **Upload Centralizado:** O usuário insere o e-mail de destino e o PDF do desenho na interface do Builder.io.
2. **Disparo Instantâneo:** A tela faz um envio HTTP POST para o Webhook do Make.com.
3. **Tratamento do Arquivo:** O Make salva o PDF no Google Drive e gera um link compartilhado direto, limpando quaisquer erros de codificação binária para a IA.
4. **Extração com Visão Computacional:** O Gemini 3.5 Flash lê a lista de materiais do desenho técnico e gera um formato JSON estruturado contendo `part_number`, `descricao` e `quantidade_desenho`.
5. **Gravação e Conciliação:** O Parse JSON traduz a resposta, o *Iterator* percorre item por item, salvando os dados na aba `Extracao_Desenho` e realizando a busca cruzada com a aba `Jobbook_Oficial`.
6. **Agregação e Relatório:** O *Text Aggregator* (posicionado estrategicamente fora do loop do iterator) renderiza as linhas dinâmicas em uma tabela HTML executiva e o módulo de e-mail envia o relatório consolidado.

---

## 📋 Estrutura da Base de Dados (Google Sheets)

O projeto utiliza uma única planilha contendo duas abas principais:

### 1. Aba `Jobbook_Oficial`
Contém todas as colunas oficiais exigidas pela operação da Wilson Sons:
* `ProjectID`, `FunctionCode`, `FunctionDescription`, `PartID`, `Código Sap`, `PartDescription`, `QtyDemand`, `QtyShipped`, `UoMShipmentID`, `ShipmentStatus`, `Note`, `DrawingNo`, `PosItemID`, `ItemCollectedInID`, `SupplierPartNumber`, `ShipmentTransportCode`, `ShipmentTransportNumber`, `HandlingUnit`, `DelActivity`, `ETADateAtPort`.

### 2. Aba `Extracao_Desenho`
Registra o histórico de leitura da Inteligência Artificial:
* `DrawingNo`, `part_number`, `descricao`, `quantidade_desenho`, `Data_Leitura`.

---

## 🧠 Engenharia de Prompt (Módulo Gemini)

O comando estruturado enviado ao Gemini 3.5 Flash garante uma resposta estrita em formato de dados, eliminando conversas desnecessárias da IA:

* **Instrução do Sistema:**
  "Examine visualmente o arquivo PDF do desenho técnico de tubulação fornecido. Localize a 'Lista de Materiais' (Bill of Materials) presente no desenho. Extraia as seguintes informações de cada item da tabela e responda EXCLUSIVAMENTE em formato JSON estruturado, sem explicações extras, tags markdown adicionais (como \`\`\`json) ou introduções. O número do desenho técnico deve ser atribuído à chave 'DrawingNo'."

* **Estrutura de Saída Alvo:**
* {
"DrawingNo": "TEXTO_DO_NUMERO_DO_DESENHO",
"items": [
{
"part_number": "CÓDIGO_OU_PART_NUMBER_DA_PEÇA",
"descricao": "DESCRIÇÃO_DO_MATERIAL_EM_PORTUGUÊS",
"quantidade_desenho": NUMERO_INTEIRO_DA_QUANTIDADE
}
]
}
---

## 🎨 Regras Visuais do Dashboard (Builder.io)

O painel de controle simula uma aplicação corporativa robusta com as seguintes diretrizes fornecidas à IA geradora de interface:
* **Tema Industrial:** Cores azul marinho escuro (`#002060`) e branco.
* **Telas em Duas Colunas:** Exibição lado a lado dos dados do Jobbook vs. Dados Extraídos pela IA.
* **Sistema de Alertas Dinâmicos (Reconciliação):**
* **🚨 DIVERGENT:** Linhas destacadas em fundo vermelho suave caso a quantidade do desenho seja diferente da demanda do Jobbook.
* **✅ MATCHED:** Linhas destacadas em verde suave caso os valores batam perfeitamente.

---

## 🚀 Como Executar este Projeto

1. **Base de Dados:** Monte o seu Google Sheets com as duas abas descritas e insira dados fictícios na tabela do Jobbook para testar.
2. **Integração:** Importe ou monte o fluxo no Make.com utilizando a URL do seu Webhook customizado.
3. **Interface:** Crie as seções no Builder.io utilizando o prompt mestre fornecido na documentação de desenvolvimento, apontando as requisições para o seu Webhook do Make.
4. **Homologação:** Clique em *Run Once* no Make, insira o e-mail do destinatário na tela do Builder, suba o PDF técnico de tubulação e clique em *Analyze Drawing & Send Report*. Confira o e-mail com o relatório executivo formatado em HTML.

---
*Desenvolvido com foco em agilidade, governança e transformação digital para o Desafio Wilson Sons.*
