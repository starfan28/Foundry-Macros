# Long Rest
A simple macro to rest the party. Only applies to characters included in a group actor called "Party". You can change the first line (const PARTY_NAME = "Party";) with what your group is called if you don't want to use that.

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
    ui.notifications.warn("No party members could be resolved.");
  } else {
    for (const actor of members) {
      await actor.longRest({ dialog: false, chat: true, newDay: true });
    }
    ui.notifications.info(`Long rest applied to ${members.length} party member(s).`);
  }
}
```
