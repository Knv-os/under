# 01 — Visão e princípios

## O problema

A informação sobre shows do cenário underground (metal extremo, punk, HC, crust, grind e derivados) vive hoje em:

- stories que somem em 24h;
- eventos de rede social que só aparecem para quem já segue a página;
- grupos de WhatsApp fechados;
- cartazes em poste.

Consequência: quem não está dentro da bolha certa não fica sabendo do rolê que acontece a 8km de casa. Bandas tocam para as mesmas 30 pessoas. Rolês colidem no mesmo sábado por falta de coordenação. Cancelamentos e mudanças de local não chegam a ninguém.

## O que o app é

Uma **agenda geolocalizada do cenário underground**, mantida pelas próprias bandas, consultável sem cadastro.

Abrir → permitir localização (ou digitar a cidade) → ver o que rola num raio de ~50km, hoje e nos próximos dias.

## O que o app **não** é

Isto é tão importante quanto o que ele é. Cada item abaixo é uma decisão de produto, não uma limitação técnica:

- **Não é rede social.** Sem feed infinito, sem curtida, sem seguidor, sem contador de presença pública, sem comentário aberto no evento.
- **Não tem algoritmo.** A ordenação é cronológica e por distância. Ninguém paga nem "engaja" para aparecer mais.
- **Não é rede de contatos.** Não existe perfil de público. Quem consome não tem conta.
- **Não tem publicidade.** Nem impulsionamento, nem destaque pago, nem parceria de marca.
- **Não é ferramenta de bilheteria.** Pode linkar para onde a venda acontece, mas não processa pagamento (ao menos na v1).
- **Não tem conta de espaço.** Local é dado dentro do evento, nunca um perfil com poderes.
- **Não funciona offline.** Descartado: o caso de uso real (saber o endereço a caminho do rolê) é resolvido pelo alerta das 10h, que leva o endereço na própria notificação.

## Quem tem conta

**Só banda.** Sem exceção.

- Coletivo, produtora, selo e espaço **não têm conta**. Quem organiza sem ser banda publica através de uma banda do próprio rolê.
- O evento pode **creditar** uma produção em texto, sem conta e sem poder de publicar ou editar.
- O local é um registro de dados dentro do evento (nome, endereço, acessibilidade), nunca uma conta.

## Princípios

1. **Consumir é livre e anônimo.** Nenhum cadastro, login ou rastreamento para ver a agenda — nem para salvar um rolê e receber o aviso (ver "Lembrete sem cadastro" abaixo).
2. **Publicar é por confiança.** Só entra a banda convidada por uma banda que já está dentro, e quem convida responde pelo convidado desde o primeiro caso.
3. **Localização não é dado de vigilância.** A coordenada do usuário é arredondada antes de sair do aparelho e nunca é armazenada associada a uma identidade.
4. **A segurança do rolê vem antes da divulgação.** O organizador controla o quanto do endereço fica público e a partir de quando.
5. **Funciona no celular ruim e na internet ruim.** Baixo peso, dark-first e rápido em 3G — não com 80 MB de app.
6. **O cenário é dono dos dados.** Exportação aberta (`.ics`, JSON), link público por evento, nada de aprisionamento.
7. **Posicionamento explícito.** Antifascismo, antirracismo e política anti-assédio são regra de uso aplicável — não estética. Quem viola, sai, e o padrinho responde.

## Lembrete sem cadastro

O app promete "salvar o rolê" e "receber aviso" sem conta. Como isso funciona, em uma frase: **o que fica registrado é um código do aparelho, não uma identidade.**

- Salvar registra `{token de push do aparelho, id do evento}`. É o mesmo token que o Android (FCM) e o navegador (Web Push) já usam para notificação.
- Esse token identifica um aparelho, não uma pessoa. Não é reversível para e-mail, telefone ou conta. Trocar de aparelho ou limpar os dados do app zera tudo.
- O registro é **apagado depois que a data do evento passa**. Não vira base de dados de quem vai a quê.
- Se a pessoa recusar a permissão de notificação, o rolê fica salvo só no aparelho e o app avisa quando ela abrir. Nada é enviado ao servidor.

## Comentários sobre locais (só entre bandas)

Como o espaço não tem conta, quem dá a informação sobre ele são as bandas que tocaram lá.

- Visível **apenas para contas de banda logadas**. Nunca para o público, nunca na tela do evento, e sem rota pública que devolva esse conteúdo.
- **Assinado** pela banda autora. Qualquer banda cadastrada pode escrever.
- Estrutura: combinado cumprido (cachê/divisão) · estrutura e som · segurança e postura do espaço · relato livre.
- **Risco assumido:** o espaço não tem conta, logo não tem direito de resposta dentro do app. A assinatura obrigatória é o que segura o abuso; o admin pode remover; o canal externo de contestação é uma decisão em aberto.

## Métricas que importam (e as que não importam)

| Acompanhar | Ignorar |
| --- | --- |
| Eventos publicados por semana, por região | Usuários ativos diários |
| % de eventos com informação completa (endereço, horário, line-up) | Tempo de sessão |
| % de line-ups com todas as bandas confirmadas | Downloads |
| Cobertura: quantas cidades têm ao menos 1 evento/mês | Compartilhamentos |
| Cancelamentos comunicados pelo app antes do dia | — |
| Nº de contas banidas / nº de contas ativas | — |

Um app que o público abre 3 vezes por mês, por 40 segundos, e sempre acha o que procura, está funcionando perfeitamente.

## Público-alvo

- **Público (sem conta):** quem quer saber onde tem rolê perto, hoje ou no fim de semana.
- **Banda (com conta, por convite):** quem publica, monta line-up, mantém os eventos e comenta sobre locais.
- **Admin:** quem abre a corrente de convites, poda o que apodrece e pode banir sem processo — o fusível, não a única via. As punições passam por denúncia, triagem das bandas veteranas e júri (ver `02-userflows.md` §E.2 e `03-modelo-de-convites.md` §6).

## Nome

**PORÃO** — decidido. O repositório continua se chamando `under`.
