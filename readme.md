# 🤖 Automação com n8n + WAHA (WhatsApp)

Este repositório contém dois fluxos criados no n8n que integram com a API **WAHA** para envio e recebimento de mensagens via **WhatsApp**:

- [`chatbot`](#fluxo-chatbot)
- [`notificacao_tabela_atualizada`](#fluxo-notificacao_tabela_atualizada)

## ✨ Requisitos

- n8n configurado localmente ou em ambiente de produção.
- Conta ativa e configurada na WAHA API.
- Banco de dados SQL Server com as tabelas de vendas (que foi disponibilizado no drive com o Banco de dados.sql).
- Redis (opcional, para memória conversacional com IA).
- API externa com números de WhatsApp: `https://atos-capital-backend-docker.onrender.com/api/auth/whatsapp_numbers/`.

---

## 🎥 Recomendações de Vídeo

Para aprofundar seu conhecimento sobre n8n e integrações com WhatsApp, confira estes vídeos:

- [Como criar chatbot no WhatsApp com WAHA e n8n](https://youtu.be/KkKlfAb3TSI?si=6i5xk-W4o2X8IPy5)
- [Automatizando notificações com n8n e Webhooks](https://youtu.be/SJuEeAuiuDE?si=DaeSfr1M6nHzyZ7J)

---

## 🧠 Fluxo: `chatbot`

### Objetivo

Automatiza o atendimento de usuários no WhatsApp, respondendo perguntas relacionadas a **vendas**, com auxílio de **Inteligência Artificial**.

### Etapas do fluxo

1. **Webhook**: Recebe mensagens do WAHA via HTTP POST.
2. **Set (dados)**: Extrai informações como `session`, `chatId`, `pushName`, `message` e `event`.
3. **Switch**: Verifica se o evento é do tipo `message`.
4. **Consulta SQL**: Recupera dados da tabela `tbVendasDashboard` com os últimos valores de vendas.
5. **Formatação**: Transforma os dados SQL em texto formatado.
6. **Merge**: Junta a mensagem do usuário com os dados formatados.
7. **IA com Langchain + Groq**: Usa linguagem natural para gerar respostas (ex: "qual loja vendeu mais hoje?").
8. **Memória com Redis**: Armazena o histórico da conversa por 1 hora.
9. **Envia resposta**: Utiliza o nó WAHA para enviar a mensagem gerada pela IA ao usuário.
10. **GET números do backend**: Recupera lista de números que devem receber notificações.
11. **Formatação e envio em lote**: Converte os números e envia mensagens para cada um.

### Exemplos de perguntas entendidas pela IA

- "Qual o total de vendas do mês?"
- "Quanto faturamos hoje?"
- "Compare os valores com o mês passado."

---

## 📢 Fluxo: `notificacao_tabela_atualizada`

### Objetivo

Envia uma mensagem automática no WhatsApp para diversos números informando que os dados da tabela de vendas foram atualizados.

### Etapas do fluxo

1. **Webhook** (opcional): Pode ser usado para disparar manualmente o fluxo.
2. **Consulta SQL**: Puxa os dados mais recentes da `tbVendasDashboard`.
3. **IF**: Verifica se a venda (`vlVenda`) não está nula.
4. **GET números**: Busca a lista de números na API externa.
5. **Formatação**: Adapta os números para o formato da WAHA.
6. **SplitInBatches**: Divide os contatos em lotes.
7. **HTTP para sessão WAHA**: Recupera sessões ativas (opcional).
8. **Envio da mensagem**: Notifica que os dados do dia `dtVenda` foram atualizados.

### Exemplo de mensagem enviada

> "As informações referentes ao dia 05/06/2025 estão atualizadas!"

---

## ⚙️ Tecnologias utilizadas

- [n8n](https://n8n.io/)
- [WAHA API](https://waha.devlikeapro.com/)
- [LangChain + Groq](https://www.langchain.com/)
- [Redis](https://redis.io/)
- [SQL Server](https://www.microsoft.com/en-us/sql-server)

---

## 🔐 Variáveis sensíveis

Certifique-se de configurar corretamente as credenciais no n8n:

- **WAHA API Credentials**
- **Microsoft SQL Database**
- **Redis Connection**
- **Groq API Key** (para IA)

---

## 📌 Observações

- O fluxo `chatbot` usa IA para interpretar linguagem natural e oferecer respostas inteligentes.
- O fluxo `notificacao_tabela_atualizada` é útil para alertas automáticos recorrentes.
- Ambos os fluxos podem ser ativados via **Webhook** ou agendados com **Cron** no n8n.
