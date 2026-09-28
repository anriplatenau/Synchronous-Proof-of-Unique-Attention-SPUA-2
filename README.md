# Synchronous Proof-of-Unique-Attention (SPUA)2

**Status:** Conceptual / research draft. Not a finished protocol — includes a known unresolved attack vector (see [Limitations](#limitations)).

## Problem Statement

Design an identification scheme for a mobile polling application satisfying **"one person — one opinion, one vote,"** without relying on:

- Government-issued ID / passport
- Biometric data (face, fingerprint, iris)
- Phone number / SMS verification

### Formalization

Let:
- `U` — the set of physical persons
- `D` — the set of app installations (devices)
- `S` — the set of secrets/keys issued by the app

Each installation `d ∈ D` receives a secret `s ∈ S` upon setup. The goal is to construct a relation `R ⊆ S × S` such that `(s₁, s₂) ∈ R` iff both secrets were issued to the same person `u ∈ U` — allowing the system to collapse duplicate identities to one vote — **without any external anchor** tying `s` to `u`.

### Core impossibility result

A secret generated locally at installation, with no external anchor, carries **zero information about the identity of the holder**. Formally, the graph on `S` induced by "same person" edges is **not derivable** from locally-generated secrets alone — there is no signal in the system that could produce such an edge. This is not a design gap; it is a structural property of the information available to the system.

Any scheme without an external uniqueness anchor (ID, biometrics, phone, stake) therefore cannot *provably* guarantee one-person-one-vote. What *can* be achieved is raising the cost/inconvenience of duplication to an acceptable threshold for the application's threat model.

## Proposed Scheme: Synchronous Unknown-Task Attestation

Rather than anchoring to identity, anchor to a physical constraint: **a single human attention span cannot be duplicated across two devices at the same instant, for a task unknown in advance.**

### Protocol flow

1. **Enrollment notice.** New users are notified in advance: "On [date], you will receive a message specifying a test time. Keep notifications on."
2. **Pre-alert.** 5 minutes before the test window, a push notification is sent: "The test will begin shortly. Prepare."
3. **Task reveal.** At the exact scheduled time `t`, an unknown, freshly-generated task/game is revealed simultaneously to all participants. The task requires continuous, real-time human engagement (perception, reaction, decision-making) for its full duration.
4. **Scoring.** A participant who reaches the required score within the time window is issued a cryptographic key `k_d`, usable as their voting credential.
5. **Uniqueness enforcement.** Because the task is unknown until `t` and requires sustained real-time attention, it cannot be pre-scripted, solved by emulator farms, or completed asynchronously on a second device by the same attention stream.

### What this defends against

| Attack vector | Defended? | Why |
|---|---|---|
| Bot / scripted automation | ✅ Yes | Task is unknown in advance; no pattern to script against |
| Emulator farms | ✅ Yes | Requires genuine real-time human perceptual response |
| Asynchronous duplication ("play on device 1, then device 2 later") | ✅ Yes | Fixed synchronous window; task requires continuous engagement |
| **Delegation (asking a second physical person to complete the task on a second device)** | ❌ **No** | See Limitations |

## Limitations

The scheme proves **"unique, continuous human attention occurred during window `t`, per device."** It does **not** and structurally **cannot** prove **"the attention on device 1 and the attention on device 2 belonged to the same, or to different, physical persons."**

The system only ever observes actions through a screen — it never observes the person. A second person (friend, family member, hired participant) can complete the task on a second device in good faith, satisfying every check the protocol performs. This is not a bug fixable within the current design; it is the same fundamental problem as Sybil resistance in general, reduced to its last remaining form.

**Mitigations that reduce (but do not eliminate) this risk:**

- **Behavioral fingerprinting** — compare reaction-time and input-pattern signatures against a historical baseline for the account. Requires prior play history; probabilistic, not conclusive; defeated by a sufficiently practiced or motivated stand-in.
- **Economic/social cost layering** — combine with invite-limited registration (web-of-trust) so that recruiting duplicate "players" requires spending trust capital, not just asking a favor.
- **Escalating stakes** — make the value of a single vote low enough that recruiting a second person is not worth the effort, for the application's specific context.

None of these convert the guarantee from probabilistic to cryptographic. Any deployment of this scheme should treat "one person, one vote" as a **statistically enforced norm**, not a proven invariant — and should disclose this honestly to stakeholders relying on poll integrity.

## Protocol Extension: Three Independent Problem Layers

The scheme decomposes into three distinct problems, each requiring its own defense. It is important not to conflate them — a mechanism that solves one does not solve the others.

```
Layer A — Identity uniqueness (Sybil resistance)
  "Is this the same physical person as another credential holder?"
  → Addressed by: SPUA synchronous unknown-task attestation
  → Residual gap: delegation (see Limitations above) — UNRESOLVED

Layer B — Credential integrity (anti-laundering / anti-resale)
  "Can a completed test's credential be stockpiled, sold, or reused later?"
  → Addressed by: seamless test-to-vote pipeline (below) — RESOLVED for this vector

Layer C — Response authenticity (anti-coaching)
  "Does the recorded answer reflect the respondent's genuine values,
   or a rehearsed/instructed position?"
  → Addressed by: indirect psycholinguistic questioning (below) — PARTIALLY MITIGATED
```

### Layer B: Seamless test-to-vote pipeline

**Mechanism:** the moment a participant passes the synchronous test, they are moved immediately — with no pause, no separate screen exit, no delay — into the actual poll or vote. The poll's questions may themselves be interleaved with, or continue directly from, the test's task flow. No standalone, transferable token is ever exposed as a discrete artifact the user could hand off.

**What this fixes:** it closes the credential-laundering vector — collecting one-time keys across multiple sessions/devices and redeeming or reselling them later, separately from the moment of attention-proof. Fusing issuance and consumption into a single uninterrupted session removes the window in which a credential could be extracted and transferred.

**What this does not fix:** it does not touch Layer A. A participant can still have someone else sit through the entire fused test-and-vote session on their behalf. Layer B closes a *logistics* attack (batch harvesting of credentials), not the *delegation* attack (a different physical person completing the whole flow).

### Layer C: Indirect psycholinguistic questioning

**Mechanism:** instead of asking a direct question ("do you support candidate A or B"), the poll opens with items establishing a moral/values profile, then presents concrete real-world facts and asks the respondent to judge the actions or inaction of parties involved — never naming the political conclusion directly. Questions are authored by psycholinguists to elicit binary (yes/no) responses, and each carries a strict time limit (e.g., 5 seconds), designed to bypass deliberate impression-management and surface implicit attitudes — methodologically adjacent to implicit-association testing.

**What this fixes:** it raises the cost of **coached delegation** — a stand-in cannot simply be told "vote for A." To answer convincingly, they would need to actually share (or have memorized in granular, internally consistent detail) the account owner's underlying values across dozens of indirectly-linked items, under time pressure, with no opportunity to consult anyone. This is a genuine, non-trivial increase in delegation cost for the "paid stranger" attack profile.

**What this does not fix — read carefully:** this layer defends a **different variable** than identity. It increases `P(the recorded answer reflects the respondent's true underlying values)`. It does **not** increase `P(the respondent is the same physical person who passed the test)`. If the delegate is a friend, family member, or ideological peer who genuinely shares the account owner's values — which is the common case for "vote as I tell you" among people who already agree — the psycholinguistic battery will be answered sincerely and will pass, while the vote is still cast by the wrong physical person. This mechanism defends **response authenticity**, not **identity uniqueness**; it should not be presented to stakeholders as closing the delegation gap in Layer A, only as independently valuable for a separate reason (reducing coached/rehearsed answers from whoever the actual respondent is).

## Comparison to Alternative Approaches

| Approach | Anchors identity via | Sybil-resistant? | Requires external ID/biometrics/SMS? |
|---|---|---|---|
| Zero-knowledge proof of uniqueness (Semaphore) | Pre-issued credential | Only re-use of *same* credential | Yes, at issuance |
| Blind signatures | Pre-issued credential | Only re-use of *same* credential | Yes, at issuance |
| Biometric proof-of-personhood (World ID) | Iris/face hash deduplication | Yes, global | Yes (biometrics) |
| Web-of-trust / invite graphs | Social vouching | Statistical only | No |
| **SPUA (this scheme)** | Synchronous unique attention | Statistical only; delegation unresolved | No |

## Open Research Question

Is there a class of tasks that can distinguish "the account owner completed this" from "someone the account owner delegated to completed this," without biometric liveness detection? This remains open. Candidate directions worth investigating:

- Long-horizon behavioral consistency modeling (requires substantial account history)
- Game-theoretic cost structures that make delegation irrational relative to abstaining
- Hybrid schemes accepting a minimal external anchor (e.g., single SIM binding) purely for deduplication, combined with ZK anonymization at the voting layer

Contributions and critiques welcome via issues/PRs.

---

# Синхронное доказательство уникального внимания (SPUA)

**Статус:** Концептуальный/исследовательский черновик. Протокол не завершён — включает известный нерешённый вектор атаки (см. [Ограничения](#ограничения)).

## Постановка задачи

Разработать схему идентификации для мобильного приложения социальных опросов, реализующую принцип **«один человек — одно мнение, один голос»**, без использования:

- Государственного документа / паспорта
- Биометрических данных (лицо, отпечаток пальца, радужка)
- Номера телефона / SMS-верификации

### Формализация

Пусть:
- `U` — множество физических людей
- `D` — множество установок приложения (устройств)
- `S` — множество секретов/ключей, выдаваемых приложением

Каждая установка `d ∈ D` получает секрет `s ∈ S` при настройке. Цель — построить отношение `R ⊆ S × S`, такое что `(s₁, s₂) ∈ R` тогда и только тогда, когда оба секрета выданы одному и тому же человеку `u ∈ U` — что позволило бы схлопывать дублирующиеся идентичности в один голос — **без какого-либо внешнего якоря**, связывающего `s` с `u`.

### Ключевой результат о невозможности

Секрет, сгенерированный локально при установке без внешнего якоря, несёт **нулевую информацию о личности владельца**. Формально, граф отношений «один и тот же человек» на `S` **не выводим** из локально сгенерированных секретов — в системе нет сигнала, который мог бы породить такое ребро. Это не пробел в дизайне, а структурное свойство доступной системе информации.

Поэтому любая схема без внешнего якоря уникальности (документ, биометрия, телефон, ставка) не может *доказуемо* гарантировать «один человек — один голос». Достижимо лишь повышение стоимости/неудобства дублирования до приемлемого для модели угроз конкретного приложения уровня.

## Предлагаемая схема: аттестация через синхронное неизвестное задание

Вместо привязки к личности — привязка к физическому ограничению: **одно человеческое внимание не может быть продублировано на двух устройствах одновременно, для задания, неизвестного заранее.**

### Ход протокола

1. **Уведомление о регистрации.** Новые пользователи заранее уведомляются: «[дата] вам придёт сообщение с указанием времени теста. Не выключайте уведомления».
2. **Предупреждение.** За 5 минут до начала окна теста приходит push-уведомление: «Тест скоро начнётся. Приготовьтесь».
3. **Раскрытие задания.** В точное назначенное время `t` всем участникам одновременно раскрывается неизвестное, только что сгенерированное задание/игра. Задание требует непрерывного вовлечения в реальном времени (восприятие, реакция, принятие решений) на всём протяжении.
4. **Оценка.** Участник, набравший необходимое число очков в отведённое время, получает криптографический ключ `k_d`, используемый как voting-credential.
5. **Обеспечение уникальности.** Поскольку задание неизвестно до момента `t` и требует непрерывного внимания в реальном времени, его нельзя заранее заскриптовать, пройти на ферме эмуляторов или выполнить асинхронно на втором устройстве тем же потоком внимания.

### От чего это защищает

| Вектор атаки | Защищено? | Почему |
|---|---|---|
| Боты / скрипты | ✅ Да | Задание неизвестно заранее; нет паттерна для скрипта |
| Фермы эмуляторов | ✅ Да | Требует настоящей человеческой перцептивной реакции в реальном времени |
| Асинхронное дублирование («сыграть на устройстве 1, потом на устройстве 2») | ✅ Да | Жёсткое синхронное окно; задание требует непрерывного вовлечения |
| **Делегирование (просьба к другому физическому человеку пройти задание на втором устройстве)** | ❌ **Нет** | См. Ограничения |

## Ограничения

Схема доказывает **«уникальное непрерывное человеческое внимание имело место в окне `t`, для данного устройства»**. Она **не доказывает** и структурно **не может доказать** **«внимание на устройстве 1 и внимание на устройстве 2 принадлежали одному и тому же (или разным) физическим лицам»**.

Система наблюдает только действия через экран — она никогда не наблюдает самого человека. Второй человек (друг, родственник, нанятый участник) может добросовестно пройти задание на втором устройстве, удовлетворив всем проверкам протокола. Это не устранимый в рамках текущего дизайна баг, а та же фундаментальная проблема Sybil-устойчивости, сведённая к своей последней форме.

**Меры, снижающие (но не устраняющие) этот риск:**

- **Поведенческий фингерпринтинг** — сравнение времени реакции и паттернов ввода с исторической базовой линией аккаунта. Требует истории игр; даёт вероятностный, а не окончательный результат; обходится достаточно натренированным или мотивированным «дублёром».
- **Наслоение экономической/социальной стоимости** — совмещение с регистрацией по ограниченным приглашениям (web-of-trust), чтобы найм «дублирующих игроков» требовал траты доверительного капитала, а не просто просьбы об услуге.
- **Эскалация ставок** — снижение ценности одного голоса до уровня, при котором найм второго человека невыгоден в конкретном контексте приложения.

Ни одна из этих мер не превращает гарантию из вероятностной в криптографическую. При развёртывании схемы принцип «один человек — один голос» следует рассматривать как **статистически обеспечиваемую норму**, а не как доказанный инвариант — и честно раскрывать это стейкхолдерам, полагающимся на целостность опроса.

## Расширение протокола: три независимых уровня задачи

Схема распадается на три отдельные проблемы, каждая из которых требует своей защиты. Важно их не смешивать — механизм, решающий одну, не решает остальные.

```
Уровень A — Уникальность личности (Sybil-устойчивость)
  «Это тот же физический человек, что и у другого держателя credential?»
  → Решается: SPUA — синхронная аттестация неизвестным заданием
  → Остаточная дыра: делегирование (см. Ограничения выше) — НЕ РЕШЕНО

Уровень B — Целостность credential (защита от накопления/перепродажи)
  «Можно ли накопить, продать или переиспользовать credential от пройденного теста позже?»
  → Решается: бесшовный переход тест→голосование (ниже) — РЕШЕНО для этого вектора

Уровень C — Достоверность ответа (защита от «натаскивания»)
  «Отражает ли зафиксированный ответ подлинные ценности отвечающего,
   или заученную/подсказанную позицию?»
  → Решается: косвенные психолингвистические вопросы (ниже) — ЧАСТИЧНО СМЯГЧЕНО
```

### Уровень B: бесшовный переход от теста к голосованию

**Механизм:** в момент прохождения синхронного теста участник немедленно — без паузы, без отдельного выхода с экрана, без задержки — переводится непосредственно к самому опросу или голосованию. Вопросы опроса могут быть переплетены с заданием теста или напрямую продолжать его. Отдельный переносимый токен, который пользователь мог бы кому-то передать как дискретный артефакт, никогда не появляется в явном виде.

**Что это исправляет:** закрывает вектор «отмывания» credential — накопление одноразовых ключей за несколько сессий/устройств с последующим их использованием или перепродажей отдельно от момента прохождения теста. Слияние выдачи и использования в один непрерывный сеанс устраняет окно, в котором credential можно было бы извлечь и передать.

**Что это не исправляет:** уровень A это не затрагивает. Участник по-прежнему может попросить кого-то другого пройти весь слитный сеанс тест-плюс-голосование за него. Уровень B закрывает атаку на *логистику* (пакетный сбор credential), а не атаку через *делегирование* (прохождение всего потока другим физическим лицом).

### Уровень C: косвенные психолингвистические вопросы

**Механизм:** вместо прямого вопроса («вы за кандидата А или Б») опрос начинается с пунктов, определяющих морально-ценностный портрет, затем предъявляет конкретные реальные факты и просит оценить действия или бездействие причастных к ним лиц — никогда не называя политический вывод напрямую. Вопросы составлены психолингвистами так, чтобы допускать только бинарный ответ (да/нет), и каждый имеет строгий тайминг (например, 5 секунд), рассчитанный на то, чтобы обойти сознательное управление впечатлением и выявить имплицитные установки — методологически близко к тестам на имплицитные ассоциации.

**Что это исправляет:** повышает стоимость **натасканного делегирования** — дублёру недостаточно просто сказать «голосуй за А». Чтобы отвечать убедительно, ему пришлось бы реально разделять (или заучить в детализированном, внутренне согласованном виде) базовые ценности владельца аккаунта по десяткам косвенно связанных пунктов, под давлением времени и без возможности с кем-то посоветоваться. Это реальное, нетривиальное повышение стоимости делегирования для профиля атаки «нанятый незнакомец».

**Что это не исправляет — важно понимать точно:** этот уровень защищает **другую переменную**, не идентичность. Он повышает `P(зафиксированный ответ отражает подлинные ценности отвечающего)`. Он **не** повышает `P(отвечающий — тот же физический человек, что проходил тест)`. Если дублёр — друг, родственник или идейный единомышленник, реально разделяющий взгляды владельца аккаунта (типичный случай «голосуй как я скажу» среди уже согласных друг с другом людей), психолингвистическая батарея вопросов будет пройдена искренне и успешно — а голос при этом всё равно подаст не тот физический человек. Этот механизм защищает **достоверность ответа**, а не **уникальность личности**; его не следует представлять стейкхолдерам как закрывающий разрыв делегирования из уровня A — только как самостоятельно ценный по отдельной причине (снижение доли натасканных/заученных ответов от того, кто фактически отвечает).

## Сравнение с альтернативными подходами

| Подход | Якорь идентичности | Sybil-устойчивость | Требует внешний ID/биометрию/SMS? |
|---|---|---|---|
| Zero-knowledge proof of uniqueness (Semaphore) | Заранее выданный credential | Только против повторного использования *того же* credential | Да, на этапе выдачи |
| Blind signatures | Заранее выданный credential | Только против повторного использования *того же* credential | Да, на этапе выдачи |
| Биометрический proof-of-personhood (World ID) | Хеш радужки/лица, дедупликация | Да, глобально | Да (биометрия) |
| Web-of-trust / граф приглашений | Социальное поручительство | Только статистическая | Нет |
| **SPUA (данная схема)** | Синхронное уникальное внимание | Только статистическая; делегирование не решено | Нет |

## Открытый исследовательский вопрос

Существует ли класс заданий, способный отличить «владелец аккаунта прошёл это сам» от «владелец аккаунта делегировал прохождение», без биометрической детекции живости (liveness)? Вопрос остаётся открытым. Возможные направления:

- Моделирование долгосрочной поведенческой согласованности (требует значительной истории аккаунта)
- Теоретико-игровые структуры стоимости, делающие делегирование иррациональным по сравнению с воздержанием от голосования
- Гибридные схемы, принимающие минимальный внешний якорь (например, привязку к одной SIM-карте) исключительно для дедупликации, в сочетании с ZK-анонимизацией на уровне голосования

Предложения и критика приветствуются через issues/PR.
