# Testing Guidelines for Streamwide SDK Conversation and Call Management

## 1. Introduction

This document serves as a comprehensive specification for testing the behavior of conversations, calls, and related entities provided by the Streamwide SDK. The entities described herein are objects created and managed by the SDK, which we interact with through the SDK API provided by Streamwide.
These entities are also stored and propagated by Streamwide's server infrastructure.

#### _Scope and Methodology_

The details provided in this document regarding entity properties, creation mechanisms, propagation rules, and management procedures are based on our empirical observations and analysis of SDK behavior. These insights have been gathered through:

- **Direct SDK API Usage**: High-level interactions with the SDK API to understand how conversations, calls, and messages are created and managed.
- **Log Analysis**: Examination of SDK logs to infer internal behavior and identify inconsistencies.
- **Code Analysis**: Review of deobfuscated SDK code to deduce implementation details where possible.

#### _Assumptions and Limitations_

It is important to note that our understanding of the SDK's internal workings is based on indirect observations and may not represent the complete or intended design. Several key limitations apply:

- **Inferred Behavior**: Many of the described procedures and rules are our best interpretations of how the SDK should function based on observed behavior, not official documentation.
- **Potential Defects**: The SDK exhibits numerous defects that we cannot fully trace to their root causes through code analysis alone. Our tests aim to identify these issues.
- **Undocumented Features**: Some SDK behaviors appear to be undocumented or not officially supported, yet are observable through API usage.

#### _Purpose of This Document_

This specification serves three primary purposes:

- **Documentation of Current Understanding**: To consolidate our knowledge of how conversations and calls work within the SDK.
- **Test Definition**: To define clear test scenarios (S₁, S₂) that will validate our understanding and identify discrepancies.
- **Issue Identification**: To highlight potential defects or unexpected behaviors that require Streamwide's attention.

#### _Next Steps_

Streamwide will be asked to:

- **Verify Assumptions**: Confirm whether our understanding of entity behavior and rules aligns with the intended SDK design.
- **Address Defects**: Correct any issues identified during test execution that deviate from expected behavior.
- **Provide Clarification**: Offer official documentation or explanations for behaviors that remain unclear.

This document represents our current state of knowledge and will evolve as we conduct tests and receive feedback from Streamwide.

---

## 2. Involved entities

1. #### Contact
    - A contact represents either a user or a group of users.

    - Users are identified by their `msisdn`, groups by their `groupId`.

        > **Note** - For uniformity purposes, when we use a SDK contact we assign the object a unique identifier, `__id`, which equals the `msisdn` for users and `group:<groupId>` for groups.

    - When a user belongs to a group, the group is called a **blue group** (or **pre‑arranged**); otherwise it is a **grey group**.

2. #### Conversation
    - There are three types of conversations:
        - **One‑to‑one conversation (type 1)** – used for personal chats and one‑to‑one calls. Only **two users** are involved.
        - **Broadcast conversation (type 2)** – used for broadcast messages, not to be confused with broadcast calls. Not involved in call management at all, so ignore this type of conversation for now.
        - **Group conversation (type 3)** – used when a user communicates with a **group** or with **more than one contact**.
    - A conversation is **not** a global, single entity stored in the server database.
    - For every participant the server creates a **distinct instance**, each its own unique `conversationID`.
    - Instance properties (name, type, participants, etc.) are computed **per user** (e.g., a one-to-one conversation's name is the recipient's name, a conversation with a pre-arranged group has the same name as the group).
    - When multiple recipients are involved (group conversation), all user instances share a common `groupId` property that links them.

3. #### Message
    - A message is **not** a global, single entity stored in the server database.
    - When a user sends a message in a conversation, the server replicates that message for **each other participant**.
    - Each replica is a **distinct object** with its own unique `id`.
    - Every message replica is linked to the conversation instance it belongs to via the `conversationId` property.
    - The `mediaType` property indicates the content type of the message (e.g., text, audio, etc.).

