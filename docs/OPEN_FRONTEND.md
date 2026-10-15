# PROTEA — Frontend aberto e ecossistema de aplicações

**STATUS: PROPOSTA EM DISCUSSÃO — Fase 0.** Este texto não aprova stack, licença, políticas de acesso, direitos de uso, custos ou cronograma. Depende de discussão com o grupo fundador e, quando existir, com a comunidade de bandas. Ver [Visão](../VISION.md) e [Governança](../GOVERNANCE.md).

> **O PROTEA não precisa ser apenas um aplicativo de streaming. Pode ser uma infraestrutura musical aberta sobre a qual pessoas e comunidades constroem diferentes interfaces.**
>
> A música pertence a quem a cria. A plataforma deve pertencer à comunidade. A interface não precisa ser única.

## 1. Objetivo e princípios

Construir uma plataforma musical comunitária, sem fins lucrativos e de código aberto, em que bandas independentes publiquem músicas, álbuns e conteúdos e participem das decisões. O aplicativo oficial será **um cliente**, não o centro obrigatório do sistema. Desenvolvedores poderão propor players alternativos, sites de bandas, interfaces acessíveis, experiências de descoberta e integrações com IA.

Princípios propostos:
- **Desacoplamento:** o backend não depende do frontend oficial; os clientes consomem contratos públicos e versionados.
- **Interoperabilidade:** múltiplos clientes podem consultar o mesmo catálogo, respeitando permissões.
- **Continuidade:** outros clientes podem sobreviver à descontinuação do aplicativo oficial.
- **Acessibilidade e colaboração:** design system, documentação, testes e contribuições revisadas.
- **Autonomia das bandas:** abertura de código e metadados não transfere direitos sobre gravações ou publicações.
- **Sustentabilidade:** tráfego, armazenamento, distribuição e abuso têm custos; políticas devem ser transparentes e deliberadas.
- **Descentralização progressiva:** blockchain, IPFS e P2P não são pré-requisitos do MVP.

## 2. Arquitetura conceitual

```text
                      ECOSSISTEMA PROTEA
          ┌───────────────────┴───────────────────┐
          │                                       │
    CLIENTES OFICIAIS                      CLIENTES COMUNITÁRIOS
    Web/PWA · mobile                       players · sites · apps · IA
          │                                       │
          └───────────────────┬───────────────────┘
                              │
                        API PROTEA v1
                  catálogo · identidade · comunidade
                       reprodução · permissões
                              │
                       SERVIÇOS BACKEND
                     banco · cache · storage
```

O banco, cache e storage serão escolhidos posteriormente. PostgreSQL, Redis e soluções de objetos são **candidatos**, não decisões. A API deve distinguir **metadados públicos**, **dados privados** e **acesso aos arquivos de áudio**.

## 3. Aplicativo oficial multiplataforma

**Hipótese preferencial para avaliar no MVP:** Angular + Progressive Web App (PWA), com experiência responsiva e instalável quando suportada. Uma base web serviria navegadores de desktop e dispositivos Android, iOS/iPadOS, Windows, Linux e macOS, sem exigir cinco aplicações independentes.

**Limites a validar em dispositivos reais:** reprodução em segundo plano, controles de mídia do sistema, interrupções de áudio, funcionamento offline, downloads, instalação e comportamento específico de navegadores. PWA não equivale automaticamente a aplicativo nativo.

Evolução condicionada à demanda:
1. **Web/PWA:** experiência funcional e acessível.
2. **Mobile:** avaliar Capacitor para integração nativa Android/iOS, se necessário.
3. **Desktop:** avaliar Tauri ou Electron somente se houver benefício demonstrado.

### Áreas propostas para o MVP

| Área | Experiência |
| --- | --- |
| Descobrir | Busca, bandas, álbuns, faixas, lançamentos e recomendações editoriais |
| Player | Reprodução, fila, controles e favoritos |
| Comunidade | Feed, posts e perfis de bandas |
| Minha biblioteca | Bandas seguidas e playlists |
| Área da banda | Publicar e administrar músicas, álbuns e posts conforme permissões |

