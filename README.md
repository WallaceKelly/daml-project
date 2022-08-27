# Daml Sample Project

## Domain Description

This project is roughly based on the domain of restaurant food delivery.

1. The guest selects their food on the ordering website and then starts the checkout process.
1. The point of sale machine (POS) at the restaurant:
   * confirms that the items are available
   * calculates the subtotal of the cart
   * calculates the tax.
1. A delivery broker service (DBS) publishes a request for bids to the local delivery service providers (DSPs).
1. The DSPs respond to the request for bids.
1. The DBS selects one of the DSPs (based on the least fee and/or soonest delivery time.)
1. The guest sees the final pricing, taxes, delivery fees, and delivery time. The guest confirms the order.

## Domain State Diagram

```mermaid
stateDiagram-v2

state "App: Guest selecting food" as ordering
state "App: Cart created" as cartCreated
state "POS: Validating" as posValidating
state isCartValid <<choice>>
state "DBS: Accepting Bids" as dbsWaiting
state dbsWaiting {
    state bidsRequested <<fork>>
    state bidsReceived <<join>>
    state "DSP1: Calculating" as dspCalculating1
    state "DSP2: Calculating" as dspCalculating2
    bidsRequested --> dspCalculating1: Bid requested
    bidsRequested --> dspCalculating2: Bid requested
    dspCalculating1 --> bidsReceived: Bid received
    bidsRequested --> bidsReceived: Timeout
    dspCalculating2 --> bidsReceived: Bid received
}
state isDriverSelected <<choice>>
state "DBS: Cart ready" as cartReady
state isCartAcceptable <<choice>>
state "App: Cart submitted" as cartSubmitted

[*] --> ordering
ordering --> cartCreated: Guest checkout
cartCreated --> posValidating: Validation requested
posValidating --> isCartValid
isCartValid --> ordering: Cart rejected
isCartValid --> dbsWaiting: Cart validated
dbsWaiting --> isDriverSelected
isDriverSelected --> ordering: Delivery not available
isDriverSelected --> cartReady: Driver selected
cartReady --> isCartAcceptable: Guest decides
isCartAcceptable --> ordering: Edit cart
isCartAcceptable --> cartSubmitted: Guest pays
cartSubmitted --> [*]
```
