
## Objective

- Plataforma editorial política focada em debate estruturado
- Versão digital de revistas clássicas de debate (tribuna, réplica, tréplica)

---

## Core Concepts

### Publications

Existem 3 tipos de publicações:

#### Essays

- Estrutura de debate hierárquico
- Níveis:
	  - 1: tribuna
	  - 2: réplica
	  - 3: tréplica
- Regras:
	  - tribunas podem ter múltiplas réplicas
	  - cada réplica pode ter apenas uma tréplica
	- Associadas a um author (pseudônimo público)

#### Translations

- Traduções de textos
- Sem hierarquia
- Possuem:
	  - original_author (ex: Karl Marx)
	  - translator_name (customizável)
- Não exibem o pseudônimo do author

#### Editorials

- Publicações institucionais
- Sem author
- Sem hierarquia
- Criadas por usuários com can_moderate

---

### Author

- Representa o usuário da plataforma
- Campos:
	  - id
	  - email (identidade única)
	  - name (pseudônimo)
	  - can_moderate
- Criado automaticamente no login (se não existir)

---

### Draft

- Unidade central de escrita
- Sempre pertence a um único author
- Contém conteúdo editável (payload)
- Pode existir antes do login
- Não existe escrita direta em publicação

---

### Submission

- Representa o envio de um draft para moderação
- Possui:
	  - draft_id
	  - email
	  - payload
	  - status
- Apenas submissions aprovadas geram publicação
- Snapshot do draft no momento do envio

---

### Moderation Token

- Token gerado para moderadores
- Um por moderador por submission
- Possui expiração
- Apenas o primeiro uso é válido (race condition controlada)

---

### Intent

Define o comportamento após autenticação

Tipos:

- access_to_dashboard
- reply_to_essay (parentId)
- resume_draft (draftId)

---

## Invariants (VERY IMPORTANT)

### Auth

- email é identidade única do author
- não existe autenticação sem magic link
- tokens são stateless (JWT)
- todo token deve ser validado com Zod
- tokens têm purpose: "magic_link" | "session"

#### confirmAccess:

- valida token
- resolve author
- cria sessão
- resolve redirect
- NÃO cria estado de domínio

---

### Intent

- todo fluxo pós-login é definido pelo intent
- intent deve ser válido no momento da geração
- intent sempre vem do backend
- intent é imutável após emissão

---

### Author

- email é único
- getOrCreateAuthor é idempotente
- name não define identidade

---

### Draft

- pertence a apenas um author
- é a única unidade de escrita
- pode existir antes do login
- pode ser acessado via intent (resume_draft)

---

### Submission

- toda publicação (exceto editorial) vem de uma submission
- apenas submissions aprovadas geram publicação
- submission representa snapshot do draft
- é imutável após aprovação

---

### Essays

- depth ∈ {1,2,3}
- tribunas podem ter múltiplas réplicas
- cada réplica pode ter apenas uma tréplica
- parent_id deve respeitar a hierarquia

---

### Editorials

- não possuem author
- não passam por submission
- derivam diretamente de draft

---

### Moderation

- cada submission gera tokens para moderadores
- tokens expiram
- apenas o primeiro uso é válido
- uso de um token invalida os demais

---

## Security

- nunca confiar em dados vindos do client
- IDs (draftId, parentId) devem ser validados no backend
- acesso a draft deve respeitar author_id

---

## Architecture

### Camadas

- #### Routes (API - Astro)
  
  - validam requests
  - chamam useCases

- #### Use Cases
  
  - orquestram fluxos
  - não acessam infra diretamente

- #### Services
  
  - lógica de domínio + acesso a dados

- #### Providers
  
  - integrações externas (email, etc.)
  - apenas I/O

- #### Schemas (Zod)
  
  - validação de entrada e saída

---

##Flows

1. Access / Login (IMPLEMENTADO)

	1. usuário informa:
		   - email
		   - baseName
		   - intent
	 1. requestAccessUseCase:
		   - gera mailToken
		   - cria confirmUrl
		   - envia email
	 2. usuário acessa link
	3. confirmAccessUseCase:
		   - valida token
		   - getOrCreateAuthor
		   - gera sessionToken
		   - resolve redirect
	4. cookie de sessão é criado

---

2. Resume Draft (IMPLEMENTADO)

- intent: resume_draft
- fluxo:
	  - usuário autentica
	  - redirect → editor com draftId

---

3. Reply Flow (PARCIAL)

- intent: reply_to_essay

Futuro esperado:
- sistema identifica parentId
- cria ou recupera draft de resposta (idempotente)
- redireciona para editor

OBS:
- criação de draft não deve ocorrer no confirmAccess
- deve ocorrer em use case específico ou evento

---

4. Submission Flow (A IMPLEMENTAR)

	1. usuário finaliza draft
	2. cria submission:
		   - snapshot do conteúdo
	3. status inicial: pending
	4. geração de moderation_tokens

---

5. Moderation Flow (A IMPLEMENTAR)

	1. moderador recebe token
	2. ação:
		   - approve → cria publicação
		   - reject → marca submission
	3. primeiro token usado invalida os demais

---

6. Publication Flow (A IMPLEMENTAR)

Essays

- criados a partir de submission aprovada
- definem:
	  - parent_id
	  - depth
	  - path

Translations

- criadas a partir de submission aprovada
- associadas a original_author

Editorials

- criadas diretamente a partir de draft

---

7. Draft Creation Strategies (A DEFINIR)

Possíveis pontos de criação:

- manual (dashboard)
- ao iniciar resposta
- ao receber réplica (pré-criação de tréplica)

Requisito:

- deve ser idempotente

---

Non-Goals

- não é um produto comercial
- não coleta dados além de email
- não possui autenticação por senha
- não utiliza OAuth

---