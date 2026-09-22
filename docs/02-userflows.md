# 02 — Userflows

Notação: `[tela]`, `<ação>`, `→` transição, `⚠` ponto de atenção.

Atores: **público** (sem conta), **banda** (conta por convite) e **admin**. Não existe conta de espaço, coletivo ou produtora — ver `01-visao-e-principios.md`.

---

## A. Público (sem conta) — o fluxo principal

Meta: da abertura do app até saber onde é o rolê em **menos de 15 segundos e sem digitar nada**.

```
[abertura]
   │  (nada de splash, nada de onboarding, nada de "crie sua conta")
   ▼
[permissão de localização]  ── negou / indisponível ──► [escolher cidade]
   │                                                        │
   └────────────────────────────────┬───────────────────────┘
                                    ▼
                          [AGENDA]  ◄──► [MAPA]   (mesmo conteúdo, dois modos)
                                    │
                                    ▼
                              [detalhe do rolê]
                                    │
      ┌─────────────┬───────────────┼───────────────┬──────────────┐
      ▼             ▼               ▼               ▼              ▼
  <como chegar>  <salvar>     <compartilhar>   <adicionar ao    <ouvir a
   (mapa ext.)   (avisos)      (link + cartaz)   calendário>      banda>
```

**[AGENDA]** — lista, é a tela padrão. Agrupada por dia: `HOJE` (destaque forte, "começa em 3h"), `AMANHÃ`, `ESTA SEMANA`, `DEPOIS`.

Cada item mostra, nessa ordem de peso visual: **data/hora → nome do rolê → cidade e distância → line-up resumido → valor**. Line-up mostra quem **confirmou**; banda ainda pendente aparece marcada como tal.

**[MAPA]** — MapLibre, estilo escuro e quase monocromático. Pin agrupado por local quando há mais de um rolê no mesmo endereço.

**Filtros** (persistidos localmente, nunca no servidor): raio (10 / 25 / 50 / 100 km), período, gênero, "só gratuitos", "só acessíveis".

⚠ **Sem resultado é o pior caso do app.** A tela vazia precisa oferecer saída: ampliar o raio automaticamente ("nada em 50km — o mais próximo é em Ponta Grossa, 120km") e um botão **"conheço uma banda que deveria estar aqui"**, que alimenta a fila de indicação do admin.

**Estados do evento que o público precisa ver:** `CANCELADO`, `ADIADO`, `LOCAL ALTERADO`, `ESGOTADO`. Sempre no topo do card, em vermelho, sem sutileza.

### A.1 — Salvar e ser avisado, sem conta

```
<salvar>
   │
   ├─ aceitou notificação  → servidor guarda {token do aparelho, id do evento}
   │                         → alerta às 10h do dia, com endereço
   │                         → aviso imediato em cancelamento / adiamento / mudança de local
   │                         → registro apagado depois que a data passa
   │
   └─ recusou notificação  → rolê fica salvo só no aparelho
                             → o app avisa quando a pessoa abrir
```

⚠ O token identifica **um aparelho, não uma pessoa**. Sem e-mail, sem login, sem perfil. É isso que permite ter lembrete sem furar o princípio 1.

---

## B. Banda — entrar na corrente

```
[e-mail de convite]  "A banda Fulana te chamou pro PORÃO. O convite vale 14 dias."
   │  <abrir link único>
   ▼
[quem te convidou]   mostra a banda padrinha e explica a responsabilidade mútua
   │  <continuar>
   ▼
[o pacto]            termo curto, em linguagem direta, não juridiquês:
   │                 o que é proibido, o que acontece com quem violar, quem decide,
   │                 e o fato de que a banda padrinha perde o convite no primeiro caso.
   │  <aceito>
   ▼
[link mágico por e-mail]   sem senha
   ▼
[perfil da banda]    nome, cidade, estilo, links (Bandcamp, YouTube, Instagram)
   ▼
[publique seu primeiro rolê]  → fluxo C
   ▼
[seus convites]      "você terá convite quando seu primeiro rolê acontecer"
                      → ver 03-modelo-de-convites.md
```