A UX/UI pode ser desenvolvida colaborativamente em Figma, com tokens, componentes reutilizáveis, estados de interface, responsividade, navegação por teclado e diretrizes de acessibilidade. A equipe técnica e colaboradores implementariam e revisariam o frontend. A identidade visual permanece em discussão.

## 4. API aberta não é acesso irrestrito

A proposta separa três níveis de API. **Rotas abaixo são exemplos de contrato futuro, não endpoints existentes.**

### 4.1 Descoberta pública (sem login)

```http
GET /api/v1/bands
GET /api/v1/bands/:id
GET /api/v1/albums
GET /api/v1/tracks
GET /api/v1/posts
GET /api/v1/search?q=hardcore
```

Retornar apenas conteúdos **publicados e públicos**, com paginação, filtros, documentação e limites de uso. Exemplo ilustrativo:

```json
{
  "id": "track_123",
  "title": "Resistência",
  "band": { "id": "band_456", "name": "Banda Exemplo" },
  "duration": 192,
  "artworkUrl": "https://exemplo.invalid/capa.jpg",
  "playbackAvailable": true
}
```

Metadados públicos não autorizam baixar, redistribuir ou treinar modelos com os arquivos de áudio. Visibilidade pública de posts, letras, imagens e dados pessoais deve ter regras específicas.

### 4.2 Recursos do ouvinte autenticado

```http
GET  /api/v1/me
GET  /api/v1/me/playlists
POST /api/v1/me/playlists
POST /api/v1/tracks/:id/playback
POST /api/v1/bands/:id/follow
```

Ações pessoais exigem identidade, autorização e verificação de titularidade.

### 4.3 Gerenciamento da banda

```http
POST   /api/v1/bands/:id/tracks
PATCH  /api/v1/tracks/:id
DELETE /api/v1/tracks/:id
POST   /api/v1/bands/:id/posts
```

Somente membros autorizados da banda podem alterar seu conteúdo. Ter um token de usuário **não** concede poderes de publicação em nome de uma banda.

## 5. Login e autorização para clientes independentes

**Proposta de evolução:** OAuth 2.0 + OpenID Connect; para aplicativos públicos, Authorization Code com PKCE. O cliente externo **não deve receber a senha** do usuário.

```text
Pessoa abre player alternativo
  → player redireciona para autenticação PROTEA
  → pessoa autentica e autoriza permissões específicas
  → cliente recebe acesso limitado
  → API valida token, escopos e titularidade a cada operação
```

Escopos ilustrativos: `catalog:read`, `playback:stream`, `playlists:read`, `playlists:write`, `profile:read`. Definir separadamente permissões sensíveis de banda. Prever tokens de curta duração, rotação/armazenamento seguro quando aplicável, revogação de clientes, rate limits, auditoria e proteção contra abuso. Não distribuir tokens permanentes manualmente. O desenho da API deve prever múltiplos clientes desde o começo, ainda que o provedor de autorização para terceiros venha depois.

## 6. IA e ferramentas externas

Uma API documentada pode permitir que assistentes encontrem bandas e lançamentos ou, com autorização, consultem o feed pessoal. Exemplo de uso: “Encontre bandas brasileiras independentes de hardcore com lançamentos recentes”.

Integrações candidatas:
- **OpenAPI:** contrato verificável por humanos e ferramentas.
- **SDK TypeScript:** cliente de referência para desenvolvedores.
- **OAuth/OIDC:** autorização delegada de acesso privado.
- **Servidor MCP opcional:** ferramentas de leitura como `search_bands`, `search_tracks`, `get_band_profile`, `get_band_posts`, `get_new_releases` e `get_my_followed_bands`.

O MCP, se adotado, deve usar a **mesma API e os mesmos controles de acesso**, nunca acesso direto ao banco. Começar com leitura; qualquer escrita exige escopos específicos, consentimento e controles adicionais.

