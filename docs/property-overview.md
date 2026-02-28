# Property Overview (PukkancsLak)

Generic reference for the property layout. Link to this document from networking, heating, and other subsystem docs when room or area names are needed.

---

## Building Layout

The house is a **split-level property** with **7 half floors** in a staggered layout:

```text
Floor layout (vertical relationship):

1
             2
3
             4
5
             6
7
```

The split-level design places floors **next to each other** (side by side) rather than stacked; 3rd Floor is above 1st Floor, 2nd Floor is halfway shifted to the side. **Cabling:** vertical runs (e.g. 1→3) are typically easier than lateral runs (e.g. 1→2). Floors 6–7 are empty spaces, so cable runs there are straightforward.

```text
─────────────────────────────────────────────────────────────────────

FLOOR 7            High Attic

─────────────────────────────────────────────────────────────────────

FLOOR 6            Low Attic

─────────────────────────────────────────────────────────────────────

FLOOR 5            ┌──────────────┬──────────────┐
                   │              │              │
              Alex Room     Vicky Room    Master Bedroom

─────────────────────────────────────────────────────────────────────

FLOOR 4            ┌──────────────┬──────────────┐
                   │              │              │
              Bathroom      Play Room        Office

─────────────────────────────────────────────────────────────────────

FLOOR 3               Main House
                         │
          ┌──────────────┼──────────────┐
          │              │              │
    Living Area    Dining Area     Kitchen
          │
    ┌─────┴─────┐
    │           │
 Couch    Entertainment

─────────────────────────────────────────────────────────────────────

FLOOR 2            Guest House (separate entrance)
                         │
    ┌────────────┬───────┴────────────────────┬────────────┐
    │            │                            │            │
Guest Bedroom  Guest Living Room  Guest Kitchen  Guest Bathroom
                      │
               ┌──────┴──────┐
               │             │
          Dining Area   Living Area
                             │
                        ┌────┴────┐
                        │         │
                    Couch   Entertainment

─────────────────────────────────────────────────────────────────────

FLOOR 1            ┌──────────────┬──────────────┐
                   │              │              │
               Garage        Laundry      Summer Kitchen

─────────────────────────────────────────────────────────────────────
```

## Outdoor Areas

**Front outdoor area:** Main Car Gate, Garage Car Gate, Main Pedestrian Gate, Waste Storage Gate.

**Back garden:** Kids Playhouse, Grill House.

---

*Update this document when the property layout changes. Keep it generic; subsystem-specific details (e.g. AP placement, Sonos) stay in the relevant current/future docs.*
