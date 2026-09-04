# Light Doggo - Development Journal

## August 21: Day 0 - Research and finding inspiration

ok so i've been obsessing over quadruped robots for weeks now. finally decided to actually do something about it. spent the entire day just watching youtube videos and reading hackaday posts about other people's builds. the goal is simple - make a walking robot that doesn't cost a fortune.

went down a massive rabbit hole looking at:
- what motors to use (BLDC vs servo vs steppers)
- what brain to run it (Raspberry Pi vs ESP32 vs STM32)
- how to talk to the motors (CAN bus seemed like the move)
- who's already done this before

found some cool projects for inspiration:
- Stanford Pupper - sick leg kinematics but way too expensive
- SpotMicro - open source Spot clone but uses servos which ain't it
- random chinese BLDC projects on github that showed CAN bus working

the game changer was finding these LingKong MG4010E motors on AliExpress. $35 each, 2.5 N.m torque, CAN bus at 1Mbps, and they come with encoders. 12 of them is $420 which is actually reasonable for a 12-DOF quadruped.

also realized the Raspberry Pi 5 is beefy enough to handle real-time motor control without needing a separate RTOS. that simplified everything.

by the end of the day i had a plan:
- RPi 5 as the brain
- 12 BLDC motors with CAN bus
- custom PCBs for power and aux stuff
- Python for control code
- MuJoCo for simulation

couldn't find my inspo image in downloads but the vision was clear - budget quadruped that actually works.

Time spent this session: 8 hours

---

## August 22: Repo setup and renaming everything

finally created the github repo and started pulling files from the upstream project i was basing this on. it was originally a micromouse project - completely different from what i needed. but the KiCad files, firmware structure, and simulation setup were solid starting points.

the hard part was figuring out what to keep and what to delete. the upstream had a bunch of maze-solving stuff - wheel encoders, line sensors - all useless for a quadruped. but the CAN bus code? the motor driver structure? that was gold.

```
git clone upstream-repo
# realize it's 80% useless code
# spend 2 hours just deleting things
# rename everything
```

ended up with this folder structure:
- `firmware/auxillary` - STM32 code for the LCD board
- `pcbs` - KiCad projects for both boards
- `cad` - OnShape exports and STL files
- `src/src` - main Python control code and simulation
- `docs` - GitHub Pages site

still can't believe i almost didn't use an upstream project. would've taken forever to write the CAN bus protocol from scratch.

![Initial CAD render](assets/light-doggo-cad-front-angle.png)

Time spent this session: 6 hours

---

## August 23: Documentation and CAD design

ok so i needed to write a proper README before i forgot all the technical details. started with the basics - RPi 5, 12x BLDC motors, 6s Li-Ion battery, 9-axis IMU - then got sucked into documenting the whole software architecture.

the architecture diagram took forever in Drawio but it's honestly one of the most useful things in the repo. shows how the main Python service talks to the CAN bus controller, which talks to the 12 motors across 4 separate CAN networks. each leg gets its own network which is overkill but keeps things clean.

spent the afternoon in OnShape modeling the frame and legs. the body is basically two parallel plates with mounting points for the RPi, battery, and all the electronics. tricky part was making everything fit while keeping the center of mass low.

the legs were the real pain. each leg has 3 DOF - hip abduction/adduction, hip flexion/extension, and knee flexion/extension. had to design mounting brackets so the motors align perfectly with the joint axes. used the motor dimensions from the MG4010E datasheet and made custom brackets that bolt directly to the motor face.

![Software architecture diagram](assets/light-doggo-block-diagram.png)

Time spent this session: 8 hours

---

## August 24: PCBs are finally done (maybe)

the Power Carrier board went through like 4 revisions before i was happy. main challenge was routing power to 12 motors while also creating 4 separate CAN bus networks. used solder jumpers so you can merge the front two legs into one network and the back two into another if you want to save on CAN channels.

the auxiliary board was even worse. originally planned to use the STM32 for everything - motor control, LCD, buzzer, neopixels. but then i realized the RPi 5 has enough GPIO and I2C pins to handle most of it directly. the only thing the STM32 really needs to do is the LCD driver since it's got that nice I2S interface.