**Acesso por IA não é consentimento para treinamento de modelos.** A comunidade e os titulares dos direitos precisam decidir regras para indexação, coleta automatizada, uso de letras, imagens, gravações e treinamento.

## 7. Streaming em aplicativos de terceiros

Fluxo conceitual:

```http
GET /api/v1/tracks/123

POST /api/v1/tracks/123/playback
Authorization: Bearer <access_token>
```

Após verificar direito de acesso, a API poderia emitir URL de reprodução temporária ou outro mecanismo controlado. Credenciais do armazenamento não devem ser expostas aos clientes. URLs temporárias **não impedem cópia de áudio**; devem ser acompanhadas de políticas de uso, controles de abuso e avaliação de risco.

Questões que precisam de decisão coletiva:
- Reprodução gratuita e anônima, mediante cadastro ou assinatura?
- Como custear banda, tráfego, CDN, armazenamento e apps independentes?
- Como contabilizar reproduções sem permitir fraude?
- Há downloads/offline? Em quais condições e com quais direitos?
- Quais limites de requisições e políticas para clientes externos?

**Hipótese de produto a discutir:** catálogo, perfis e posts públicos; regras de reprodução decididas separadamente. Não presumir que o modelo de financiamento já está aprovado.

## 8. Código aberto, contribuição e governança

**Frontend open source** e **API aberta** são decisões distintas:
- Código aberto permite estudar, contribuir e criar derivados nos termos da futura licença.
- API aberta permite criar clientes sem reutilizar o frontend oficial, respeitando contratos e políticas de acesso.

Publicar exemplos, guias de contribuição, testes, critérios de revisão e documentação do design system. Contribuições passam por revisão de segurança, acessibilidade, qualidade e alinhamento com a governança. A **licença do código ainda não foi escolhida**; nenhuma licença de software concede automaticamente direitos sobre músicas e outros conteúdos das bandas.

A arquitetura não deve transformar o mantenedor da API ou o responsável pelo frontend em autoridade permanente sobre regras econômicas ou culturais. Ver [GOVERNANCE.md](../GOVERNANCE.md).

## 9. Sequência sugerida, sem prazo aprovado

| Etapa | Entrega | Prioridade proposta |
| --- | --- | --- |
| 1 | Contrato/versionamento da API; separação público/privado; identidade e permissões | Alta |
| 2 | Protótipo UX/UI e aplicativo Web/PWA: catálogo, perfis, player, comunidade | Alta |
| 3 | OpenAPI pública, documentação, SDK TypeScript e cliente de exemplo | Média |
| 4 | OAuth/OIDC para terceiros, servidor MCP opcional, empacotamento mobile/desktop | Posterior |

**Critérios de validação antes de ampliar:** testes de player em dispositivos reais, controle de acesso entre bandas, experiência acessível, custo de streaming, documentação reproduzível por desenvolvedores externos e avaliação de segurança.

## 10. Decisões em aberto

- Stack do frontend e do backend; hospedagem, banco e storage.
- Licença do software e licenças/termos de uso do conteúdo musical.
- Política de reprodução, assinaturas, acesso anônimo e downloads.
- Políticas para aplicativos externos, rate limits e financiamento da infraestrutura.
- Escopos de autorização, registro de clientes e regras para publicação por terceiros.
- Uso de conteúdo por IA, indexação e treinamento.
- Identidade visual, UX, critérios de acessibilidade e prioridades de produto.
- Processo de aprovação das propostas pela comunidade.

## 11. Visão cultural

> O PROTEA não é apenas um aplicativo. É uma comunidade musical aberta.
>
> A plataforma oferece infraestrutura, mas não determina uma única maneira de descobrir, apresentar ou compartilhar música.
>
> Qualquer pessoa pode contribuir com novas experiências, respeitando os direitos das bandas e as regras construídas coletivamente.
>
> **A música é de quem faz. A plataforma também.**

---

*Documento conceitual de Fase 0. Nenhuma tecnologia, licença, política ou regra aqui descrita foi aprovada pela comunidade.*