⚠ Convite **nominal, de uso único, preso ao e-mail, expira em 14 dias**. Não existe link repassável. Convite não aceito devolve a cota para quem chamou.

⚠ **Quarentena:** enquanto a conta não tiver um rolê realizado, só publica evento com data a partir de 48h. Custo baixo para quem é de verdade, freio real contra sabotagem de véspera.

---

## C. Banda — publicar um rolê e montar o line-up

O fluxo precisa caber em **4 telas e ~90 segundos**, porque quem publica está no ônibus, com 12% de bateria.

```
1 [quando]     data · hora de início · hora prevista de término (ou "até tarde")
2 [onde]       busca do local (reaproveita locais já cadastrados)
               + endereço público OU horário de liberação (opcional)
3 [quem]       line-up:
                 ├─ bandas cadastradas  → <convidar>  (estado: pendente)
                 └─ banda de fora       → texto simples, sem conta e sem convite
               + produção creditada em texto (opcional)
4 [como]       valor · idade mínima · gêneros · acessibilidade · selos
               · contato · link externo · cartaz (imagem)
   ▼
[pré-visualização]  exatamente como o público vai ver
   ▼
<publicar>  →  link público /e/{slug}  +  cartaz pronto pra story
```

### C.1 — Line-up: convite e pedido

O line-up se monta nos dois sentidos, e sempre fecha com as duas pontas de acordo.

```
CONVITE →                              ← PEDIDO
[organizadora] <convidar Fossa>        [Carniça] vê o rolê na agenda
        │                                     │  <pedir pra entrar>
        ▼                                     ▼
[Fossa recebe] notificação             [organizadora recebe] pedido
        │                                     │
   ┌────┴────┬──────────┐              ┌──────┴──────┐
   ▼         ▼          ▼              ▼             ▼
<aceitar> <recusar>  (sem resposta) <aceitar>    <recusar>
   │         │          │              │             │
 CONFIRMADO RECUSADO  PENDENTE      CONFIRMADO   (some, sem registro público)
```

- Estados: `pendente`, `confirmado`, `recusado`, `desistiu`, `pediu_entrada`.
- O **pedido** resolve o caso da banda em turnê: ela vê que tem rolê na cidade onde vai estar naquela data e se oferece, em vez de depender de conhecer alguém.
- ⚠ Pedido recusado **não aparece em lugar nenhum** — nem para o público, nem como recusa pública. Recusa não é humilhação.
- Quem organiza tem a palavra final nos dois sentidos.
- **Nenhuma banda aparece como confirmada sem ter aceitado.** O público vê a diferença entre "vai tocar" e "foi anunciada".
- ⚠ Convite de line-up só alcança **banda já cadastrada**. Banda de fora entra como texto. Isso é deliberado: o line-up não pode virar porta dos fundos da corrente de entrada.
- A banda organizadora responde pelo evento: é ela que edita, adia, cancela e muda o local.

### C.2 — Depois de publicado

Três ações de peso, sempre a um toque: **adiar**, **cancelar**, **alterar local**. Todas avisam quem salvou.

⚠ Alterações **não** apagam o evento: viram um aviso visível no card. Um evento cancelado continua na lista, marcado, até a data passar.

**Recorrência:** "repetir semanalmente até \<data\>" já na v1 — rolê fixo toda quinta é comum.

---

## D. Banda — comentar sobre um local

```
[local]  (aberto a partir de um evento ou pela busca de locais)
   │
   ├─ [comentários]  visível SÓ para conta de banda logada
   │      └─ cada item: banda autora · data · eixos · relato
   │
   └─ <escrever>
         ├─ combinado cumprido?      sim / parcial / não
         ├─ estrutura e som          nota + texto
         ├─ segurança e postura      nota + texto
         └─ relato livre
      → publicado ASSINADO pela banda
```

