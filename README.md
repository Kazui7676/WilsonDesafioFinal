# 🏗️ Smart Piping Reconciliation — Wilson Sons

> **Desafio 1 (Power Developers) — Wilson Sons**  
> *Sistema Inteligente de Conciliação de Planilhas (Jobbook) e Desenhos Técnicos de Tubulação via Inteligência Artificial Multimodal e Automação de Processos.*

---

## 📝 Visão Geral do Projeto

Na logística de suprimentos e engenharia de operações marítimas e portuárias da **Wilson Sons**, a conferência manual entre as Listas de Materiais (BOM) dos desenhos técnicos de tubulação e a planilha oficial de envios (**Jobbook**) é um processo moroso e vulnerável a falhas humanas.

O **Smart Piping Reconciliation** automatiza integralmente esse fluxo. Por meio de uma interface intuitiva no **Builder.io**, o usuário faz o upload do PDF do desenho técnico. Uma esteira automatizada no **Make.com** dispara a API de IA Multimodal do **Google Gemini**, que executa a visão computacional para ler e extrair os dados da tabela do PDF. Em seguida, o sistema realiza o cruzamento automático com a base de dados do **Google Sheets**, **identificando divergências em tempo real** e disparando um relatório executivo formatado por **E-mail** com os status de convergência e inconsistência.

---

## 🔍 Tratamento de Divergências (O Que Não Coincide)

A inteligência da solução foca no cruzamento exato entre a demanda planejada e o projeto técnico fornecido:

* **🚨 DIVERGENT (Divergência Encontrada):** Ocorre quando a quantidade do item no desenho técnico (`quantidade_desenho`) difere da quantidade cadastrada no Jobbook (`QtyDemand`), ou quando o item extraído do desenho não possui cadastro prévio na base.
* **✅ MATCHED (Dados Coincidentes):** Ocorre quando os códigos de peça (Part Number / Código SAP) e as quantidades batem perfeitamente em ambas as fontes.

O relatório enviado por e-mail compila essa análise em uma tabela HTML com sinalização visual imediata, permitindo tomada de decisão rápida e evitando envio de materiais incorretos para a operação.

---

## 🛠️ Stack Tecnológica (Arquitetura No-Code & GenAI)

| Camada | Tecnologia | Função |
| :--- | :--- | :--- |
| **Front-end (Interface)** | Builder.io | Dashboard interativo e formulário de upload do PDF |
| **Mensageria (Gatilho)** | Webhooks (Make) | Recebimento instantâneo dos dados da requisição (PDF + E-mail) |
| **Armazenamento de Arquivos**| Google Drive | Armazenamento do PDF e geração de link direto para a IA |
| **Cérebro de IA** | Google Gemini 1.5 Flash | Visão computacional e extração de dados em formato JSON estrito |
| **Banco de Dados** | Google Sheets | Base do Jobbook Oficial e registros de extração da IA |
| **Notificação & Relatório** | Gmail | Disparo automatizado do relatório dinâmico em HTML com os status de conciliação |

---

## 📐 Fluxo da Arquitetura de Dados (Data Pipeline)

A esteira de automação segue uma sequência robusta de ponta a ponta:

```text
[Interface Builder.io] ➔ [Webhook Make] ➔ [Upload Google Drive] ➔ [Geração de Link Direto] ➔ [Leitura Gemini IA] ➔ [Parse JSON & Iterator] ➔ [Cruzamento Google Sheets] ➔ [Text Aggregator HTML] ➔ [E-mail de Notificação]
```

