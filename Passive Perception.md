# Passive Perception
Whispers passive perception to the DM. Only includes passive perception for characters included in a group actor called "Party". You can change the first line (const PARTY_NAME = "Party";) with what your group is called if you don't want to use that.

## Dependencies
- Foundry 14, build 364
- D&D5e Game System, version 5.3.3


## Code
```js
const PARTY_NAME = "Party";

const party = game.actors.getName(PARTY_NAME);
if (!party) {
  ui.notifications.error(`Could not find a party actor named "${PARTY_NAME}".`);
} else {
  const rawMembers = party.system.members ?? [];

  const members = rawMembers.map(m => {
    if (m instanceof Actor) return m;
    if (typeof m === "string") return fromUuidSync(m);
    if (m?.actor instanceof Actor) return m.actor;
    if (typeof m?.actor === "string") return fromUuidSync(m.actor);
    if (typeof m?.uuid === "string") return fromUuidSync(m.uuid);
    return null;
  }).filter(a => a);

  if (members.length === 0) {
    ui.notifications.warn("No party members could be resolved");
  } else {
    const rows = members
      .map(a => ({ name: a.name, pp: a.system?.skills?.prc?.passive ?? "?" }))
      .sort((a, b) => b.pp - a.pp)
      .map(r => `<li><strong>${r.name}:</strong> ${r.pp}</li>`)
      .join("");

    ChatMessage.create({
      content: `<h3>Passive Perception</h3><ul>${rows}</ul>`,
      whisper: [game.user.id]
    });
  }
}
```