- Qualquer banda cadastrada pode escrever; a assinatura é obrigatória.
- ⚠ O espaço não tem conta e **não responde dentro do app**. O admin pode remover; o canal externo de contestação ainda não está definido (ver decisões em aberto).
- ⚠ Nunca aparece na tela do evento nem em nenhuma rota pública.

---

## E. Admin

```
[painel]
 ├─ [árvore de convites]   grafo: quem trouxe quem, com profundidade e datas
 │     └─ <revogar convite pendente>
 │     └─ <banir conta>  → oferece <podar subárvore>
 ├─ [torneira]             teto global de novas contas por mês/região
 ├─ [denúncias]            evento falso · banda com histórico de abuso ·
 │                         local inseguro · conteúdo fascista
 │                         ⚠ chega só ao admin — nunca vira linchamento público
 ├─ [fila de indicação]    bandas indicadas pelo público (válvula contra panelinha)
 │                         → <gerar convite>
 └─ [comentários de local] remoção de relato abusivo
```

### E.1 — Escada de punição

| Evento | Consequência para a banda padrinha |
| --- | --- |
| 1º afilhado banido | **Perde o direito de convidar na hora.** Convites pendentes são cancelados. Motivo fica registrado na árvore. |
| 2º afilhado banido | A conta entra em revisão. Continua publicando, sob análise. |
| 3º afilhado banido | Banimento da própria conta, com poda da subárvore oferecida ao admin. |

### E.2 — Quem pune: três instâncias

Ditadura na emergência, júri no resto.

```
[banda Y denuncia]  descreve o caso: o que houve, onde, quando
        │           (denúncia sem caso concreto não anda)
        ▼
[TRIAGEM · bandas veteranas]
   ├── arquiva
   ├── bane na hora  (só flagrante; exige mínimo de bandas de acordo
   │                  e é reversível pelo admin por alguns dias)
   └── manda a júri
        │
        ▼
[DEFESA]  a acusada recebe a denúncia inteira e tem prazo pra responder
        │
        ▼
[JÚRI]    bandas votam: nada · advertência · perda do convite · banimento
        │
        ▼
[REGISTRO]  decisão e motivo visíveis para as bandas. Nunca para o público.

[ADMIN]  ── corta qualquer etapa: bane sem processo e derruba decisão de júri.
            Atalho pro caso flagrante e pro erro coletivo.
```

- A escada da regra 2 (E.1) **não passa por júri**: o júri decide se houve falta; a consequência em cadeia é mecânica.
- Denúncia, defesa e votação acontecem longe do público. O app não organiza linchamento.

### E.3 — O desenho do júri está em aberto

Quatro composições possíveis, com o trade-off de cada uma:

| Composição | A favor | Contra |
| --- | --- | --- |
| Quem estava no mesmo rolê | Viram acontecer | São parte, amigas ou rivais; rolê pequeno pode não ter ninguém além dos envolvidos |
| Bandas da mesma região | Conhecem o espaço, a cena e o histórico | É onde moram as rixas; a panelinha decide |
| Sorteio fora da região | Não tem rabo preso | Julga no escuro, sem contexto |
| Misto: depõe quem viu, vota quem é de fora | Junta contexto e imparcialidade | Mais lento e mais trabalhoso de operar |

Também sem decisão:

- **O que define uma banda "veterana".** Nível na árvore é só tempo de chegada, não é juízo — do jeito mais óbvio, as primeiras bandas viram juízas permanentes por acidente de ordem. Alternativas: tempo de casa, número de rolês realizados, eleição pelas bandas da região.
- Tamanho do júri e quórum.
- Maioria simples ou dois terços.
- Prazo de voto e o que fazer no empate.
- Denúncia assinada ou anônima.
- Se a decisão pode ser revista, e por quem.

⚠ Risco de fundo, comum a todos os desenhos: cenário pequeno, todo mundo se conhece, e o júri vira tribunal de rixa. É por isso que o admin segue com poder de derrubar decisão — como fusível enquanto o processo não prova que funciona, não como ideal.
