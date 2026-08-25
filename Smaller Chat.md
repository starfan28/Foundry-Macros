# Chat Modifications

## Dependencies
- Foundry 14, build 364
- D&D5e Game System, version 5.3.3
- **Custom CSS** *created with Version 2.4.4* [Link](https://foundryvtt.com/packages/custom-css)


## Code
```css
.chat-message {
  padding: 4px 6px;
  margin-bottom: 2px;
  font-size: 12px;
  position: relative;
}

.chat-message .dnd5e2.chat-card {
  font-size: 12px;
}

.chat-message .dnd5e2.chat-card .card-header {
  padding: 2px 4px;
  gap: 4px;
}

.chat-message .dnd5e2.chat-card .card-content {
  padding: 2px 4px;
  font-size: 12px;
  line-height: 1.3;
}

.chat-message .dnd5e2.chat-card button {
  padding: 2px 6px;
  font-size: 11px;
  min-height: unset;
}

.chat-message .dice-roll .dice-formula,
.chat-message .dice-roll .dice-total {
  font-size: 12px;
  padding: 2px 4px;
}

/* ===== Hide Portrait Image =====
  Hides the portrait image of the character to make the box smaller.
*/

.chat-message img {
  display: none;
}

/* ===== Message Type Labels =====
  These can be removed if you don't want the "Whisper" and "Blind" labels added to messages.
  I added these because I'm colorblind
*/

.chat-message.whisper::after {
  content: "WHISPER";
  position: absolute;
  top: 22px;
  right: 6px;
  font-size: 14px;
  font-weight: bold;
  color: #005fcc;
}

.chat-message.blind::after {
  content: "BLIND";
  position: absolute;
  top: 22px;
  right: 6px;
  font-size: 14px;
  font-weight: bold;
  color: #cc7a00;
}
```
