# 03 — Modelo de convites

Especificação da regra de entrada e das sanções. É a parte mais delicada do app: define quem entra, quanto o cenário cresce por mês e o que acontece quando alguém erra na escolha.

Toda a lógica descrita aqui é **do servidor e transacional**. Nada disso pode morar no cliente.

---

## 1. Entidades

```
banda        id · nome · cidade · estilo · links · email_login · estado · padrinho_id · criada_em
convite      id · emissor_id · email_destino · token · criado_em · expira_em · estado · aceito_por
sancao       id · banda_id · tipo · motivo · aplicada_por · aplicada_em · resposta · respondida_em
```

**Estados da banda:** `convidada` → `quarentena` → `ativa` → `com_convite`, além de `em_revisao` e `banida`.

**Estados do convite:** `pendente` · `aceito` · `expirado` · `revogado` · `cancelado`.

---

## 2. O convite

| Propriedade | Regra |
| --- | --- |
| Natureza | Nominal: preso ao e-mail para o qual foi enviado |
| Uso | Único. Não existe link repassável |
| Validade | 14 dias corridos |
| Expirou | Estado vira `expirado` e **a cota volta para o emissor** |
| Revogação | O emissor pode revogar enquanto estiver `pendente` |
| Registro | Grava emissor, destino, data e a aresta na árvore |

---

## 3. Do convite recebido ao direito de convidar

| Estado | Como se chega | O que pode fazer |
| --- | --- | --- |
| `convidada` | Recebeu o e-mail nominal | Aceitar em até 14 dias |
| `quarentena` | Aceitou e criou o perfil | Publica, mas **só evento com data ≥ 48h**. Zero convites |
| `ativa` | Publicou um rolê que **aconteceu** (data passou, evento não cancelado) | Publica sem trava de data. Ainda zero convites |
| `com_convite` | Junto com a transição para `ativa`, ganha o **primeiro convite** | Convida |

**Acúmulo depois do primeiro:** +1 convite a cada **60 dias** de conta ativa, com **teto de 4 simultâneos**. Convite parado não vira estoque: chegou ao teto, para de acumular.

> Os 60 dias e o teto de 4 são pontos de calibragem, não dogma. Se a corrente estagnar, afrouxa; se crescer rápido demais, aperta. O que não muda é o convite ser **conquistado e escasso**.

---

## 4. Freios independentes

1. **Escassez** — o convite é conquistado, não dado na entrada.
2. **Lentidão** — quarentena de 48h nas publicações até o primeiro rolê acontecer.
3. **Teto agregado (torneira)** — limite global de novas contas por mês, definido pelo admin. Passou do teto, o convite fica em fila em vez de ser recusado.
4. **Profundidade** — limite de níveis da árvore, configurável. Ampliar é decisão consciente.

Qualquer um dos quatro segura sozinho um surto pontual.

---

## 5. Responsabilidade em cadeia

A aresta da árvore não é decorativa: é o registro de quem respondeu por quem.

| Evento | Consequência para a banda padrinha |
| --- | --- |
| **1º** afilhado banido | Perde o direito de convidar **imediatamente**. Convites `pendentes` viram `cancelado`. Motivo registrado na árvore. |
| **2º** afilhado banido | Conta vai para `em_revisao`: continua publicando os próprios rolês, sob análise. |
| **3º** afilhado banido | Banimento da própria conta. O admin recebe a opção de **podar a subárvore**. |

**Poda:** remove, numa operação, toda a subárvore criada por uma conta. É a ferramenta para quando alguém abre a porta para um grupo hostil — resolve em minutos o que a moderação caso a caso levaria semanas.

⚠ Isso vai gerar treta real entre gente do cenário. É o preço combinado, e está escrito no pacto que a banda aceita **antes** de criar a conta.

---

## 6. Quem pune: três instâncias

Ditadura na emergência, júri no resto. A punição deixa de ser ato de uma pessoa e vira processo — com o admin como fusível, não como única via.

### 6.1 As instâncias

| Instância | Pode o quê | Sem passar por |
| --- | --- | --- |
| **Admin** | Banir a qualquer momento, sem processo. Derrubar decisão de júri. | Tudo |
| **Bandas veteranas** (triagem) | Arquivar a denúncia, banir na hora em caso flagrante, ou mandar a júri. | Júri |
| **Júri de bandas** | Decidir o mérito: nada · advertência · perda do direito de convite · banimento. | — |

O atalho da triagem (**banir na hora**) tem trava: exige um número mínimo de bandas de acordo e é reversível pelo admin durante alguns dias. Sem isso, "banir na hora" é poder pessoal com outro nome.

### 6.2 O rito

1. **Denúncia.** A banda Y descreve o caso: o que houve, onde e quando. Denúncia sem caso concreto não anda.
2. **Triagem.** As veteranas leem e arquivam, banem (flagrante) ou mandam a júri.
3. **Defesa.** A acusada recebe a denúncia inteira e tem prazo para dar a versão dela.
4. **Júri.** As bandas votam.
5. **Registro.** Decisão e motivo ficam visíveis para todas as bandas. Nunca para o público.

Denúncia, defesa e votação acontecem longe do público. Só a decisão final é registrada, e só para bandas.

⚠ A escada em cadeia da seção 5 **não passa por júri**: o júri decide se houve falta; a perda do convite pela banda padrinha é mecânica e automática.

### 6.3 O desenho do júri — em aberto

| Composição | A favor | Contra |
| --- | --- | --- |
| Quem estava no mesmo rolê | Viram acontecer | São parte, amigas ou rivais; rolê pequeno pode não ter ninguém além dos envolvidos |
| Bandas da mesma região | Conhecem o espaço, a cena e o histórico | É onde moram as rixas; a panelinha decide |
| Sorteio fora da região | Não tem rabo preso | Julga no escuro, sem contexto |
| Misto: depõe quem viu, vota quem é de fora | Junta contexto e imparcialidade | Mais lento e mais trabalhoso de operar |

Parâmetros ainda sem decisão:

- **O que define uma banda "veterana".** Nível na árvore é tempo de chegada, não é juízo: do jeito mais óbvio, as primeiras bandas viram juízas permanentes por acidente de ordem. Alternativas: tempo de casa, número de rolês realizados, eleição pelas bandas da região.
- Tamanho do júri e quórum · maioria simples ou dois terços · prazo de voto e critério de desempate.
- Denúncia assinada ou anônima.
- Se a decisão pode ser revista, e por quem.
- Quantas veteranas de acordo o "banir na hora" exige, e por quantos dias o admin pode reverter.

⚠ Risco de fundo, comum a todos os desenhos: cenário pequeno, todo mundo se conhece, e o júri vira tribunal de rixa. O poder do admin de derrubar decisão existe como fusível enquanto o processo não prova que funciona — não como ideal.

---

## 7. A válvula contra panelinha

Convite fechado é ótimo contra zoeira e péssimo para alcance: a banda fora da rede de quem já está dentro nunca entraria.

```
[qualquer pessoa, sem conta]
      "conheço uma banda que devia estar aqui"
      nome · cidade · link · contato
                 │
                 ▼
      [fila de indicação do admin]
                 │  <aprovar>
                 ▼
      [convite enviado] — padrinho é o admin
```

A indicação **nunca** entra sozinha. O controle continua com o admin; o que muda é que a porta não é mais só "quem eu já conheço".

---

## 8. O que ainda não está decidido

- Todo o desenho do júri (seção 6.3).
- O que acontece com a conta quando a pessoa dona do e-mail sai da banda.
- Se a poda apaga ou apenas suspende os eventos publicados pela subárvore.
