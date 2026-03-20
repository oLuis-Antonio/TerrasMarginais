# CONTEXT

## Objective
- Plataforma editorial política focada no debate sistematizado;
- Versão digital e moderna das clássicas revistas de debates;

## Core Concepts
- 3 tipos de publicações: Ensaios, Traduções e Editoriais;
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
## Flows (implementados até o momento)
### Login e auth
1. ao entrar na página de "login", o usuário informa seu email, define o baseName (é o name do author ou o pseudônimo escolhido na hora com o gerador, caso seja um author novo) + o intent derivado da página pai -> isso é submetido a api/access/request
2. a api chama o requestAccessUseCase que chama a função generateMailToken do token service, cria um url de confirmação e manda um link de acesso com o mail service
3. o link de acesso chama a api/access/confirm que:

## Architecture
Arquitetura em camadas: front-end; backend dividido pela sessão de API (Astro) e domínio (src/lib); banco de dados
a sessão de domínio;

lib/access: controla o acesso dos authors
dividida em useCases -> controlAccess e requestAccess

lib/auth: controla a autenticação
possui o tokenService e cria o sistema de intent

lib/authors: controla o sistema de authors com o authorService
futuramente deve incluir o sistema para original_authors tbm

lib/mail: controla o sistema de emais; agnóstico de provider (pretendido usar o resend, mas pode ser acoplado a qualquer outro)

useCases -> orquestram os fluxos, chamam services e não acessam o infra
services -> encapsulam lógica reutilizável + acesso a dados
providers -> infra externa, só fazem I/O
routes (API) -> valida requests, chamam useCases, retornam responses

## Non-Goals
- Não é um produto comercial
- Não pretende coletar dados do usuário (apenas email)
- Tem direcionamento político claro