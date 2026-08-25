# Macro Name
Select a token and apply conditions. You can select multiple tokens, but the checkboxes for available statuses only look at the first token and the change will override conditions for ALL selected tokens.

The Conditions are pulled from CONFIG.statusEffects so any available conditions in the world appear on the list. There's also a search bar. If a condition is already applied to a token, then it asks if you want to modify or remove the current conditions. You can choose 'modify' and select / deselect conditions to apply them.

## Dependencies
- Foundry 14, build 364
- D&D5e Game System, version 5.3.3


## Code
```js
const DialogV2 = foundry.applications.api.DialogV2;

if (canvas.tokens.controlled.length === 0) {
  ui.notifications.warn("Select at least one token first.");
} else {
  const reference = canvas.tokens.controlled[0].actor;
  const currentStatuses = reference?.statuses ?? new Set();

  let proceedToConfigure = true;

  if (currentStatuses.size > 0) {
    const activeLabels = CONFIG.statusEffects
      .filter(e => currentStatuses.has(e.id))
      .map(e => game.i18n.localize(e.name ?? e.label ?? e.id))
      .join(", ");

    const choice = await DialogV2.wait({
      window: { title: "Conditions Already Applied" },
      content: `<p>${reference?.name ?? "This token"} currently has: <strong>${activeLabels}</strong>.</p>
                 <p>Clear all conditions, or modify them?</p>`,
      buttons: [
        { action: "clear", label: "Clear Conditions", default: true },
        { action: "modify", label: "Modify Conditions" },
        { action: "cancel", label: "Cancel" }
      ]
    });

    if (choice === "cancel" || choice === null) {
      proceedToConfigure = false;
    } else if (choice === "clear") {
      proceedToConfigure = false;
      const actors = canvas.tokens.controlled.map(t => t.actor).filter(a => a);
      for (const actor of actors) {
        const toClear = actor.statuses ?? new Set();
        for (const statusId of toClear) {
          try {
            await actor.toggleStatusEffect(statusId, { active: false });
          } catch (err) {
            console.error(`Failed to clear ${statusId} on ${actor.name}:`, err);
          }
        }
      }
      ui.notifications.info(`Cleared conditions on ${actors.length} token(s).`);
    }
    // choice === "modify" falls through to the configure dialog below
  }

  if (proceedToConfigure) {
    const sortedEffects = CONFIG.statusEffects
      .filter(e => e.id)
      .map(e => ({ id: e.id, label: game.i18n.localize(e.name ?? e.label ?? e.id) }))
      .sort((a, b) => a.label.localeCompare(b.label));

    const checkboxRows = sortedEffects
      .map(e => {
        const checked = currentStatuses.has(e.id) ? "checked" : "";
        return `<label class="cond-row" data-label="${e.label.toLowerCase()}" style="display:flex; align-items:center; gap:6px;">
                  <input type="checkbox" name="cond" value="${e.id}" ${checked}> ${e.label}
                </label>`;
      })
      .join("");

    const result = await DialogV2.wait({
      window: { title: "Set Conditions" },
      content: `
        <p style="font-size:12px; color:var(--color-text-dark-secondary,#666);">
          Checkboxes are pre-filled from ${reference?.name ?? "the first selected token"}. Applying will set every selected token to match exactly what's checked here.
        </p>
        <input type="text" id="cond-filter" placeholder="Search conditions..." style="width:100%; margin-bottom:6px; box-sizing:border-box;">
        <div id="cond-list" style="display:flex; flex-direction:column; gap:4px; max-height:300px; overflow-y:auto;">
          ${checkboxRows}
        </div>
      `,
      render: (event, dialog) => {
        const filterInput = dialog.element.querySelector("#cond-filter");
        const rows = dialog.element.querySelectorAll(".cond-row");
        filterInput?.addEventListener("input", () => {
          const val = filterInput.value.toLowerCase();
          rows.forEach(row => {
            row.style.display = row.dataset.label.includes(val) ? "flex" : "none";
          });
        });
      },
      buttons: [
        {
          action: "apply",
          label: "Apply",
          default: true,
          callback: (event, button) => {
            const checked = Array.from(button.form.querySelectorAll('input[name="cond"]:checked')).map(el => el.value);
            const all = Array.from(button.form.querySelectorAll('input[name="cond"]')).map(el => el.value);
            return { checked, all };
          }
        },
        { action: "cancel", label: "Cancel" }
      ]
    });

    if (result && result !== "cancel") {
      const actors = canvas.tokens.controlled.map(t => t.actor).filter(a => a);
      for (const actor of actors) {
        for (const statusId of result.all) {
          const shouldBeActive = result.checked.includes(statusId);
          try {
            await actor.toggleStatusEffect(statusId, { active: shouldBeActive });
          } catch (err) {
            console.error(`Failed to set ${statusId} on ${actor.name}:`, err);
          }
        }
      }
      ui.notifications.info(`Updated conditions on ${actors.length} token(s).`);
    }
  }
}
```
