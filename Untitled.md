# CONTEXT

## Objective
[1-2 frases sobre o produto]
- Plataforma editorial política focada no debate sistematizado;
- Versão digital e moderna das clássicas revistas de debates;

## Core Concepts
[entidades principais com definição clara]
- 3 tipos de conteúdos: Ensaios, Traduções e Editoriais;
- Essays: a tribuna de debates, são divididos em 3 níveis (tribuna, réplica e tréplica). Tribunas podem ter várias réplicas, mas cada réplica pode ter apenas uma tréplica. São atreladas a um author (ou seja, a autoria se dá pelo pseudônimo registrado);
- Translations: Traduções.  Não possuem hierarquia, são atreladas a um original_author (ex. Karl Marx) e tem um campo customizável de nome do tradutor, por mais q a tradução seja atrelada a um author, seu pseudônimo não é exibido neste caso;
- Editorials: Publicações do coletivo. Não possuem hierarquia e não possuem um author atrelada a ele. São publicadas por authors com capacidade de moderação;
- Authors: são os "usuários" da plataforma. Possuem um ID, um pseudônimo (name), são atrelados a um e-mail e podem ou não ter capacidade de moderação (can_moderate);
- Drafts: Rascunhos de publicação.
- Submisisons: Submissão de um texto. Atrelada a um draft_id
- Moderation Tokens: tokens gerados para cada author moderador sempre que uma submission é enviada, eles tem data de expiração e possuem uma race condition (o primeiro que aprovar ou reprovar uma submission, consome o seu token e expira os dos demais)
## Invariants (VERY IMPORTANT)
[regras que NUNCA podem ser quebradas]
### Auth
- email é identidade única do author
- não existe autenticação sem os stateless tokens (magic link)
- todo token deve ser validado com Zod
- tokens tem purpose (magic link e session)
- o evento de confirmAccess é responsável por:
	- validar o token
	- resolver o author
	- criar sessão
	- resolver o redirect
#### Intent
- todo fluxo pós-login é definido pelo intent
- intent deve ser válido no momento da geração do token
- intent sempre vem do backend
- access_to_dashboard; reply_to_essay; resume_draft
### Author
- getOrCreateAuthor é indempotente
- name não define identidade
### Draft
- pertence a apenas um único author
- draft pode existir antes de um login
- draft pode ser acessado diretamente pelo intent resume_draft
### Submission
- toda publicação, exceto editorial, vem de uma submission
- apenas submissions aprovadas geram uma publicação
- é imutável após aprovação
## Flows
[resumo dos fluxos principais]

## Architecture
[como o sistema é organizado]

## Non-Goals
[o que o sistema NÃO faz]