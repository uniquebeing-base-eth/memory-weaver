# Memory Weaver

Clone this repo and continue building dear diary https://github.com/uniquebeing-base-eth/dear-diary-memories



Dear Diary — Agent-Powered Generation

The existing Dear Diary UI is already good. Do not redesign the UI or change the core visual language.

The task is to implement the real agent-powered memory generation system behind the existing UI.

1. No traditional database

Do not add Supabase/Postgres or another traditional database for the MVP.

The generation system should be stateless where possible, with the generated artwork and memory data handled through the existing app/storage architecture.

Do not introduce database/auth complexity just to support agent generation.

2. Dear Diary generation pricing

Dear Diary charges a fixed $0.10 platform fee per generation.

The user pays:

Agent generation price + $0.10 Dear Diary fee

Example:

* Agent costs $0.05 → user pays $0.15
* Agent costs $0.10 → user pays $0.20
* Agent costs $0.25 → user pays $0.35

Dear Diary fee destination:

0xF7A2d71253701a7706972002c8718728E0e98b49

The fee must be configured through an environment variable rather than hard-coded throughout the application.

Example:

DEAR_DIARY_FEE_WALLET=0xF7A2d71253701a7706972002c8718728E0e98b49

3. ERC-8004 / 8004scan agent discovery

Use the current official 8004scan API.

Base:

https://api.8004scan.io/api/v1

Semantic discovery:

GET /agents/search/semantic

The API key must remain server-side.

Do not expose the 8004scan API key in browser/client code.

Search specifically for active agents capable of:

* image generation
* text-to-image
* visual generation
* memory/story-to-image
* creative image generation

Prefer agents that:

* are active
* support Base where possible
* support x402
* expose a usable HTTP/web service
* have clear registration metadata
* have a valid service endpoint
* have evidence of working generation capability

8004scan is a discovery/registry layer, not the image-generation provider itself.

4. ERC-8004 metadata parsing

When retrieving an agent registration:

Use the current services field first.

Support legacy endpoints as a fallback.

Example:

const services =
  registration.services ??
  registration.endpoints ??
  [];

Normalize both formats into one internal representation.

Extract:

* agent name
* agent ID
* chain
* registration URI
* description
* services
* service endpoints
* x402 support
* active status
* supported protocols
* supported capabilities
* agent wallet where available

Do not assume every agent has the same service structure.

5. Agent selection

Create an internal agent-selection layer.

Example:

discoverAgents()
      ↓
normalizeAgentMetadata()
      ↓
filterImageGenerationAgents()
      ↓
filterActiveAgents()
      ↓
filterX402CompatibleAgents()
      ↓
rankAgents()
      ↓
selectBestAgent()

Ranking should consider:

1. Image-generation capability
2. x402 support
3. Base compatibility
4. Price
5. Availability
6. Service endpoint quality
7. Agent reputation/feedback where available

The application must not depend on one hard-coded agent.

If the selected agent fails, times out, returns invalid output, or becomes unavailable, automatically try the next compatible agent.

6. Memory interpretation

Do not send the user’s raw memory blindly to an image model.

Before generation, create an internal visual interpretation.

Extract:

* subjects
* relationships
* actions
* setting
* location
* important objects
* time
* weather
* distinctive details
* emotional tone
* visual composition
* camera/viewpoint
* relevant visual style

Example memory:

“Two university friends met under a tiny shelter outside campus during heavy rain after their evening class. They were both soaked, laughing while sharing one umbrella.”

The generated visual interpretation should preserve:

* two university friends
* university/campus environment
* small shelter
* heavy rain
* evening
* umbrella
* wet clothing
* laughter/friendship

Do NOT replace this with a generic “two friends in the rain.”

The result should make the user think:

“That’s my memory.”

7. Insufficient memories

The minimum memory length is 20 words.

If the user writes fewer than 20 words:

“Tell us a little more about this memory.”

Do not generate.

More importantly, even if the memory has 20+ words, if there is insufficient visual information to create a meaningful representation, the system should ask for one additional detail rather than inventing major story elements.

Do not fabricate major people, locations, events, objects, or circumstances that aren’t supported by the memory.

8. x402 payment architecture

Use the current x402 TypeScript packages:

* @x402/core
* @x402/evm
* @x402/fetch

Use the current x402 V2 architecture.

Base mainnet:

eip155:8453

Base Sepolia:

eip155:84532

Use a facilitator rather than implementing custom settlement logic.

The backend should handle the agent’s HTTP 402 response and payment requirements.

Do not invent old x402 APIs or use deprecated package names.

9. Payment calculation

The generation endpoint should determine the selected agent’s required price.

Then:

const agentPrice = getAgentPrice(agent);
const dearDiaryFee = 0.10;
const total = agentPrice + dearDiaryFee;

The UI should show the user the total before payment.

Example:

Create your memory
Agent generation     $0.10
Dear Diary fee       $0.10
────────────────────────
Total                $0.20
[ Create Memory ]

Keep the presentation friendly and simple.

Do not expose complicated x402 terminology to the user.

10. Fee routing

The Dear Diary fee must go to:

0xF7A2d71253701a7706972002c8718728E0e98b49

The agent’s required amount must go to the agent/service according to the x402 payment requirements.

Do not create a private Dear Diary custodial wallet just to pay agents.

Do not expose private keys.

Do not expose facilitator secrets.

Do not expose 8004scan API keys.

Do not expose agent credentials.

All sensitive operations stay server-side.

11. Generation endpoint

Create a backend abstraction similar to:

POST /api/memory/generate

Input:

{
  "memory": "user's memory text"
}

Internal flow:

validate memory
        ↓
interpret memory visually
        ↓
discover agents
        ↓
select agent
        ↓
determine agent price
        ↓
calculate Dear Diary fee
        ↓