4. #### Call
    - A call between an initiator (the caller) and recipient users (the callees) possesses a `sessionId` property.

    - When multiple callees are involved, a property `sessionGroupId` is also attached to the object.

    - The call can be of different types. Here are the ones under test :

        | Name              | Symbol | Description                                                                                                                                                                                                                                                                                                                                       |
        | ----------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
        | Free call         | $AF$   | A one‑to‑one VoIP call between the current user and a single **user** contact. It is started with `STWVoipCallManager.startFreeCall(contact)` and represents a normal voice conversation. To call from a one-to-one conversation, we use the `startFreeCall(contact)` function of the conversation object.                                        |
        | Video Free call   | $AV$   | A one‑to‑one VoIP video call between the current user and a single **user** contact. It is started with `STWVoipCallManager.startVideoFreeCall(contact)` and represents a normal video conversation. We cannot launch this type of call from a conversation.                                                                                      |
        | PTT call          | $AP$   | A _Push‑To‑Talk_ VoIP call that can involve one or more contacts (any type). It is started with `STWVoipCallManager.startPttCall(contacts)`. To call from a conversation, we use the `startPTTCall(contacts)` function of the conversation object.                                                                                                |
        | Conference call   | $AC$   | A multi‑party VoIP call where several participants (any type of contact) can join simultaneously. It is started with `STWVoipCallManager.startConferenceCall(contacts)`.To call from a conversation, we use the `startConferenceCall(contacts)` function of the conversation object.                                                              |
        | Broadcast call    | $AB$   | A one‑to‑many VoIP call in which the caller broadcasts to multiple recipients (any type). The call is automatically accepted by the callee side and is started with `STWVoipCallManager.startBroadcastCall(contacts)`.To call from a conversation, we use the `startBroadcastCall(contacts)` function of the conversation object.                 |
        | Emergency call    | $AE$   | A VoIP call that involves the caller and the contacts (any type) listed in the caller **emergency profile**. After having configured the caller settings, the call can be started either with `STWVoipCallManager.startEmergencyCall(contacts)` or from a conversation with `STWEmergencyManager.startEmergencyCall(conversationName, contacts)`. |
        | Ambient listening | $AA$   | A special case of emergency calls, used for passive listening during emergency situations. It is started with `STWVoipCallManager.startAmbientListening(contacts)`. We cannot launch this type of call from a conversation.                                                                                                                       |

5. #### Call message
    - A call message is a **message** with an **audio** `mediaType`. This type of message always represents a specific **call**, identified by the message `messageUuid` property that equals the call `sessionId` property.
    - A call message represents a call log in a conversation. It carries the call information, like the type of the call, its direction (incoming or outgoing) and its duration (updated after the call ended).

---

## 2. Management rules

1. #### Conversation creation and propagation
    - When a user initiates a conversation, the SDK first creates a temporary conversation object locally, populating its fields (name, type, etc.).
    - It then sends a _create conversation_ request to the server. At this point, the only conversation instance that exists is the initiator’s one, and it will persist in that state until a message is sent in the initiator conversation.
    - The first message sent on that conversation triggers the server to _propagate_ the conversation for every other participant :
        - For each recipient, the server creates and stores a **distinct conversation instance** (with its own `conversationId`). For group conversations, the recipient's conversation instance is linked to the other instances via a shared `groupId`.
        - For each recipient, the server also creates a **distinct message replica** (with its own `id`). The recipient's message replica is linked to the recipient's conversation instance through `conversationId`.

2. #### Call creation and propagation
    - A caller can initiate a call by selecting one or more contacts (users or groups).
        1. The SDK first searches for an existing conversation between the caller and the callees.
            - If found, the call will be linked to that conversation.
            - If not, the SDK creates the conversation.
            - In certain situations (e.g. in case of emergency calls), the SDK should always create a new conversation, even if there is already an existing one. Thus the search is not executed and a new conversation is directly created.
        2. To link the call to the conversation, the SDK creates a call message that represents the call, and sends it in that conversation.
        3. As for any message sent in a conversation, the server replicates the call message for each participant of the conversation (i.e for each callee).
        4. Each call message replica carries the call information, some computed based on the user for whom the instance was created (e.g. is the call incoming or outgoing, etc.).
        5. Each user that received the call message can then join the call.

    - A call message can be sent in any existing conversation, even if the conversation already contains another call message. It allows the conversation to keep a history of the calls between the conversation's participants.

    - If a caller initiates a call by selecting recipients he already has a conversation with, the SDK finds that conversation and sends the call message in it. The call is then propagated in the same way as above.

    - A caller can also select an existing conversation to call the conversation's participants. The call message is sent in that conversation, and so on.

