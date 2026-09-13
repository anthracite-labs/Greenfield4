# Domain

**Status: discovery in progress.**

This vocabulary captures meanings already established by product discovery. It intentionally avoids implementation classes, framework concepts, and architecture choices.

## Entities

| Entity | Definition | Identity | Invariants |
| :-- | :-- | :-- | :-- |
| **Device** | A controllable living-room endpoint discovered or added by the user. V1 is TV-first. | Stable device identity where the ecosystem exposes one; never just the current IP address. | Displayed capabilities must reflect the actual device as closely as the protocol permits. |
| **TV** | A Device that is a television and is in V1 product scope. | Device identity plus ecosystem-specific pairing identity where applicable. | A TV may expose different capabilities by model, firmware, power state, or transport. |
| **Capability** | A user-relevant action the current device can actually support, such as navigation, volume, text input, app launch, casting, or power. | Semantic capability name within the product domain. | Unsupported capabilities must not masquerade as working controls. |
| **Transport** | A control path used underneath the visible remote, such as local network control or built-in phone IR. | Transport type plus device/session context. | The user-facing remote may combine transports without forcing the user to manage them separately. |
| **Paired device** | A Device for which the app has completed any required trust/pairing ceremony and can reconnect later. | Device identity plus protected pairing material. | Pairing material is secret; identity changes require safe re-pairing rather than silent trust. |
| **IR profile** | A verified set of IR commands that works for a particular TV/device family. | Profile source plus verified command coverage. | A profile is selected from demonstrated command behavior, not merely from a database label. |

## Ubiquitous language

| Term | Means | Does NOT mean |
| :-- | :-- | :-- |
| **Universal** | One app designed to support many TV/control ecosystems honestly behind a consistent experience. | A claim that every television or device is supported. |
| **Smart-TV ecosystem** | A family of TVs sharing a sufficiently coherent remote-control protocol/product environment for support and maintenance to be evaluated as a unit. | A guarantee that every model/firmware in that brand or family behaves identically. |
| **Capability-driven remote** | A remote whose available controls are derived from what the connected device actually supports. | One fixed button layout for every TV. |
| **Hybrid control** | One visible remote transparently using more than one Transport when needed. | Making the user switch between separate Wi-Fi and IR remotes. |
| **Discovery** | Finding candidate devices on the local environment and identifying enough trustworthy information to present them to the user. | Proof that a discovered device is already supported. |
| **Pairing** | The device-specific trust/authorization step required before persistent control. | Account creation or cloud login. |
| **Remembered device** | A previously paired/configured device that the app can attempt to reconnect to automatically. | A permanently reachable IP address. |
| **Tested** | Compatibility verified on real hardware/model/firmware represented in the release matrix. | A broad inference from protocol documentation alone. |
| **Expected** | Compatibility predicted from verified family/protocol evidence but not represented by a specific tested device. | The same confidence level as Tested. |
| **Unsupported** | Known not to work or deliberately not supported in the current product/release. | Merely offline or temporarily unreachable. |
| **Reconnecting** | A temporary state where the app is restoring control to a remembered device. | A reason to force the user back through setup immediately. |
| **Local-first** | Core remote control works directly in the user’s local environment without depending on our cloud or internet service. | A promise that every vendor protocol itself is offline or unauthenticated. |

## State transitions

| From | Event | To | Guard |
| :-- | :-- | :-- | :-- |
| Unknown | Device discovered | Discovered | Enough trustworthy information exists to present a candidate device. |
| Discovered | User selects a supported device requiring authorization | Pairing | Ecosystem support exists and pairing is required. |
| Discovered | User selects a supported device not requiring pairing | Connected | A safe supported connection is established. |
| Pairing | Pairing succeeds | Paired / Connected | Device identity and any required credentials are stored safely. |
| Pairing | Pairing fails or identity cannot be trusted | Discovered / Failed | Never bypass required security checks. |
| Paired / Connected | Connection drops | Reconnecting | Device remains remembered. |
| Reconnecting | Safe connection restored | Connected | Device identity still matches expected trust state. |
| Reconnecting | Recovery cannot complete | Offline | User gets a simple actionable state rather than repeated errors. |
| Paired | Device security identity changes unexpectedly | Re-pair required | Silent trust replacement is forbidden. |

## Business rules

| Rule | Rationale | Enforced where |
| :-- | :-- | :-- |
| The actual connected device is the source of truth for capabilities. | Model/family assumptions can be wrong across firmware and variants. | Product behavior/spec; later protocol adapters. |
| Core remote control is local-first. | The remote should not stop working because our service or the internet is unavailable. | Product requirements; later architecture. |
| Security wins over popularity or convenience. | A popular ecosystem is not worth shipping through an unsafe trust model. | Product requirements; security review; later adapters. |
| Pairing secrets are treated like passwords. | They authorize control of household devices. | Security requirements; later platform storage. |
| No behavioral usage analytics in V1. | Remote behavior is private household activity and is not required to deliver core value. | Product requirements; telemetry design. |
| Built-in IR is a V1 capability only where the phone exposes suitable hardware. | IR extends reach without requiring external accessories. | Product requirements; later platform capability detection. |
| One visible remote may combine transports. | Users care that a command works, not which protocol carried it. | Product UX/domain behavior. |
| Compatibility claims distinguish Tested, Expected, and Unsupported. | “Universal” must remain honest and maintainable. | Compatibility documentation and release process. |
| Release support needs a deliberate real-hardware matrix. | One successful TV proves an integration, not an ecosystem. | Release criteria. |

## Engineering vocabulary retained from the foundation

| Term | Meaning here |
| :-- | :-- |
| **Foundation** | The generic engineering system originally shipped by App-Factory and retained as this repository’s engineering base. |
| **Adapter** | `.ecc/` — the ECC-on-Arena adaptation. Not native ECC. |
| **Gate** | `scripts/verify.sh` — deterministic, non-zero on failure. |
| **Lifecycle phase** | `PROJECT_PHASE` in `config/project.env`. |
| **No-stack guard** | The `no_app_stack` check, driven by `ALLOW_APP_STACK`. |
| **Project memory** | `docs/MEMORY.md` — append-only, Git-tracked. |
| **ADR** | A record under `docs/decisions/` for a durable trade-off. |

## Related

- [PRODUCT.md](PRODUCT.md) — current product discovery definition
- [ARCHITECTURE.md](ARCHITECTURE.md) — engineering system; no application stack selected yet
- [decisions/](decisions/README.md) — durable decisions
