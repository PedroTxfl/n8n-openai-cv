# 🤖 Triagem de Currículos com IA (n8n + OpenAI)

Este projeto contém um workflow de [n8n](https://n8n.io/) que automatiza a triagem e análise de currículos (CVs) usando a API da OpenAI.

O fluxo de trabalho baixa um CV em PDF, extrai seu conteúdo e o envia para o modelo `gpt-4o-mini` junto com uma descrição de vaga. A IA então retorna uma análise estruturada em JSON, avaliando a adequação do candidato com uma pontuação, um resumo e listas de prós e contras.

## 📜 Descrição do Fluxo de Trabalho

Este fluxo de trabalho é **ideal para agências de recrutamento, profissionais de RH e gerentes de contratação** que procuram automatizar a triagem inicial de currículos. É especialmente útil para **organizações que lidam com grandes volumes de candidaturas** e que procuram otimizar o seu processo de recrutamento.

Ele fornece uma **pontuação de correspondência**, um **resumo da adequação do candidato** e **principais *insights*** sobre porque o candidato se encaixa (ou não) na vaga.

## ⚙️ Nós Utilizados

O workflow é composto pelos seguintes passos:

1.  **When clicking ‘Test workflow’** (`manualTrigger`): Inicia o fluxo manualmente para testes.
2.  **Set Variables** (`Set`): Define as variáveis estáticas essenciais:
    * `file_url`: O link direto para o currículo em PDF (ex: armazenado no Supabase, S3, etc.).
    * `job_description`: O texto completo da descrição da vaga.
    * `prompt`: O prompt do sistema que instrui a IA a agir como um recrutador rigoroso.
    * `json_schema`: O schema JSON que força a saída da OpenAI para um formato estruturado.
3.  **Download File** (`httpRequest`): Baixa o arquivo PDF a partir da `file_url`.
4.  **Extract Document PDF** (`extractFromFile`): Extrai o texto puro do arquivo PDF baixado.
5.  **OpenAI - Analyze CV** (`httpRequest`): Envia o texto extraído do CV, o `prompt` e o `json_schema` para a API do OpenAI (`gpt-4o-mini`) para análise.
6.  **Parsed JSON** (`Set`): Converte a resposta em string JSON da OpenAI em um objeto JSON utilizável dentro do n8n.

## 🚀 Como Usar

Siga estes passos para configurar e executar o workflow na sua instância n8n.

### 1. Pré-requisitos

* Uma instância do n8n (local ou na nuvem).
* Uma chave de API da OpenAI.

### 2. Instalação

1.  **Baixe o Workflow**: Faça o download do arquivo `Triagem_CVs.json` deste repositório.
2.  **Importe no n8n**: Na sua interface do n8n, clique em "Import from File" e selecione o arquivo `.json`.

### 3. Configuração

1.  **Credenciais da OpenAI**:
    * Vá até o nó **OpenAI - Analyze CV**.
    * Na seção "Authentication", selecione "Predefined Credential Type".
    * Em "Credential", crie uma nova credencial do tipo "OpenAI API" e cole sua chave de API.

2.  **Ajuste das Entradas**:
    * Vá até o nó **Set Variables**.
    * Modifique o campo `value` das seguintes variáveis:
        * `file_url`: Insira o link direto (URL pública) do currículo em PDF que você deseja analisar.
        * `job_description`: Cole a descrição da vaga para a qual o CV será avaliado.

### 4. Execução

Após configurar as credenciais e as variáveis de entrada, clique em **"Test workflow"** no nó inicial. A execução passará por todos os nós, e o resultado final da análise estará disponível na saída do nó **Parsed JSON**.

## 📄 Estrutura da Saída

A IA é instruída a retornar um JSON que corresponde ao schema definido. O resultado final no nó `Parsed JSON` terá a seguinte estrutura:

```json
{
  "percentage": 80,
  "summary": "Candidato forte com vasta experiência em desenvolvimento de software e cloud, mas sem menção direta a ferramentas de NLP como Langchain, o que é um requisito.",
  "reasons-suit": [
    {
      "name": "Experiência em Cloud",
      "text": "Proficiência demonstrada com AWS, Azure e Google Cloud, alinhada com os requisitos da vaga."
    },
    {
      "name": "Anos de Experiência",
      "text": "Possui 5+ anos de experiência como engenheiro de software, conforme solicitado."
    }
  ],
  "reasons-notsuit": [
    {
      "name": "Familiaridade com NLP",
      "text": "O currículo não menciona experiência com Langchain, LangGraph ou outros pacotes de NLP (Spacy, PyTorch) listados na vaga."
    },
    {
      "name": "Localização",
      "text": "A vaga tem preferência por NYC, Boston ou SF, e a localização do candidato não foi identificada no CV."
    }
  ]
}
