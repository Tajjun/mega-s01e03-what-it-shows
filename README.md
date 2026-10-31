# S01E03 · what it shows

Ticket #4471 holds 412 reports (427 export rows, 15 of them exact duplicates). They are two problems, not one: the shared parser (379 reports) and the booking-v3 form hint (33 reports), counted the way the desk counts. One issue per problem, with its report count, is filed in this repo's Issues.

- `src/routes/kb/incidents/4471-three-problems/+page.svx`: the new kb page (signed with the course ticket)
- `src/lib/kb/ProblemSplit.svelte`: computes the counts from the kb data at render time
- `page.html`: the page as the desk will see it (static copy, rendered from the same data)

Only the new files are here. The page imports components and data from the support desk's kb export.
