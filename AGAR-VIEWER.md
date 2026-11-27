# Agar Viewer

**Milestone · 27 November 2026**

A browser-based NABA projection sketch. Each participant controls a solid colored circle by sending positions over MQTT. When circles meet, they form a larger circle that sends their colors outward in repeating ripples.

## Open and test

1. Open the [Agar Viewer](https://marcobrianza.github.io/mqtt-websocket-test/agar-viewer.html).
2. Click **Test circles** once for separate circles, twice for a two-color ripple, and three times for a three-color ripple. The fourth click returns to separate circles.
3. Click **Fullscreen** for projection. Press **H** to toggle the controls or **Esc** to exit fullscreen.

The viewer is a single [HTML file](agar-viewer.html). It can also be opened locally in a modern browser. Live MQTT requires network access to the broker and to the CDN that serves MQTT.js.

## Send a position

Use the [MQTT debugger](https://marcobrianza.github.io/mqtt-websocket-test/debug.html) or another MQTT publisher.

| Setting | Value |
| --- | --- |
| Browser broker | `wss://mqtt.marcobrianza.it:8084/mqtt` |
| Viewer subscription | `naba/position/+` |
| Example topics | `naba/position/1`, `naba/position/2`, `naba/position/3` |
| Example payload | `0.25 0.70` |

Send two space-separated coordinates as plain text, **x then y**. Each ranges from 0 to 1. The origin is the top-left, x increases to the right, and y increases downward. Use a different numeric topic suffix for each participant. The controls show the connection status and most recent received position.

To trigger a ripple with the debugger, send `0.50 0.50` to `naba/position/1`, then send `0.51 0.50` to `naba/position/2`. Send `0.80 0.50` to the second topic to separate them again.

## How the sketch works

- Each numeric topic gets a color for the browser session. A valid update moves the circle and restarts its 30-second timeout. Send updates more often than every 30 seconds to keep a stationary circle visible.
- Touching or overlapping circles form a group, including chains where one circle connects two others. The group is drawn at the average position of its members. A two-member group has three times the diameter of a single circle and grows as more members join.
- The group's colors repeat across its radius as bands that travel from the center to the edge. A new color enters at the center about every 450 ms. Position updates keep the animation running while the group's membership stays the same.
- When circles move apart or a position expires, the viewer recalculates the groups. An isolated circle is solid again.

The HTML handles the MQTT subscription, coordinate parsing, timeouts, and grouping. A CSS animation moves the repeating radial gradient on each merged circle.