lesson learned: just because you CAN use a separate microcontroller doesn't mean you SHOULD. extra chip = extra firmware = extra headaches.

![Power carrier PCB render](assets/light-doggo-power-carrier-v1-render.png)

Time spent this session: 7 hours

---

## August 25: KINEMATICS WORK IN SIMULATION!!

ok this was the big one. got the inverse kinematics running in MuJoCo and it actually looks like a dog walking?? still can't believe it.

the approach is pretty standard for quadrupeds - each leg has 3 joints (hip, knee, ankle) and you use inverse kinematics to figure out what angle each joint needs to be at to put the foot where you want it. the tricky part is generating the foot trajectories.

for walking in a straight line, i'm using Bezier curves to move each foot in a smooth arc. the control points define how high the foot lifts and how far forward it reaches. for turning, i project the linear trajectory onto a curved path using curvature projection. basically you take the straight-line trajectory and "bend" it around a circle.

the simulation started out... not great. the quadruped was bouncing around like crazy because i hadn't set the masses or inertial properties in the MuJoCo XML file. just used placeholder values. eventually got it stable enough to at least stand up and take a few steps.

the real validation came from overlaying the simulated quadruped on top of the real motor feedback in the UI. when the simulation matches the real thing, you know your kinematics are right.

![Bezier curve control points](assets/beizer-control-points-chart.png)

Time spent this session: 8 hours

---

## August 26: UI is actually usable now

got the React Native UI to the point where it's actually useful for debugging and development. the UI shows everything - joint positions, motor currents, sensor data, loop times, gamepad inputs - all in real time.

the coolest feature is the 3D visualization that shows both the simulated quadruped and the real motor feedback overlapped. you can literally see if your control code is making the robot do what you think it's doing.

built it with Expo so it runs on web during development (no need to deploy to a phone every time you change something). it'll also run on Android but without the 3D plot since WebGL support is spotty.

the hardest part was getting the WebSocket connection stable between the UI and the main Python service. kept dropping connections when too much data was flowing. ended up throttling the motor feedback to 50Hz instead of trying to push all 12 motors at full speed.

![Live UI with motor data](assets/light-doggo-ui-live.png)

Time spent this session: 6 hours

---

## August 27: Firmware and website launch

got the STM32 firmware compiling and running on the Black Pill dev kit. the auxiliary board powers the RPi, drives the LCD display, controls the buzzer, and handles the neopixel strips.

the LCD driver was the main challenge. it's an SPI display and the STM32's HAL library makes it really easy to write raw SPI commands but the initialization sequence for the LCD controller is like 50 lines of register writes. found a reference implementation in an Adafruit library and adapted it.

the buzzer was fun - it's just a simple PWM output but i added variable pitch so it can play different tones. eventually want to add "bark" sounds but for now it just beeps.

also set up GitHub Pages to host the project documentation and created the BOM.csv with all the parts and costs. total cost is somewhere around $700 which is honestly not bad for a 12-DOF quadruped.

![Auxiliary board render](assets/light-doggo-auxiliary-board-v1-render.png)

Time spent this session: 7 hours

---

## August 28: License, cleanup, and final polish

made the decision to switch from Apache 2.0 to MIT. main reason is simplicity - MIT is just one paragraph and everyone knows what it means. Apache 2.0 has all this extra stuff about patents and contributions that honestly doesn't matter for a small personal project.

the switch required deleting the old LICENSE file, creating a new one, and updating all the references in the README and documentation. also found some character encoding issues in the README that were causing weird rendering on GitHub - turns out i had some UTF-8 smart quotes that needed to be regular ASCII quotes.

spent the rest of the day cleaning up generated files and build artifacts that had somehow snuck into the repo. KiCad generates a ton of backup files and fabrication outputs that definitely shouldn't be in version control.

added the last few documentation sections - motor calibration process, gamepad controls, and the wireframe demo GIF. the motor calibration script (zero-motors.py) is actually really useful. it connects to each motor over CAN, reads the current encoder position, and lets you set the zero offset.

also added the wireframe demo GIF that shows the quadruped walking in a circle. it's just the simulation output but it looks cool and gives people an idea of what the project does.

![Wireframe demo](assets/light-doggo-wireframe-demo.gif)

Time spent this session: 6 hours
