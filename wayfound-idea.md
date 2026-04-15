# Wayfound — A Context-Aware AI Travel Itinerary Planner
### Product Idea Archive
*Written to be read cold. Everything you need to rebuild this idea from scratch.*

---

## The Problem

Most travel itinerary tools treat trip planning like a list problem. You search for places, add them to a list, and the tool gives you a day-by-day breakdown. The result looks organised but falls apart the moment you actually travel.

The fundamental failures are:

**Geographic blindness.** Most itineraries don't account for the fact that places have physical locations. Day 2 might take you from the north of the city to the south and back to the north again, wasting hours in transit. The tool never considered that grouping places by geography — not by category — would make the day dramatically more efficient.

**Operational ignorance.** A tool might suggest visiting a famous museum on a Monday without knowing it's closed every Monday. Or schedule a sunset viewpoint at 2pm. Or recommend a restaurant that requires booking six weeks in advance. These aren't edge cases — they're the kind of silent failures that ruin days.

**Context deafness.** A family travelling with a two-year-old has completely different constraints than a solo backpacker. Someone who hates crowds needs a fundamentally different itinerary to someone who doesn't mind them. A vegan travelling through Tokyo needs food stops treated as first-class constraints, not afterthoughts. Most tools don't know any of this about you, and the few that do ask you about it once in a setup form and then forget.

**The perfect plan fallacy.** Most tools try to generate one perfect itinerary. But the best trip isn't the one perfectly optimised before you leave — it's the one that adapts as things change. You're tired. It's raining. You spent longer at that café than planned. You stumbled across a market you didn't know existed. A good travel companion adapts. A static plan doesn't.

---

## The Core Idea

Wayfound is a map-first, AI-assisted travel planning tool where the map is the primary surface and the AI is an ambient intelligence layer that makes the map smarter than anything a user could build alone.

The mental model shift that defines the product:

**Most AI travel tools:** AI asks questions → generates a plan → shows it on a map.

**Wayfound:** User plans on the map → AI watches what they're doing, infers what they need, and surfaces intelligence at the right moment → user stays in control.

The AI never drives. It observes and assists. The user is always the planner. The AI is the knowledgeable local friend sitting next to them — the one who quietly says "that place closes at five, you won't make it" without being asked, who suggests the ramen spot two streets over that fits perfectly into the gap in the afternoon, who re-routes the day wordlessly when the user says their feet hurt.

---

## The Graph Model — How the Product Thinks About Trips

The most important technical and conceptual insight in Wayfound is this: **a trip is a graph, not a list.**

Every place the user wants to visit is a **node**. Every journey between two places is an **edge**. The entire trip — across all days, all cities — is a directed graph of nodes connected by edges.

### Nodes (Places)

Each node carries a set of properties:

- Name and category (museum, restaurant, park, hotel, transport hub)
- Opening hours and closed days
- Last entry time (often different from closing time)
- Entry cost
- Typical crowd level by time of day and day of week
- Accessibility information
- Estimated time to spend there
- Dietary options (for food nodes)
- Weather sensitivity (outdoor vs indoor)
- "Based on" signals — why the AI suggested or placed this node

### Edges (Journeys Between Places)

Each edge carries:

- Travel mode (walk, taxi, metro, car, train)
- Duration (affected by time of day and real-time conditions)
- Cost
- Departure time
- Arrival time
- Whether the mode was explicitly chosen by the user or assumed by the AI (the "soft lock" state)

### Why This Model Matters

When you think about a trip as a graph, everything becomes clearer:

- **Geo-optimisation** is just finding the most efficient ordering of nodes to minimise total edge cost
- **Constraint validation** is just checking each node and edge's properties against the user's constraints
- **Time conflicts** are detectable by checking whether you can traverse an edge and arrive at a node within its operating window
- **Re-routing** is just updating edge properties and propagating the changes downstream through the graph
- **"My feet hurt"** translates to temporarily increasing the cost threshold of long walk edges, which forces the graph to find shorter alternatives

The AI's job is to help populate node properties (it knows about places), validate constraints (it knows about the user), resolve incomplete edges (it asks when travel mode is genuinely ambiguous), and suggest new nodes to add to the graph (it knows what's nearby and relevant).

---

## The User Experience

### Onboarding — The Hard Constraints

