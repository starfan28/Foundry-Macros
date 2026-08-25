# Group Initiative
Rolls initiative for a group of NPCs. Specifically ignores tokens controlled by players, so you can highlight every token on the map and it won't mess with player initiative.

## Dependencies
- Foundry 14, build 364
- D&D5e Game System, version 5.3.3


## Code
```js
Rolls initiative for a group of NPCs. Specifically ignores tokens controlled by players, so you can highlight every token on the map and it won't mess with player initiative.

```js
if (canvas.tokens.controlled.length === 0) {
  ui.notifications.warn("Select at least one token first.");
} else {
  const npcTokens = canvas.tokens.controlled.filter(t => !t.actor?.hasPlayerOwner);

  if (npcTokens.length === 0) {
    ui.notifications.warn("No NPC tokens in the current selection, everything selected is player-owned.");
  } else {
    if (!game.combat) {
      await Combat.create({ scene: canvas.scene.id, active: true });
    }
    const combat = game.combat;

    const combatantIds = [];
    for (const token of npcTokens) {
      let combatant = combat.combatants.find(c => c.tokenId === token.document.id);
      if (!combatant) {
        const created = await combat.createEmbeddedDocuments("Combatant", [{
          tokenId: token.document.id,
          sceneId: token.document.parent.id,
          actorId: token.actor.id,
          hidden: token.document.hidden
        }]);
        combatant = created[0];
      }
      combatantIds.push(combatant.id);
    }

    await combat.rollInitiative(combatantIds);
    ui.notifications.info(`Rolled initiative for ${combatantIds.length} NPC token(s).`);
  }
}
```
