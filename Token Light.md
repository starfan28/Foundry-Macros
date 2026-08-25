# Token Light
Adds a light source to the selected token. Allows you to choose dim/bright light radius, and effect. If a light source is already applied to the token, then it asks if you want to modify or remove the light setting.

## Dependencies
- Foundry 14, build 364
- D&D5e Game System, version 5.3.3


## Code
```js
const DialogV2 = foundry.applications.api.DialogV2;

if (canvas.tokens.controlled.length === 0) {
  ui.notifications.warn("Select at least one token first.");
} else {
  const token = canvas.tokens.controlled[0];
  const current = token.document.light;
  const isLit = (current.dim ?? 0) > 0 || (current.bright ?? 0) > 0;

  let proceedToConfigure = true;

  if (isLit) {
    const choice = await DialogV2.wait({
      window: { title: "Token Already Has Light" },
      content: `<p>This token currently has light (dim: ${current.dim}, bright: ${current.bright}).</p>
                 <p>Remove it, or change the settings?</p>`,
      buttons: [
        { action: "remove", label: "Remove Light", default: true },
        { action: "change", label: "Change Settings" },
        { action: "cancel", label: "Cancel" }
      ]
    });

    if (choice === "cancel" || choice === null) {
      proceedToConfigure = false;
    } else if (choice === "remove") {
      proceedToConfigure = false;
      for (const t of canvas.tokens.controlled) {
        await t.document.update({
          light: { dim: 0, bright: 0, animation: { type: null } }
        });
      }
      ui.notifications.info("Light removed.");
    }
    // choice === "change" falls through to the configure dialog below
  }

  if (proceedToConfigure) {
    const animOptions = Object.entries(CONFIG.Canvas.lightAnimations)
      .map(([key, cfg]) => `<option value="${key}" ${current.animation?.type === key ? "selected" : ""}>${cfg.label}</option>`)
      .join("");

    const result = await DialogV2.wait({
      window: { title: "Configure Token Light" },
      content: `
        <div style="display:flex; flex-direction:column; gap:8px;">
          <label>Dim Radius (ft)
            <input type="number" name="dim" value="${current.dim ?? 20}" step="5">
          </label>
          <label>Bright Radius (ft)
            <input type="number" name="bright" value="${current.bright ?? 20}" step="5">
          </label>
          <label>Animation Style
            <select name="animation">
              <option value="">None</option>
              ${animOptions}
            </select>
          </label>
        </div>
      `,
      buttons: [
        {
          action: "apply",
          label: "Apply",
          default: true,
          callback: (event, button) => ({
            dim: Math.max(0, Number(button.form.elements.dim.value) || 0),
            bright: Math.max(0, Number(button.form.elements.bright.value) || 0),
            animation: button.form.elements.animation.value
          })
        },
        { action: "cancel", label: "Cancel" }
      ]
    });

    if (result && result !== "cancel") {
      for (const t of canvas.tokens.controlled) {
        await t.document.update({
          light: {
            dim: result.dim,
            bright: result.bright,
            angle: 360,
            animation: result.animation ? { type: result.animation, speed: 5, intensity: 5 } : { type: null }
          }
        });
      }
      ui.notifications.info(`Light updated: dim ${result.dim}, bright ${result.bright}${result.animation ? `, ${result.animation}` : ""}`);
    }
  }
}
```
