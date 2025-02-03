# Spaceport America Cup

This repo is for the competition code for spaceport america cup 2024-2025

Spaceport america cup is a competition to see which team can get their rocket as close to 10k feet as possible. Since wind conditions, weather conditions, and motor power output can all vary significantly enough to affect rocket end height, we are planning to use an airbrakes system to slow the rocket down

The main feature planned is an airbrakes controller which will need to filter sensor data to get the rocket's exact position and velocity, predict the rocket's end location, and use all the data to determine how long the airbrakes must be deployed for.

The other features planned are a simulation framework to test the program without launching the rocket and a ground station to view rocket state when it is in the air.

The guidance code is (probably) restricted from export under US law, so please do not share this code with anyone that is not inside the club.

## Things we need to account for

- logger never connects to radio before rocket launch
- miss a sensor packet during flight
  - we could rebuild kalman filter matrices with new deltat
    - preferrable if we have enough time
  - or generate a fake packet off of what the kalman filter predicts

## libs/ vs src/ folder

Ideally, the entire program is testable.
Since we cant really test the algorithms with real sensors, we will have to simulate sensors in both simulink and rocketpy.
This means that the sensor interface code must be sepearated from the actual algorithms that use the sensor data.

So, the question of what do you put in the libs/ versus the src/ folder is not super complicated.
Basically, put any algorithms that need to be tested in the libs/ folder and make sure they have a clear and defined interface with the rest of the program (a good example of this is the madgwick filter we have in the madgwick branch right now).
Put any sensor interface code in the src/ folder because we can't really test that anyway.

For the logging subsystem and other things like that we _probably_ want to keep it in the src/ folder because I dont think tests for that without the pcb wont be very productive.

## Things to do for program

### Boot up sequence

This will probably be done later and we will need to talk to structures to ensure everything's correct

This is essentially all code that will run between powerup and motor burnout

- [ ] A state machine that keeps track of everything
- [ ] Some self test of all components
- [ ] Wait for launchpad?
- [ ] Launchpad ready
- [ ] Detect launch
- [ ] Wait for burnout (or simple time delay for however long the motor is supposed to run)

### Airbrakes interface

- [ ] PI loop for AB angle
- [ ] Simulink for PI tuning

### Sensor interface

- [ ] Actual sensor data get (will need to wait for board completion)
- [ ] Simulink sensor interface

### Filters

- [ ] Madgwick filter
- [ ] Kalman filter
- [ ] Kalman filter state update
- [ ] Simulink to test Madgwick
- [ ] Simulink to test Kalman

### CFD Sims

This is data that flight performance will give us

- [ ] Drag coeff for each velocity and altitude

### Prediction

- [x] One axis engine
- [ ] Extremely fast (should be less than 1ms per cycle IN THE WORST CASE)
- [ ] Full vector simulation
- [ ] Extremely high accuracy AND precision
- [ ] Determine simulation variance
- [ ] Error generator for PI Loop

### LOGGING

Need logs of _EVERYTHING_ that goes on on board (aka all sensor data received, all filter output, all predictions, and all expected vs actual airbrake states)

- [ ] _no_ writing to SD card during flight (too much vibration)
- [ ] Prolly need a memory efficient binary format so that the data doesnt take too much space
- [ ] Need an sd test to confirm whether the sd can be safely written to
- [ ] Need checksums of the data
- [ ] Write to sd card after burnout

### Simulink

- [ ] Put together a simulator in simulink that can run all the above code

## Things to do for PCB

### Parts

- [ ] Vector nav

## Environment

### Software

#### Github

All our code will be hosted on github, the link to the repository for this year will be shared once yall become members of the rocketry organization

#### Git

Git will be used for managing the source code. It can be downloaded for windows [here](https://gitforwindows.org/), for linux, it can be installed via package manager, on mac, the install instructions can be found [here](https://git-scm.com/download/mac)

#### Platformio

We use platformio for connecting and uploading software to the circuit board.

If you use vscode, platformio has a [vscode extension](https://platformio.org/install/ide?install=vscode).

If you use neovim, you should first install the platformio [cli](https://platformio.org/install/cli) (this can probably also be installed through your os's package manager), then there is a [lua plugin](https://github.com/anurag3301/nvim-platformio.lua) that I personally have not used yet, but it seems very promising (last year, I followed a modified version of [these instructions](https://docs.platformio.org/en/latest/integration/ide/vim.html) to get lsp support, but I will be trying out nvim-platformio to see how well it works and I might end up contributing to it to get dap support).

#### Drivers

For sensor drivers, there are a bunch of free and open source drivers published by [adafruit](https://www.adafruit.com/). Some partial usages of some of these drivers exist in our competition code from [last year](https://github.com/UVARocketry/SpaceportAmericaCup23-24/blob/main/gyrodriver/src/main.cpp).

#### RocketPy

We will probably be using [rocketpy](https://docs.rocketpy.org/en/latest/) to test our program, though this might change later in the year. You dont have to worry too much about this for now as this wont become important until we start getting our guidance code working

### Hardware

#### Sensors

To order sensors we will be using [digikey](https://www.digikey.com/en/products/result?s=N4IgTCBcDaIOYAcDOIC6BfIA) and [mouser](https://www.mouser.com/). Right now, the sensors we will probably end up using are a GPS, IMU, and altimeter/barometer.

#### Altium

To actually design the board, we will use [altium designer](https://www.altium.com/altium-designer?srsltid=AfmBOopCOouTD5QUP6iWCQxnMLjkHSUI74x4dRBxMl9jCRZfiPy_f8Zu).
