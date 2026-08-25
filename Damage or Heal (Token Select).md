# Macro Name
A simple macro to apply damage or healing to tokens.

## Dependencies
- Foundry 14, build 364
- D&D5e Game System, version 5.3.3


## Code
```js
const DialogV2 = foundry.applications.api.DialogV2;

if (canvas.tokens.controlled.length === 0) {
  ui.notifications.warn("Select at least one token first.");
} else {
  const result = await DialogV2.wait({
    window: { title: "Apply Damage / Healing" },
    content: `
      <div style="display:flex; flex-direction:column; gap:8px;">
        <label>Amount
          <input type="number" name="amount" min="0" value="0" autofocus>
        </label>
        <label>
          <input type="radio" name="mode" value="damage" checked> Damage
        </label>
        <label>
          <input type="radio" name="mode" value="heal"> Healing
        </label>
      </div>
    `,
    buttons: [
      {
        action: "apply",
        label: "Apply",
        default: true,
        callback: (event, button) => ({
          amount: Math.abs(Number(button.form.elements.amount.value) || 0),
          mode: button.form.querySelector('input[name="mode"]:checked').value
        })
      },
      { action: "cancel", label: "Cancel" }
    ]
  });

  if (result && result !== "cancel" && result.amount > 0) {
    const actors = canvas.tokens.controlled.map(t => t.actor).filter(a => a);

    for (const actor of actors) {
      try {
        if (result.mode === "damage") {
          await actor.applyDamage(result.amount);
        } else {
          await actor.applyDamage([{ value: result.amount, type: "healing" }]);
        }
      } catch (err) {
        console.error(`Failed to apply ${result.mode} to ${actor.name}:`, err);
        ui.notifications.error(`Failed for ${actor.name} — see console.`);
      }
    }

    ui.notifications.info(`Applied ${result.amount} ${result.mode} to ${actors.length} token(s).`);
  }
}
```
