<h1> Instituto Aritana - Agente Inteligente de Acolhimento e Serviços Sociais (RAG + n8n + Telegram) </h1>

O **Instituto Aritana Bot** é um agente de IA Generativa capaz de consultar o guia de programas, serviços e diretrizes acadêmicas do Instituto em tempo real e responder a dúvidas de famílias e educandos em linguagem natural via Telegram, utilizando arquitetura RAG (Retrieval-Augmented Generation) no n8n.

<h2>📌 Sumário </h2>
- Visão Geral  
- Principais Funcionalidades  
- Arquitetura da Solução  
- Tecnologias Utilizadas  
- Como Executar o Projeto  
- Evidências e Demonstração  
- Deploy e Acesso ao Projeto  
- Autor  



<h2>🚀 Visão Geral</h2>

O projeto consiste em um **sistema de atendimento autônomo focado em serviços socioassistenciais e educacionais**. A partir de uma base de conhecimento estruturada (PDF/CSV contendo o guia de programas do Instituto, com públicos-alvo, faixas etárias, horários, matriz de cursos profissionalizantes, documentos necessários, regras de frequência e políticas internas), o agente utiliza busca semântica para encontrar as informações mais adequadas à solicitação do usuário e formular respostas acolhedoras, alinhadas ao Sistema Preventivo Salesiano.



<h2>🎯 Principais Funcionalidades</h2>

- **Busca Semântica Completa**: Encontra o serviço ideal (CEI, CCA, Circo, CEDESP, SAICA, NCI, MSE) com base na idade, necessidade ou palavra-chave, retornando detalhes como horários de funcionamento, documentação exigida e pré-requisitos.
- **Consulta à Matriz de Cursos e Oficinas**: Informa sobre os cursos profissionalizantes do CEDESP (Stop Motion, Assistente Administrativo, RH, Alfaiate) em todos os turnos, além da grade de oficinas do Circo Social (Capoeira, Teatro, Taekwondo, etc.).
- **Orientação sobre Matrícula e Documentação**: Explica o fluxo de encaminhamento pelo CRAS, lista os documentos obrigatórios (RG, CPF, NIS, Comprovante de Residência, etc.) e orienta sobre a necessidade de presença do responsável legal para menores.
- **Esclarecimento sobre Regras e Políticas**: Informa sobre a frequência mínima de 75%, o protocolo de Busca Ativa (5 faltas consecutivas ou 10 intercaladas) e as diretrizes da Lei Federal nº 15.100/2025 sobre uso de celulares.
- **Memória Conversacional**: Mantém o contexto do diálogo durante toda a sessão do usuário no Telegram.
- **Atendimento Via Telegram**: Interface direta e acessível ao cliente final (famílias e educandos) por meio de um bot autônomo.



<h2> 🏗️ Arquitetura da Solução </h2>

O projeto é dividido em **dois workflows independentes no n8n** para otimizar o processamento e a manutenção:

```
[ PDF ] ──> [ Default Data Loader ] ──> [ Cohere Embeddings ] ──> [ Simple Vector Store ]
                                                                                       │
                                                                                       ▼
[ Usuário / Telegram ] ──> [ Telegram Trigger ] ──> [ AI Agent (Groq) ] <─────────── [ Ferramenta de Busca (Tool) ]
                                                           │
                                                           ▼
                                                 [ Memória de Sessão ]
```

<h2> Workflow de Ingestão (Pipeline de Dados) </h2>
1. Lê a base de dados (arquivo `guia_ de_programas_e _servicos_ Instituto_aritana.pdf` contendo os dados estruturados).
2. Formata os atributos e metadados enriquecidos (faixa etária, horários, cursos, documentos, regras) em texto contínuo via **Default Data Loader** do n8n.
3. Vetoriza as informações usando o modelo **Cohere Embeddings** (`embed-multilingual-v3.0` para suporte ao português).
4. Armazena as representações vetoriais no **Simple Vector Store** (LangChain/n8n).

<h2> Workflow do Agente Inteligente (Telegram & Atendimento) </h2>
1. Recebe e escuta as mensagens dos clientes via **Telegram Trigger**.
2. Processa a intenção do usuário usando o LLM **Groq** (`openai/gpt-oss-120b`) para interpretação de linguagem natural e otimização de resposta.
3. Consulta o **Simple Vector Store** via **Tool Calling (RAG)** para obter dados em tempo real sobre serviços, cursos ou regras.
4. Preserva o contexto da sessão usando **Simple Memory**.
5. Retorna a resposta formatada e humanizada para o chat do Telegram do usuário.