1. **Upload do Arquivo:** O usuário insere o e-mail do destinatário e faz o upload do PDF do desenho na interface.
2. **Gatilho HTTP:** O Builder.io envia uma requisição `POST` para o Webhook do Make.com.
3. **Persistência no Drive:** O arquivo é salvo no Google Drive e um link de visualização direta é gerado para consumo da IA.
4. **Extração Multimodal:** O Gemini lê a Lista de Materiais do PDF e devolve um JSON padronizado com os itens encontrados.
5. **Conciliação Automatizada:** O módulo *Iterator* percorre item por item extraído, gravando o log na aba `Extracao_Desenho` e fazendo a busca na aba `Jobbook_Oficial` para validar o balanceamento de quantidades.
6. **Consolidação do Relatório:** O *Text Aggregator* formata as linhas comparativas em HTML e o módulo do Gmail dispara o relatório para o responsável.

---

## 📋 Estrutura da Base de Dados (Google Sheets)

A planilha de apoio é estruturada em duas abas:

### 1. Aba `Jobbook_Oficial`
Registra os dados operacionais mestres exigidos pela Wilson Sons:
* `ProjectID`, `FunctionCode`, `FunctionDescription`, `PartID`, `Código Sap`, `PartDescription`, `QtyDemand`, `QtyShipped`, `UoMShipmentID`, `ShipmentStatus`, `Note`, `DrawingNo`, `PosItemID`, `ItemCollectedInID`, `SupplierPartNumber`, `ShipmentTransportCode`, `ShipmentTransportNumber`, `HandlingUnit`, `DelActivity`, `ETADateAtPort`.

### 2. Aba `Extracao_Desenho`
Registra o histórico de varreduras realizadas pela IA:
* `DrawingNo`, `part_number`, `descricao`, `quantidade_desenho`, `Data_Leitura`.

---

## 🧠 Engenharia de Prompt (Módulo Gemini)

O prompt enviado à API do Gemini força uma resposta em formato estruturado limpo, garantindo previsibilidade para o código do Make:

* **System Instruction:**
  > "Examine visualmente o arquivo PDF do desenho técnico de tubulação fornecido. Localize a 'Lista de Materiais' (Bill of Materials) presente no desenho. Extraia as informações de cada item da tabela e responda EXCLUSIVAMENTE em formato JSON estruturado, sem explicações extras, introduções ou marcadores de bloco markdown. O número do desenho técnico deve ser atribuído à chave 'DrawingNo'."

* **Estrutura de Saída (JSON Target):**
```json
{
  "DrawingNo": "TEXTO_DO_NUMERO_DO_DESENHO",
  "items": [
    {
      "part_number": "CÓDIGO_OU_PART_NUMBER_DA_PEÇA",
      "descricao": "DESCRIÇÃO_DO_MATERIAL_EM_PORTUGUÊS",
      "quantidade_desenho": 0
    }
  ]
}
```

---

## 🎨 Regras Visuais da Dashboard (Builder.io)

* **Design System Industrial:** Paleta baseada nas cores institucionais da Wilson Sons (Azul Marinho `#002060` e Branco).
* **Layout em Colunas Duplas:** Exibição comparativa entre a Demanda do Jobbook vs. Extração da IA.
* **Sinalização Visual de Status:**
  * **🚨 DIVERGENT:** Fundo e rótulo em destaque quando a quantidade do desenho difere da demanda da planilha.
  * **✅ MATCHED:** Fundo em tom verde suave quando as quantidades e dados batem 100%.

---

## 🚀 Como Executar o Projeto

1. **Configuração da Base:** Crie uma planilha no Google Sheets contendo as abas `Jobbook_Oficial` e `Extracao_Desenho` preenchidas com os dados de teste.
2. **Importação no Make:** Monte o cenário no Make.com configurando o Webhook, as credenciais do Google Drive, Google Sheets, Gmail e a API Key do Google Gemini.
3. **Publicação da Interface:** Configure os componentes no Builder.io direcionando a requisição `POST` do formulário para o endpoint do Webhook do Make.
4. **Homologação:** Acesse a tela, suba um PDF de desenho técnico de tubulação, insira o e-mail para recebimento e clique em **Analyze Drawing & Send Report**. Verifique o e-mail com a tabela de divergências gerada.

---
*Desenvolvido com foco em governança, eficiência operacional e transformação digital para o Desafio Wilson Sons.*