return payment requirements
        ↓
user authorizes x402 payment
        ↓
invoke selected agent
        ↓
receive generated image
        ↓
validate image response
        ↓
store/retrieve image
        ↓
return generation result

12. Agent adapter

Create a provider-independent interface:

interface ImageGenerationAgent {
  id: string;
  name: string;
  price: string;
  network?: string;
  supportsX402: boolean;
  generate(input: {
    prompt: string;
    originalMemory: string;
  }): Promise<{
    imageUrl: string;
    metadata?: Record<string, unknown>;
  }>;
}

The rest of Dear Diary should not care which agent generated the image.

This makes it possible to replace agents without changing the UI.

13. Fallback

If Agent A fails:

Agent A
  ↓ failure
Agent B
  ↓ failure
Agent C
  ↓
success

Do not charge the user multiple times for failed attempts.

Payment/settlement state must be handled carefully so the user does not accidentally pay twice for one generation.

14. Async agents

Some agents may not return an image immediately.

Support asynchronous generation:

pending
processing
completed
failed

The UI should already have a beautiful magical generation state.

The backend should support polling/webhook/task-based completion depending on the selected agent’s service protocol.

Do not assume every agent is synchronous.

15. Existing Dear Diary UX

Keep the existing UI.

The generation experience should be:

Create Memory
      ↓
User writes memory
      ↓
Validate 20+ words
      ↓
Show price
      ↓
User confirms
      ↓
Magical generation animation
      ↓
Artwork reveal
      ↓
Memory detail

After generation:

* Save
* Publish
* Mint
* Gift
* Sell

The artwork remains the visual focus.

16. Farcaster

Keep this as a Farcaster Mini App.

Use the current Farcaster Mini App SDK correctly.

Support:

* SDK initialization
* ready() after the app has rendered sufficiently
* Farcaster identity
* sharing/composer
* wallet capabilities where appropriate

Avoid unnecessary login/authentication screens.

The user’s Farcaster identity should be used where available.

17. Wallet interaction

Use the user’s connected wallet for payment authorization where the Mini App wallet capabilities allow it.

Do not ask users for:

* private keys
* seed phrases
* API keys

Do not create unnecessary wallet setup screens.

18. Storage

The generated image must be retrievable after generation.

Use the existing image/storage mechanism already present in the application.

Do not introduce Supabase/Postgres simply for generation.

At minimum the generation result should preserve:

{
  originalMemory,
  imageUrl,
  agentId,
  agentName,
  generationStatus,
  createdAt
}

If persistence is currently implemented elsewhere in the application, integrate with that existing mechanism rather than introducing a new database.

19. Error states

Implement polished states for:

* memory too short
* insufficient visual detail
* no compatible agents found
* agent unavailable
* payment rejected
* payment failed
* generation failed
* generation timeout
* image retrieval failure
* all fallback agents failed

Never leave the user stuck on an infinite loading screen.

Provide:

Try Again

where appropriate.

20. Development mode

If the required credentials are unavailable:

Use mock adapters for local development.

Example:

8004SCAN_API_KEY missing
        ↓
Mock agent discovery
x402 credentials unavailable
        ↓
Mock payment flow
production
        ↓
Real 8004scan + real x402 + real agent

Do not silently use mocks in production.

Make production fail clearly if real generation credentials/configuration are missing.

21. Environment variables

Use environment variables for all secrets/configuration.

At minimum:

8004SCAN_API_KEY=
X402_FACILITATOR_URL=
DEAR_DIARY_FEE_WALLET=0xF7A2d71253701a7706972002c8718728E0e98b49
BASE_NETWORK=eip155:8453
BASE_SEPOLIA_NETWORK=eip155:84532
GENERATION_FEE_USD=0.10
IMAGE_STORAGE_URL=
IMAGE_STORAGE_API_KEY=
AGENT_DISCOVERY_QUERY=

Add any additional agent-specific configuration only when required.

Never commit .env secrets.

22. Production/testnet

Development should support Base Sepolia:

eip155:84532

Production should support Base:

eip155:8453

Use the appropriate facilitator for each environment.

Do not use the x402 test facilitator for production unless its current documentation explicitly confirms production Base support.

23. Core principle

The user should never feel like they are interacting with:

* ERC-8004
* 8004scan
* x402
* payment infrastructure
* agent routing
* AI APIs

All of that happens behind the scenes.

To the user, Dear Diary simply feels like:

Write a memory → Create → watch it come alive → receive your artwork.

The agent economy is the infrastructure underneath the experience.

24. Priority

Implement in this order:

1. Existing Farcaster Mini App shell
2. Real memory validation
3. Memory → visual interpretation
4. ERC-8004/8004scan agent discovery
5. Agent selection
6. x402 pricing/payment
7. Dear Diary $0.10 fee
8. Real agent image generation
9. Image reveal/storage
10. Retry/fallback
11. Farcaster publishing/sharing
12. Gift Memory
13. Mint Memory scaffolding
14. Sell Memory scaffolding

Do not allow marketplace features to delay the core:

Memory → Payment → Agent → Artwork.

The generation system is the most important part of this implementation.

This project was built with [Lovable](https://lovable.dev).

## Build with Lovable

Continue developing this project in the [Lovable editor](https://lovable.dev/projects/1abe583d-d939-4b73-9875-6b9e7872d8d2).

- **Ship faster**: describe what you want to build and Lovable handles the code.
- **Stay in sync**: every change made in Lovable is committed straight to this repository.
- **Full ownership**: this code is yours. Push to `main` on GitHub and your changes sync back into Lovable, ready for your next prompt.

## Development

Prefer working locally? You need Node.js and npm — [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating).

```sh
git clone <this-repository-url>
cd <repository-name>
npm i
npm run dev
```