---

<h2> 🛠️ Tecnologias Utilizadas </h2>

- **Orquestração de Automação**: n8n (Deploy em nuvem via OCI ou Self-Hosted)
- **Canal de Comunicação**: Telegram Bot API (ex: `@InstitutoAritanaBot`)
- **Modelo de Linguagem (LLM)**: Groq (`openai/gpt-oss-120b`)
- **Modelo de Embeddings**: Cohere (`embed-multilingual-v3.0`)
- **Banco de Dados Vetorial**: Simple Vector Store (LangChain/n8n)
- **Base de Conhecimento**: PDF (`guia_ de_programas_e _servicos_ Instituto_aritana.pdf`) ou Google Sheets / CSV

---

<h2> ⚙️ Como Executar o Projeto </h2>

Pré-requisitos
- Instância do **n8n** (Local, Docker ou OCI).
- **Chave de API da Groq** (acesso ao LLM).
- **Chave de API da Cohere** (para geração de embeddings).
- **Bot criado no Telegram** via @BotFather (Token de acesso).
- Arquivo de dados: PDF do guia ou CSV estruturado com os campos: *Serviço, Público-alvo, Idade, Horário, Documentos, Cursos, Regras*.

<h2>Passo a Passo</h2>

1. **Importar os Workflows**:
   - Baixe os arquivos `.json` da pasta `workflows/` do repositório (um para Ingestão, outro para o Agente).
   - No n8n, crie um novo workflow e selecione **Import from File** para cada um dos fluxos.

2. **Configurar Credenciais**:
   - No menu **Credentials** do n8n, adicione suas credenciais do:
     - **Telegram Bot API** (Token do bot).
     - **Groq API** (API Key).
     - **Cohere API** (API Key).
     - (Opcional) **Google Sheets** se optar por usar planilha em vez de PDF.

3. **Ajustar a Fonte de Dados**:
   - No Workflow de Ingestão, substitua o nó de leitura para apontar para o seu arquivo PDF ou CSV. Se usar PDF, configure o nó `Read Binary Files` ou um parser específico; se usar CSV, aponte para o arquivo ou URL do Google Sheets.

4. **Executar a Ingestão de Dados**:
   - Abra o **Workflow de Ingestão** (Default Data Loader → Cohere → Simple Vector Store).
   - Clique em **Execute Workflow** para ler o guia, vetorizar todo o conteúdo e popular a memória vetorial do n8n. *Aguarde a finalização com sucesso.*

5. **Publicar o Bot no Telegram**:
   - Abra o **Workflow do Agente**.
   - Certifique-se de que o nó `Telegram Trigger` está configurado com as credenciais do seu bot (ex: `@InstitutoAritanaBot`).
   - Ative o botão **Publish** (ou "Ativar") no canto superior direito para manter o fluxo em execução contínua (listening mode).

6. **Testar a Interação**:
   - Inicie uma conversa com o bot no Telegram.
   - Envie perguntas como: *"Meu filho tem 3 anos, onde posso matricular?"*, *"Quais cursos têm à noite?"*, *"Preciso levar quais documentos?"* ou *"O que acontece se faltar 5 dias?"*.

---

<h2> 📸 Evidências e Demonstração </h2>

(Insira aqui capturas de tela do n8n mostrando a execução da Ingestão, o nó do Agente ativo, e prints da conversa no Telegram respondendo perguntas sobre os serviços do Instituto Aritana com base nos dados vetorizados.)

---

<h2> 🌐 Deploy e Acesso ao Projeto </h2>

- **Workflows n8n**: Disponíveis no repositório oficial do projeto (pasta `/workflows`).
- **Bot Telegram**: Disponível publicamente em [@InstitutoAritanaBot](https://t.me/InstitutoAritanaBot) (exemplo).
- **Infraestrutura**: Recomendamos deploy em nuvem (OCI, AWS ou Render) para manter o bot ativo 24/7.

---

<h2>👤 Autor </h2>

Desenvolvido por Alexandre da Silva Cunha como parte do desafio prático ONE IA.

LinkedIn: https://www.linkedin.com/in/alexandrescunha/

GitHub: https://github.com/alesbc