Before the user touches the map, the product collects a small set of information that would silently break the plan if the AI assumed incorrectly. The design principle is strict: **only ask what cannot be recovered from getting wrong through conversation or assumption.**

The onboarding questions are:

**1. Where to? (multi-city)**
Each destination gets its own date range. A user planning Tokyo → Kyoto → Osaka adds three destination cards, each with arrival and departure dates. This is structural information — the AI cannot begin without it.

**2. Who's coming?**
Just me / Couple / Friends / Family + kids. If family with kids is selected, the form expands to show age band counters (Under 3 / Ages 3–6 / Ages 7–12 / Ages 13–17). A two-year-old changes everything — stroller access, nap windows, no late nights. A ten-year-old means discounted entry at most attractions. A fifteen-year-old means teen-appropriate activities. These aren't nice-to-haves; they fundamentally shape every suggestion.

**3. Daily budget ceiling per person**
A single hard number — not a tier, not a range. "Places above this won't be suggested." The framing is important: this is an eliminator, not a preference. It doesn't mean every day will cost this amount — it means the AI won't suggest a $400 tasting menu to someone with a $100/day ceiling. The field is skippable for users with no budget concern. Budget as a soft preference is handled in the map and chat later. Budget as a hard ceiling is an onboarding constraint.

**4. Dietary needs?**
None / Vegan / Vegetarian / Halal / Gluten-free / Nut allergy. Selecting "None" collapses the question into a compact summary row — the form gets shorter as you fill it out. This is intentional: the cognitive load of onboarding should decrease as the user progresses, not stay flat.

**5. Do you drink alcohol?**
Yes / No / Occasionally. "No" collapses. This affects whether bars and wine-focused restaurants appear in food suggestions, whether nightlife venues get recommended, and how evening slots are filled.

**6. Accessibility or medical needs?**
None / Mobility / Sensory / Medical condition / Service animal. Deliberately broad — the AI asks in chat if it needs specifics. "Mobility" covers wheelchair and low-mobility. "Sensory" covers visual and hearing. "Medical condition" is a flag that prompts the AI to ask what's relevant in context. Selecting "None" collapses the question.

That's it. Six questions. Everything else is handled in the map and through conversation.

### The Map Screen — The Primary Surface

After onboarding, the user lands on a large map of their first destination. This is the canvas. Everything happens here.

**The map shows:**
- Pins for every place added to the plan, color-coded by category (purple for sightseeing, teal for food/evening, amber for accommodation, etc.)
- A dashed route line connecting stops in order, drawn geographically so the user can see the actual path their day will take
- A crowd heatmap overlay (toggleable) showing where it gets busy at the current planned time
- Filter chips floating at the top of the map — All / Culture / Food / Nature / Nightlife / Crowd map

**Below the map:**
- Day tabs for switching between days
- A draggable stop list showing each stop with time, a brief note, and a grip handle for reordering
- Each stop has a "Based on" subtext line showing why the AI placed it there (e.g., "Based on: museum closes 6pm · mentioned broth")
- A chat input at the bottom

**The split between what lives in the map vs what lives in the chat:**
- Map: the plan itself, all visual state, constraints, conflicts
- Chat: shortcuts and corrections ("add a sake bar near stop 3", "day 2 feels too rushed", "my feet hurt")
- Neither drives the other — they're two views of the same underlying graph

### How the Map Gets Populated

The user can add places in several ways:

1. **Tap anywhere on the map** — a lightweight info card surfaces with crowd times, a one-line description, and an "Add to day" button
2. **Type in chat** — "find a ramen spot near stop 3" → the AI identifies candidates, pins them on the map, user taps one to confirm
3. **Category selection** — the user taps a category filter and the AI surfaces relevant candidates in the current area as ghost pins the user can accept or dismiss
4. **AI suggestion in a gap** — if there's unscheduled time between two stops, the AI may quietly offer 2–3 contextual suggestions for that slot

When a place is added, the AI immediately fills in the node properties — opening hours, cost, crowd level, accessibility — from data (Google Places API primarily). The edge from the previous stop is created with an incomplete travel mode property, which the AI resolves by asking or assuming based on distance.

---

## The AI Intelligence Layer

### The Four Types of AI Intervention

The AI never speaks unprompted unless it has something genuinely useful to say. Its interventions fall into four categories:

