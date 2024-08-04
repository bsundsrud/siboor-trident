# Errata from assembly process

Below is the list of issues I ran into while assembling where I had to deviate from the assembly instructions.

- 1 roll of 3mm foam tape was missing from the BOM
- Printed part for power inlet is the wrong size.  Corrected STL offered via discord, they also sent me a reprint.
- No notes on how to properly sync AWD motors, all 4 included motors are D-posts instead of 2 D-posts and 2 round posts
- M3x8 SHCS screws for the HDMI screen connecting the mount to the 2020 extrusion are too short to grip the KlipperScreen, switched to M3x12 SHCS instead
- One or two of the pre-drilled holes on the base plate were too small, rectified by drilling out.
- The Siboor nameplate that goes on the bed frame was loose and rattly, I just omitted it.
- CW2 tensioning spring was too long and meant tension was always too high, rectified by trimming the spring to 12mm (number via official stealthburner manual)
- Drag chain swivel mount would bind and not utilize the bearing as listed in the instructions, fixed by adding 1-2 M3 shims between the bearing and the printed part.
- Peeling the sheath off the CAN cable is prone to error and nicking the wires inside
- Several included wires meant for screw terminals were only tinned with solder, replaced with ferrules
- Stepper Driver fan cable too short to reach designated port, either move to a closer port or make an extension cable
- PC4-01 pneumatic connector on the fume pack has an internal spacer that doesn't allow PTFE tubing to be shoved all the way through
- Back plate is missing a drill hole for the bottom middle screw for the fume pack.  I added this by measuring the distance from far side of a side hole to the middle of the bottom hole on the fume pack, and then using the calipers to score an X on the panel from each of the existing side holes, then drilling a hole at the X.
- TPU washers for the fume pack seem... highly optimistic.  I skipped them.
- Reverse bowden clips for the drag chain are unlikely to stay on for very long
- Clickyclacky door printed parts had too-small tolerances for the bushings (pin tolerances were also tight but could be forced in).  Holes for the bushings needed to be enlarged via drill.