3. #### Conversation naming
    - For a **one-to-one** conversation (type 1), the conversation name should always be the recipient's one.

    - When a user initiates a **group** conversation (type 3), he is required to chose a name.

    - When the SDK needs to create a conversation to start a call, the SDK should give the conversation a name by following the rules below (according to what we can observe on the Webchat) :

        | Call type                                                    | Expected name of the conversation        |
        | ------------------------------------------------------------ | ---------------------------------------- |
        | One-to-one call (conversation of type 1)                     | Name of the recipient user               |
        | PTT call from one user to a single group                     | Group name                               |
        | PTT call from one user to several contacts (users or groups) | “Push‑To‑Talk conversation”              |
        | Conference call                                              | “Conference call conversation”           |
        | Broadcast call                                               | “Broadcast call conversation”            |
        | Emergency call                                               | “Emergency Call”                         |
        | Ambient listening call                                       | “Ambient listening! $N_{\text{caller}}$” |

    - To follow Airbus requirements, a conversation created when a user initiates a call by selecting the recipients in the contacts list, the conversation should be named **on the Dispatcher** as follow :

        | Call type                                                        | Expected name of the conversation           | Adjusted according to the user localization |
        | ---------------------------------------------------------------- | ------------------------------------------- | ------------------------------------------- |
        | One-to-one call (conversation of type 1)                         | Name of the recipient user                  | No                                          |
        | Broadcast call                                                   | “Broadcast call conversation”               | Yes                                         |
        | Emergency call                                                   | “Emergency Call”                            | Yes                                         |
        | Ambient listening call                                           | “Ambient listening! $N_{\text{caller}}$”    | Yes (except for user identifier)            |
        | Other call from one user to a prearranged group                  | $N_{\text{Group}}$                          | No                                          |
        | Other call from one user to a grey group                         | “$N_{\text{caller}}$ to $N_{\text{Group}}$” | Yes (except for user and group identifiers) |
        | Other call from one user to a list of contacts (users or groups) | “$N_{\text{caller}}$ to multiple contacts”  | Yes (except for user identifier)            |

        > **Note** - $N_{\text{user}}$ is the user identifier. It depends on the organization policy (“$\text{First name}$ $\text{Second name}$”, “$\text{Alias}$” or “$\text{First name}$ $\text{Second name}$ ($\text{Alias}$)”)

    - The new conversation, propagated through the server to the recipients, should always keep the name chosen at its creation.

---

## 3. Testing Methodology

#### _Comprehensive testing scope_

We aim to test every scenario that the SDK functions make possible, even when they don't correspond to real use cases. This approach will help identify:

- Potential future use cases
- Temporary solutions to work around defects
- Inconsistencies in SDK behavior

The tests will be conducted using the provided SDK without any patches or interventions from our part. We will use directly the functions provided by the SDK api, without any pre-processing or post-processing of data, and will collect data directly from Streamwide objects, not from our internal
representations.

#### _Narrowed reporting scope_

Each identified issue will be helpful for us but we will narrow the list when coming to reporting the defects found to Streamwide. Once Streamwide will have confirmed or clarified our understanding of entity behavior and rules of the SDK, we will be able to strictly identify which test scenarios
that correspond to real use cases and which issues suggest the presence of SDK defects that Streamwide will need to fix.

For instance, we may identify conversation naming issues that we will not need to report to Streamwide as they respond to Airbus Dispatcher requirements and not to the SDK expected behavior. But we will report any incoherence, inconsistent conversation names or problematic modifications.

---

## 4. Test definitions

### 1. Contacts

Every tests were conducted on Dispatcher users. No mobile was involved for now.