**Ambient** — information placed on the card automatically, no message, no notification. Opening hours, crowd level, entry cost, the "Based on" subtext. The user doesn't need to ask because it's already there.

**Warning** — something is wrong with a node or edge. Severity levels:
- Info (purple indicator): context only, no action needed. "Opens at 9am · last entry 4:30pm."
- Soft warning (amber indicator): worth knowing, user decides. Dismissable. Budget 20% over daily average.
- Hard warning (red indicator): genuinely breaks the plan. Re-warns even if dismissed once. "Last entry strictly 1 hour before close — you arrive after cutoff."

**Suggestion** — a gap in the day, a complementary place, a better ordering. Offered quietly, never forced.

**Inference** — the AI reads a pattern and updates its internal model. Several similar pins → inferred interest. "My feet hurt" → temporary constraint. "Not a wine person" → permanent preference stored.

### The Three-Condition Rule for Asking Questions

The AI asks a question only when all three conditions are true:
1. A node or edge has a property that is **missing**
2. That property **cannot be inferred or assumed** from available context
3. The answer **materially changes the plan**

If any one condition fails, the AI either assumes and states the assumption, fills in a default, or skips.

Example — travel mode between two stops that are 400 metres apart: condition 3 fails (the difference between 5-minute walk and 4-minute taxi doesn't change anything downstream), so the AI assumes walk and never asks.

Example — travel mode between two stops that are 4 kilometres apart: all three conditions are true (mode unknown, can't infer, changes the edge duration by 15+ minutes which affects downstream slots), so the AI asks.

### Information States

The AI manages four states of knowledge about any plan property:

**Known** — the user told us directly. Use it, no question needed.

**Inferred** — derived from context. "Family + kids" → no late-night venues. Apply silently, mention only if it shapes something significant.

**Assumed** — agent makes a smart default. Hotel area unknown → use budget to pick likely neighbourhood, state the assumption. "I'll anchor your plan around Midtown based on your budget — tell me if your hotel ends up somewhere different." Proceed immediately. Never hold the plan hostage to missing information.

**Blocking unknown** — genuinely can't proceed, no reasonable assumption exists. Rare. Ask exactly this one thing.

The key principle: **a plan with explicit assumptions is always more useful than no plan.** The agent proceeds, states what it assumed, and updates when corrected.

### The Soft Lock System

When a node or edge property is based on an assumption rather than explicit user input, a small icon appears on that card element. It looks like a note, not a warning. Tapping it reveals a tooltip explaining the assumption and offering alternatives.

For example: a 12-minute walk indicator between two stops shows a walking icon. Tapping it opens: "Assuming you're walking — tap to switch: Walk / Taxi / Metro." The user changes it if needed, the edge duration updates, downstream times ripple through the graph.

The soft lock is zero words in the main chat interface. The assumption is communicated entirely through the UI element's state.

### The "Based On" Subtext

Every stop card has a secondary text line that shows the AI's reasoning for placing it there — not a description of the place, but the working hypothesis. For example:

> Ramen Nagi — 7:00pm
> Based on: museum closes 6pm · mentioned broth

This is not prose. It's a label. The user sees immediately why the AI chose this place at this time. If the reasoning is wrong, they know exactly what signal to correct. It also naturally surfaces the AI's logic without requiring it to explain itself in chat — which would be verbose and annoying.

### Constraint Chips — Learning Through Correction

When a user corrects an assumption in chat, a small chip appears above the message input confirming what was learned. For example:

User: "Actually not really a wine person."
Chip: "Avoiding wine-focused spots ×"

The user didn't fill out a field. They just corrected a bad suggestion. The system now has a hard constraint moving forward. The chip is the confirmation. Tapping × dismisses it if the user wants to review it later. The constraint persists across the entire trip.

### The Warning Dismissal Model

When a user sees a warning and does nothing — scrolls past, reads it, shrugs — the pin gets a grey dot indicator. The warning is noted as seen. The AI internally records "user was okay with this type of conflict at this severity level" and doesn't re-warn for similar issues unless they're hard closures.

There's an important distinction between scroll-past (ambiguous — they may not have fully registered it) and explicit dismiss (confident ignore). Scroll-past gets the grey dot but doesn't update the AI's learned tolerance. Explicit dismiss does both.

Hard closures — last entry strictly enforced, fully closed, no exceptions — always re-warn regardless of dismissal history. Some conflicts are genuinely too important to learn away.

### Budget at the Day Level, Not the Pin Level

Budget warnings don't live on individual stop cards. They live in the day header. When cumulative cost for the day exceeds the user's daily average (derived from the ceiling they set in onboarding), the total cost figure in the day header changes from muted grey to amber. It doesn't say "OVER BUDGET." It just shows the number in amber — "Day 2: $215 / $180 est." — inviting the user to tap it if they want to see the breakdown.

Tapping opens a line-by-line cost breakdown with the spend spike highlighted and a quiet swap option for the expensive item. The note at the bottom contextualises across the trip: "If you skip tomorrow's paid museum, you're back on overall estimate."

This is deliberate. Budget isn't linear — a $200 dinner might be intentional because the user planned a cheap lunch tomorrow. A per-pin warning on a $200 dinner is potentially a false positive. A day-level amber indicator is accurate and non-alarming.

### Temporary vs Permanent Constraints

The AI distinguishes between constraints that are temporary states and constraints that are permanent attributes of the traveller.

"My feet hurt" → temporary hard constraint. The AI tightens the walk radius for the next 2–3 hours — the map cluster visually contracts — then dissolves the constraint after time passes or when the user adds a distant stop themselves. No explanation given. The map just adjusts.

"I have bad knees" → permanent constraint. The AI treats this as an accessibility attribute, equivalent to what was set in onboarding. It becomes a constraint chip.

The visual feedback in both cases is the same — the map tightens, a brief "Walking minimised" overlay flashes on the map — but the persistence differs. The user doesn't manage this distinction themselves; the AI infers it from language.

---

## The Onboarding-to-Map Transition

After onboarding, the user has two entry modes:

**Guide me** — the AI asks a small set of contextual questions that build the first day's structure through conversation. Each answer populates the map in real time.

The questions are not scripted. They emerge from what the AI needs to resolve incomplete node and edge properties. But there's a predictable starting sequence:

1. "How are you arriving in [city]?" — drops an arrival pin at the correct terminal or station. This isn't just geographic — arriving by a long train tells the AI the user might be tired, and it adjusts Day 1's intensity accordingly. The arrival time tells it when Day 1 actually starts.

2. "Do you want to go straight to your hotel or see a bit along the way?" — this single question determines whether Day 1's route originates from the arrival point or the hotel. It draws the first edge on the map in real time as the user answers.

3. "Any fixed commitments during the trip — flights, reservations, meetings, anything already booked?" — one question that catches everything. A flight home at 6pm, a dinner reservation on Day 3, a conference session on Tuesday morning. The AI parses what each means, blocks those slots in the graph, and builds around them.

4. "Anything you know you want to do or see?" — named places get pinned immediately. Interests mentioned ("street food", "temples", "jazz bars") become scoring signals that weight all subsequent suggestions.

After these questions — however many or few the AI determines it needs — the plan generates and the map populates.

**Give me an overview** — the AI generates a full plan immediately from onboarding data alone, stating its assumptions explicitly. "I've anchored your plan around Midtown based on your budget. I've assumed you start at 9am each day. No commitments blocked — let me know if anything changes." The user gets an immediate full plan and corrects from there.

The two modes aren't presented as a formal choice screen. The chat bar simply opens with the guided path's first question, and there's a "Just generate a plan" option if the user wants to skip to overview.

---

## The Ambient UI Philosophy

The most important design principle in Wayfound is: **the interface is the communication medium, not the chat.**

Most AI products use the UI to display what the chat said. Wayfound uses the chat for intent and corrections, and the UI for state. The AI doesn't say "I assumed you're walking" — it shows a walking icon on the edge. It doesn't say "this place closes before you arrive" — it shows a red dot on the pin. It doesn't say "I've adjusted your route because you're tired" — the map visually tightens.

This has a specific name: state, not statement. The position of a UI element, the color of an indicator, the subtext on a card — these are the communication. Chat is for the things that can't be shown.

When this works well, the user barely notices the AI. Information is already there. Conflicts are already flagged. Gaps are already filled. The AI earns its place by being useful at the right moment without announcing itself.

---

## The Technical Architecture

### The Mental Model

The product has five layers:

1. **Frontend** — the mobile app, the map, the UI
2. **Map and places data** — real-world information about nodes
3. **Graph engine** — the trip data model and its real-time state
4. **AI layer** — intelligence on top of the graph
5. **Backend and infrastructure** — the plumbing

### Frontend

**React Native** for the mobile app — single codebase for iOS and Android, mature ecosystem, good map library support. Expo for fast iteration in MVP phase.

**Mapbox SDK** for the map — custom styling, offline tiles, real-time layer updates, pin clustering. More control than Google Maps for the custom pin states, route overlays, and crowd heatmap layers the product requires.

**React Query + Zustand** for state — React Query handles server state (places data, routes). Zustand manages the local graph state — nodes, edges, constraint store. Keeps the map reactive without over-engineering.

**Reanimated 3** for animations — the ambient UI patterns require smooth, non-blocking animations. Map cluster tighten, pin state transitions, budget header amber fade. Runs on the UI thread so nothing drops frames.

### Map and Places Data

**Google Places API** — opening hours, popular times (this is where crowd data comes from), photos, ratings, price level. Best global coverage. Powers automatic node property population when a pin is dropped.

**Google Directions API** — edge properties: walk, transit, and drive duration with real-time traffic. Powers the travel mode edge resolution.

**OpenWeatherMap API** — forecast per day for weather-aware node flagging and indoor/outdoor suggestions.

**Foursquare or Yelp API** — supplements Google Places for dietary filtering, menu data, and neighbourhood-level food discovery.

**Important:** Cache all Places API responses in Redis with a 24-hour TTL. Popular places get looked at repeatedly. Google charges per request. Without caching, API costs become unmanageable quickly.

### Graph Engine

**PostgreSQL + PostGIS** — the source of truth. Nodes stored as rows with geospatial point columns. Edges as rows with foreign keys to source and destination nodes. PostGIS enables "find all nodes within 0.5mi of this point" queries efficiently. pgRouting can handle geo-optimised stop ordering.

**Redis** — live session state. The active trip graph lives in memory during planning sessions for fast reads. Redis pub/sub pushes real-time graph updates to the client when nodes or edges change.

**Supabase Realtime** — WebSocket layer that delivers graph changes to the mobile client instantly. When a node's opening hours are fetched and stored, the pin on the map updates without a poll. Powers the live diff animations.

### AI Layer

**Claude API** — the language model. Natural language understanding for chat messages, constraint inference from conversational input ("my feet hurt" → temporary mobility constraint), explaining decisions when asked. Use `claude-sonnet-4-6` for the balance of speed and quality.

**Tool use / structured outputs** — the most architecturally important piece. Claude doesn't just respond with text. It calls tools that directly mutate the graph. "Add a ramen spot near stop 3" triggers a `search_candidates` tool call, then a `create_node` tool call, then an `add_edge` tool call. The chat message is a side effect, not the action. The map updates because tools were called, not because text was generated.

The tools the model has access to:
- `search_candidates` — find places matching criteria near a location
- `create_node` — add a place to the trip graph
- `update_node` — modify a node's properties or timing
- `remove_node` — remove a place
- `create_edge` — add a journey between two nodes
- `update_edge` — modify travel mode, timing, etc.
- `flag_constraint` — surface a warning on a node or edge
- `store_preference` — persist a user preference or constraint from conversation
- `get_graph_state` — read the current state of a day or the full trip

**Constraint engine (custom, lightweight)** — runs before the AI on every graph mutation. Checks each new or modified node/edge against the user's hard constraints: budget ceiling, dietary needs, accessibility requirements, opening hours. These checks don't need a language model — they're deterministic rules. The constraint engine catches most issues cheaply. The AI only fires when the constraint engine flags something ambiguous, or when the user sends a chat message.

This separation is critical. The AI should be reserved for genuinely hard problems — ambiguous language, inference, generation. Using it for "is this restaurant vegan?" is expensive and slow when a simple API lookup would do.

**pgvector** — stores user preference signals as embeddings. Interest inferences, past corrections, constraint history. Powers personalised recommendations across multiple trips. Allows the product to get smarter about a user over time without requiring them to re-onboard.

### Backend

**Node.js + tRPC** — type-safe API between frontend and backend. tRPC eliminates REST boilerplate — graph mutations are typed function calls. Fast to iterate on for a small team.

**Supabase** — PostgreSQL + auth + realtime + storage in one. Dramatically reduces infrastructure overhead for MVP. Scales reasonably well before needing to self-host.

**BullMQ** — async job queue backed by Redis. Handles background work: fetching place data when a node is created, recalculating routes after a graph mutation, generating export files. Keeps API responses fast by doing heavy work asynchronously.

**Vercel / Railway** — Vercel for any web surface (share links, export pages). Railway for the Node backend.

### Build Order

**MVP:** React Native + Mapbox + Supabase + Claude API with tool use. Manual pin dropping, basic graph in Postgres, Claude handling chat and constraint inference. Google Places for node data. Short-interval polling instead of true realtime.

**V2:** WebSocket graph updates, PostGIS proximity queries, Redis session state, the constraint engine separating cheap checks from AI calls. Crowd heatmap via Google Popular Times. BullMQ for async data fetching.

**V3:** pgvector preference embeddings, shared itinerary links, collaborative editing, calendar export, PDF generation. Live traffic on edges via Directions API. Multi-user support.

---

## The AI Engineer's Specific Scope

If building this product as an AI engineer specifically (as opposed to a full-stack team), your surface area is:

**Tool design** — defining the tools the model has access to, their input schemas, their descriptions. How tools are named and described directly affects when and how the model chooses to call them. This is where most of the product intelligence lives.

**Context window management** — the model needs to know the current graph state, the user's hard constraints, their preference and correction history, and the current conversation. On a multi-city, multi-day trip, the raw graph state can be large. How you compress and prioritise what goes into context — what to include in full, what to summarise, what to omit — directly affects output quality and cost.

**The constraint engine / invocation logic** — deciding when to invoke the model at all. Not every graph event needs AI. The constraint engine handles the deterministic checks. The model fires for ambiguous language, inference, and generation. The threshold between cheap rule and expensive model call is something you tune over time.

**The preference pipeline** — taking conversational corrections and classifying them as temporary state, permanent constraint, or interest signal. Deciding what gets stored in pgvector, in what form, and how it gets retrieved and injected into future context.

**System prompt design** — how you frame the model's role, what context it has at different moments (onboarding, active planning, correction), and how it should handle the different intervention types (ambient fill, warning, suggestion, inference).

Everything else — the map, the UI, the database schema, the WebSockets, the deployment — belongs to frontend and backend engineers.

---

## Key Design Principles — The Short Version

For quick reference, the principles that everything else is derived from:

1. **The map is the source of truth.** Chat is a shortcut. Filters are a view. The pins and routes on the map are what the trip actually is.

2. **AI never blocks the user.** No required answers. No waiting for the agent. The map is always usable.

3. **State, not statement.** The interface is the communication medium. UI element position and state carry the meaning. Chat is for what can't be shown.

4. **A plan with explicit assumptions is always more useful than no plan.** Proceed, state the assumption, update when corrected.

5. **Questions emerge from the three-condition rule.** Ask only when a property is missing, can't be inferred, and materially changes the plan. All three must be true.

6. **The constraint engine runs before the AI.** Cheap rules catch cheap problems. The model is reserved for hard problems.

7. **Onboarding captures only what would silently break the plan.** Everything else is inferred, assumed, or handled in context.

8. **Signal strength matches problem severity.** Info is info. A soft warn is amber. A hard warn is red. Never scream when a whisper is appropriate.

9. **Geo-optimisation is the default.** The user never needs to ask for it. It's just what the routing engine does.

10. **The plan is never done.** It's the current best version of what the agent knows. It updates as the user books things, changes their mind, or the world changes around them.

---

## What Makes This Different

There are many travel planning tools. Most of them are sophisticated lists with map views bolted on. What Wayfound is specifically:

**A geo-first planner** that thinks about places as physical locations that need to be visited in an efficient order, not categories in a list.

**An operationally aware planner** that knows places open and close, that knows some things are better at 6am and some things are better at midnight, that knows Monday is a bad day for museums in Paris.

**A constraint-respecting planner** that knows your hard limits — dietary, accessibility, budget — and applies them silently without asking about them again.

**A live document** that updates as you plan, as you book, as you correct it, as things change in the real world.

**An ambient intelligence** that communicates through the interface, not through a chatbot. The AI earns its place by being useful at the right moment without announcing itself.

---

*This document was written in April 2026 through a long product design conversation. If you're reading this to rebuild the idea: start with the graph model. Everything else follows from it.*
