# Group Saving Throw
Rolls a saving throw for all of the selected tokens and whisper it to the DM. Choose the saving throw type, and add a DC if you want. Or don't. 

## Dependencies
- Foundry 14, build 364
- D&D5e Game System, version 5.3.3


## Code
```js
const DialogV2 = foundry.applications.api.DialogV2;

if (canvas.tokens.controlled.length === 0) {
  ui.notifications.warn("Select at least one token first.");
} else {
  const abilityOptions = Object.entries(CONFIG.DND5E.abilities)
    .map(([key, a]) => `<option value="${key}">${a.label}</option>`)
    .join("");

  const result = await DialogV2.wait({
    window: { title: "Group Saving Throw" },
    content: `
      <div style="display:flex; flex-direction:column; gap:8px;">
        <label>Ability
          <select name="ability">${abilityOptions}</select>
        </label>
        <label>DC (Optional: leave blank for an unmarked roll)
          <input type="number" name="dc" placeholder="e.g. 15">
        </label>
      </div>
    `,
    buttons: [
      {
        action: "roll",
        label: "Roll",
        default: true,
        callback: (event, button) => ({
          ability: button.form.elements.ability.value,
          dc: button.form.elements.dc.value ? Number(button.form.elements.dc.value) : null
        })
      },
      { action: "cancel", label: "Cancel" }
    ]
  });

  if (result && result !== "cancel") {
    const actors = canvas.tokens.controlled.map(t => t.actor).filter(a => a);
    const originalRollMode = game.settings.get("core", "rollMode");

    try {
      await game.settings.set("core", "rollMode", "gmroll");

      for (const actor of actors) {
        try {
          await actor.rollSavingThrow(
            { ability: result.ability, target: result.dc ?? undefined },
            { configure: false },
            { rollMode: "gmroll" }
          );
        } catch (err) {
          console.error(`Saving throw failed for ${actor.name}:`, err);
          ui.notifications.error(`Saving throw failed for ${actor.name} : see console.`);
        }
      }
    } finally {
      await game.settings.set("core", "rollMode", originalRollMode);
    }

    ui.notifications.info(`Requested ${CONFIG.DND5E.abilities[result.ability].label} saves from ${actors.length} token(s) whispered to GM.`);
  }
}
```
