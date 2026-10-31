<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // The five reports the desk read as a one-day move (desk-notes.json) are their own problem:
  // their bodies show no day/month swap, so the parser fix does not cover them.
  type Problem = 'parser' | 'hint' | 'one-day-move' | 'unplaced';

  const LABEL: Record<Problem, string> = {
    parser: 'shared parser reads dates month first',
    hint: 'booking-v3 form hints MM/DD/YYYY to day-first tenants',
    'one-day-move': 'booking lands exactly one day off',
    unplaced: 'tenant unknown',
  };

  const problemOf = (r: Report): Problem =>
    notes.oneDayMove.includes(r.id) ? 'one-day-move' : (causeOf(r, tenantOf(r)) ?? 'unplaced');

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));

  const rows = $derived(
    (Object.keys(LABEL) as Problem[])
      .map((p) => {
        const rs = merged.filter((r) => problemOf(r) === p);
        const tags = rs.reduce<Record<string, number>>((n, r) => ({ ...n, [r.meta.intakeTag]: (n[r.meta.intakeTag] ?? 0) + 1 }), {});
        return { problem: p, reports: rs, tenants: new Set(rs.map((r) => r.tenant)).size, tags };
      })
      .filter((row) => row.reports.length > 0)
      .sort((a, b) => b.reports.length - a.reports.length),
  );

  // Intake's tag against the problem the data shows: where the tag was wrong.
  const wrongTag = $derived(
    merged.filter((r) => {
      const p = problemOf(r);
      return (p === 'parser' || p === 'hint') && r.meta.intakeTag !== p;
    }).length,
  );
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>intake tagged them</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr>
        <td><code>{row.problem}</code> · {LABEL[row.problem]}</td>
        <td>{row.reports.length}</td>
        <td>{row.tenants}</td>
        <td class="tags">{Object.entries(row.tags).sort((a, b) => b[1] - a[1]).map(([t, n]) => `${t} ×${n}`).join(' · ')}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each rows.filter((row) => row.reports.length <= 10) as row (row.problem)}
  <p class="note">{row.problem}: {row.reports.map((r) => r.id).join(', ')}</p>
{/each}

<p class="note">Intake's tag disagrees with the problem for {wrongTag} reports.</p>

<style>
  .sum { font-size: 1.1rem; }
  .tags, .note { color: var(--muted); font-size: 0.85em; }
</style>