| Symbol                        | Meaning                                                                                                                                                                                                                          |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $U_1,\;U_2,\;U_3,\;U_4$       | Users in the **Org1** organisation, each granted the following rights: _Dispatcher, Calls, Cellular Call, Push‑To‑Talk, Broadcast Call Initiation Rights, Video Calls, Always connected users, Always connected calls, Add‑ons_. |
| $U_a,\;U_b$                   | Users in the **Org2** organisation, with the same rights as above.                                                                                                                                                               |
| $G_1=\{U_1,U_2,U_3\}$         | “Prearranged group” (blue group) for $U_1$ as $U_1$ is a member of the group.                                                                                                                                                    |
| $G_2=\{U_2,U_3\}$             | “Grey group” for $U_1$, as $U_1$ is not a member.                                                                                                                                                                                |
| $G_{\text{Org2}}=\{U_a,U_b\}$ | Group with Org2 users.                                                                                                                                                                                                           |
| $G_4=\{U_1,U_4\}$             | Group with Org1 users, linked to $G_{\text{Org2}}$. When a member of $G_4$ start a PTT call to $G_4$, $G_{Org2}$ members should receive the call. This is not required yet for other call types.                                 |

> **Note** – $U_1$ will always be the caller for the tests.

### 2. Contacts Presets

| Preset | Contacts      |
| ------ | ------------- |
| $P_1$  | $\{U_2\}$     |
| $P_2$  | $\{G_1\}$     |
| $P_3$  | $\{G_2\}$     |
| $P_4$  | $\{G_4\}$     |
| $P_5$  | $\{U_2,U_3\}$ |
| $P_6$  | $\{U_2,G_1\}$ |
| $P_7$  | $\{U_4,G_2\}$ |
| $P_8$  | $\{G_1,G_2\}$ |

---

## 5. Tests Scenarios

### 1. Scenario S₁ – “Start a call with no existing conversation”

#### _Pre-requisites :_

The caller ($U_1$) has no conversation with any other contact.

#### _Description :_

The caller ($U_1$) initiates a call by selecting every contact in the preset under test.

#### _Expected procedure :_

1. The SDK creates a conversation named according to naming rules. The caller sees the conversation appear, with the correct name (according to naming rules).

2. The SDK creates a call message and sends it to the conversation. The caller sees the call message appear in the conversation.

3. The server propagates the conversation by creating the instances for each callee. Each callee sees a conversation appear, with the correct name.

4. The server propagates the call message by creating a replica for each callee, linked to the callee conversation. Each callee sees the call message appear in the conversation.

5. Each callee receives the call notification and answer the call (unless it is a broadcast call, which is auto‑accepted).

6. Each participant hangs up to end the call. The call message instances are updated with the call new information (e.g. duration). Each participant sees the visible modifications of the call message in their conversation.

---

### 2. Scenario S₂ – “Call from an existing empty conversation”

#### _Pre-requisites :_

The caller ($U_1$) has no conversation with any other contact. He first creates a _custom-named_ conversation with every contact in the preset under test. The conversation will stay empty until the call test begins. That means **no message is sent in it** and none of the recipients has an instance
of that conversation yet.

#### _Description :_

The caller ($U_1$) initiates a call from that conversation.

#### _Expected procedure :_

1. The SDK creates a call message in the already‑existing conversation. The caller sees the call message appear in the conversation.

2. The server propagates the conversation by creating the instances for each callee. Each callee sees a conversation appear, with the correct name (the custom name chosen when the conversation was created).

3. The server propagates the call message by creating a replica for each callee, linked to the callee conversation. Each callee sees the call message appear in the conversation.

4. Each callee receives the call notification and answers the call (unless it is a broadcast call, which is auto‑accepted).

5. Each participant hangs up to end the call. The call message instances are updated with the call’s final information (e.g., duration). Each participant sees the visible modifications of the call message in their conversation.

---

### Collected data

For each test case, identified by its triplet (scenario, call type, preset), we list detected issues and their information :

| Category                           | Severity | Participant Type | Participants Msisdn                       | Details                     |
| ---------------------------------- | -------- | ---------------- | ----------------------------------------- | --------------------------- |
| Call not propagated                | 100-109  | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Conversation not created           | 90-99    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Conversation not correctly created | 80-89    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Conversation duplicated            | 70-79    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Call message not created           | 60-69    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Call message not correctly created | 50-59    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Call message duplicated            | 40-49    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Custom conversation name modified  | 30-39    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Custom conversation name modified  | 20-29    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| Incoherent conversation name       | 20-29    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
| SDK error in console               | 10-19    | Caller / Callee  | Id(s) of user(s) that presented the issue | Details of identified issue |
