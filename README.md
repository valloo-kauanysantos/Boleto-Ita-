# Boleto-Itau-


## Resumo Executivo - Integração BolePix Itaú (Boleto + Pix)

Esta integração conecta o fluxo de faturas de carga da Valloo à API BolePix/BoleCode do Itaú, permitindo gerar boletos registrados com QR Code Pix diretamente no B2B e no portal Valloo.

Com isso, o cliente:
- Gera o boleto a partir da própria fatura.
- Visualiza QR Code Pix e linha digitável na tela.
- Baixa o PDF do boleto após ação manual.
- Em uma fase seguinte, terá baixa automática da fatura via webhook e fallback, reduzindo o trabalho manual do financeiro.

A arquitetura segue o Provider Pattern para provedores de pagamento, preparando o sistema para integrar novos bancos sem reescrever regras de negócio.

---

### Problema

- Pagamento de faturas de carga hoje é manual e pouco rastreável.
- O financeiro precisa identificar e baixar faturas pagas sem apoio automático do banco.
- Cada novo banco exige integração ad-hoc, sem padrão de provedores.

### Objetivo

- Registrar boletos BolePix do Itaú diretamente a partir das faturas.
- Exibir QR Code Pix, linha digitável e PDF do boleto no B2B e no portal Valloo.
- Automatizar a confirmação de pagamento (webhook + fallback).
- Padronizar integração via Provider Pattern, preparando o sistema para múltiplos bancos.

---

## Funcionalidades

### [FEAT-01] Autenticação API Itaú — Gestão de Credenciais e Renovação de Token  
https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1239

Garante que o backend consiga se autenticar com segurança na API do Itaú (sandbox e produção), usando certificado mTLS e OAuth2, sem quebrar o fluxo quando o token expirar.

**Tasks:**
- **[TASK-001] Setup de ambiente de testes - Sandbox Itaú**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1275  
  Configura sandbox, certificado e testes básicos de autenticação, registro de cobrança e webhook.

- **[TASK-002] Camada de abstração de provedor de pagamento**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1276  
  Implementa o provedor de pagamento do Itaú (BolePix) seguindo o Provider Pattern, com autenticação, registro de cobrança, consulta de status e validação de webhook.

---

### [FEAT-02] Registro de Boleto na API Itaú + Persistência e Idempotência  
https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1240

Permite que o usuário gere, a partir da fatura, um boleto registrado no Itaú com QR Code Pix, salvando todos os dados necessários para reexibir a cobrança sem duplicidade.

**Tasks:**
- **[TASK-003] Modelagem de dados e migrations - BolePix Itaú**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1277  
  Cria/ajusta tabelas para armazenar dados bancários por produto, identificador de cobrança Itaú, QR Code, código de barras e eventos de pagamento.

- **[TASK-004] Endpoint de geração de cobrança - Backend**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1278  
  Implementa o endpoint que registra a cobrança no Itaú a partir da fatura, garantindo idempotência (não duplicar cobrança), vínculo obrigatório fatura↔Itaú e tratamento de erros.

- **[TASK-005] Interface de pagamento - Frontend**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1280  
  Implementa a UI no B2B/portal para gerar o boleto, exibir QR Code Pix, linha digitável, copiar código de barras e tratar estados de loading/erro.

- **[TASK-006] Gestão de configuração bancária por produto/whitelabel**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1283  
  Permite cadastrar e validar os dados bancários do Itaú (agência, conta, carteira etc.) por produto/whitelabel, bloqueando geração de cobrança quando a configuração não estiver completa ou inválida.

---

### [FEAT-03] Webhook Itaú + Confirmação automática de pagamento  
https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1241

Responsável por automatizar a baixa da fatura/pedido quando o pagamento é confirmado pelo Itaú, evitando conciliação manual pelo financeiro.

**Tasks:**
- **[TASK-007] Endpoint de webhook - Confirmação de pagamento Itaú**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1284  
  Cria o endpoint que recebe notificações de pagamento, valida assinatura, garante idempotência, atualiza fatura/pedido e registra o evento.

- **[TASK-008] Job de consulta fallback - Faturas sem webhook**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1285  
  Implementa um job periódico que consulta o status das cobranças no Itaú para faturas que não receberam webhook, garantindo que nenhuma fatura paga fique presa em “aguardando pagamento”.

---

### [FEAT-04] Exibição do PDF do Boleto  
https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1242

Gera e disponibiliza o PDF do boleto registrado (com QR Code Pix e código de barras) para download/visualização tanto no B2B quanto no portal Valloo.

**Tasks:**
- **[TASK-009] Geração e Download de PDF do Boleto Registrado**  
  https://github.com/valloo-tecnologia/Projetos-Desenvolvimento/issues/1286  
  Cria o job assíncrono de geração do PDF logo após o registro do boleto e os endpoints necessários para visualização/download, garantindo idempotência e que falhas de PDF não impactem o fluxo de cobrança.

---

## Prioridade de Entrega

1. **FEAT-02 + FEAT-04** — Registrar boleto e permitir download do PDF (valor direto para o cliente).
2. **FEAT-01** — Autenticação robusta e segura com o Itaú (pré-requisito técnico).
3. **FEAT-03** — Automatizar confirmação de pagamento (redução de esforço do financeiro).

