# Entity relationship

```
- Destinations
  - id
- Homes
  - id
  - destinationId
- User
  - Name
  - Id
```

#### [Mermaid Diagram](https://mermaid.live/)
```
erDiagram
    Destinations {
        string id PK
    }
    
    Homes {
        string id PK
        string destinationId FK
    }
    
    User {
        string id PK
        string name
    }
    
    Destinations ||--o{ Homes : "contains"
```

![mermaid diagram](/public/mermaid.png)

# Sequence diagram

```
- UI -> AirBnB search service: Find search results for this query (where, check in, checkout, and # of guests)
- AirBnB search service -> UI: Results
- UI -> UI: Select home
- UI -> AirBnB API: Booking
- AirBnB API -> UI: Payment information
- UI -> Payment processor:
```

#### [Mermaid Diagram](https://mermaid.live/)
```mermaid
sequenceDiagram
    participant UI
    participant SearchService as AirBnB Search Service
    participant API as AirBnB API
    participant Payment as Payment Processor
    participant Email as Email Service
    participant DB as Database

    UI->>SearchService: Find search results (where, check-in, check-out, guests)
    SearchService->>DB: Query available homes
    DB-->>SearchService: Return matching homes
    SearchService->>UI: Results
    UI->>UI: Select home
    UI->>API: Get home details & availability
    API->>DB: Check availability
    DB-->>API: Availability confirmed
    API-->>UI: Home details & availability
    UI->>UI: Confirm booking details
    UI->>API: Create booking request
    API->>DB: Reserve home
    DB-->>API: Reservation confirmed
    API->>UI: Booking confirmation
    UI->>Payment: Process payment
    Payment-->>UI: Payment confirmation
    UI->>API: Confirm booking with payment
    API->>DB: Update booking status
    DB-->>API: Booking finalized
    API-->>UI: Final confirmation
    UI->>Email: Send confirmation email
    Email-->>UI: Email sent
    UI->>UI: Show booking success
```
![sequence diagram](/public/sequence.png)

# State diagram
```
- Start with destination When the destination is input, put in the check in date
- Then put in the checkout date
- Then add the number of guests
- And then when the user presses search, it'll take us to a search results page
- The search results will be loading, and eventually show up
- The user can click on one of the homes and be taken to a details page
- They can go back from the details page back to the search page
```

#### [Mermaid Diagram](https://mermaid.live/)
```
stateDiagram-v2
    [*] --> DestinationInput
    DestinationInput --> CheckInDate: destination entered
    CheckInDate --> CheckOutDate: check-in date entered
    CheckOutDate --> GuestCount: check-out date entered
    GuestCount --> SearchLoading: guests entered + search pressed
    SearchLoading --> SearchResults: results loaded
    SearchResults --> HomeDetails: home selected
    HomeDetails --> SearchResults: back button pressed
    SearchResults --> SearchLoading: search again
    GuestCount --> CheckOutDate: back button pressed
    CheckOutDate --> CheckInDate: back button pressed
    CheckInDate --> DestinationInput: back button pressed
```
![State diagram](/public/state.png)

#### [xState Diagram](https://mermaid.live/)
```typescript
import { createMachine } from 'xstate';

const bookingMachine = createMachine({
  id: 'booking',
  initial: 'destinationInput',
  states: {
    destinationInput: {
      on: {
        DESTINATION_ENTERED: 'checkInDate'
      }
    },
    checkInDate: {
      on: {
        CHECKIN_ENTERED: 'checkOutDate',
        BACK: 'destinationInput'
      }
    },
    checkOutDate: {
      on: {
        CHECKOUT_ENTERED: 'guestCount',
        BACK: 'checkInDate'
      }
    },
    guestCount: {
      on: {
        GUESTS_ENTERED: 'searchLoading',
        BACK: 'checkOutDate'
      }
    },
    searchLoading: {
      on: {
        RESULTS_LOADED: 'searchResults',
        ERROR: 'guestCount'
      }
    },
    searchResults: {
      on: {
        HOME_SELECTED: 'homeDetails',
        SEARCH_AGAIN: 'searchLoading',
        BACK: 'guestCount'
      }
    },
    homeDetails: {
      on: {
        BACK: 'searchResults',
        BOOK_NOW: 'bookingConfirmation'
      }
    },
    bookingConfirmation: {
      on: {
        CONFIRMED: 'payment',
        BACK: 'homeDetails'
      }
    },
    payment: {
      on: {
        PAYMENT_SUCCESS: 'success',
        PAYMENT_ERROR: 'bookingConfirmation'
      }
    },
    success: {
      type: 'final'
    }
  }
});
```

![xState Diagram](/public/xstate.png)