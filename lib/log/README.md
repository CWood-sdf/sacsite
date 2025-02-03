# logging system

the logging system is a system that produces a lot of data in a space where we don't really have that much storage.
Thus, the logging system takes several measures to ensure that no memory region is overflowed

## memory regions

The three (as of writing this) usable memory regions are heap memory, the Flash spi chip, and a MicroSd card

These regions ranked in terms of read/write speed are:

1. Heap
2. Flash SPI
3. MicroSd

The regions ranked in terms of _usable_ capacity are:

1. MicroSd
2. Flash SPI
3. Heap

Furthermore, the MicroSd is not able to be written to if the rocket is moving due to vibrations

## system constraints

the biggest constraint is the rocket's program cycle when it is in flight. The filters require a constant update rate so losing time to a chip read/write (rw) could possibly be fatal to the success of a flight if enough errors happen.

The secondary constraint is that we only have a limited amount of memory available in the fast rw regions

## other requirements

The logging system (in order to test it well) must also be runnable on a desktop. Thus, the logging system itself should not be directly calling the Arduino APIs to ensure portability

We also have to be able to radio data down to the ground station while the rocket's in flight

the flash spi chip stores data in "files" and you can only create up to 600 files on the chip.

The system must have absolutely no chance of failure: there should be zero possibility of the system overflowing heap memory. It is better that a packet is lost than the teensy crashes because of the logging system.

It should be impossible for logging updates to significantly delay the system

## system diagram

Now that the requirements and constraints have been specified, the system we landed upon is this:

- during initialization the baud of the flashspi chip is measured
- essential data is radio'd off to the ground station
  - Data needed for the ground station to be able to understand the rocket's data (like struct field names)
- Log items are created once per program cycle
- after each cycle, they are radio'd off to the ground station and also stored to an in memory buffer
- if the in memory buffer gets too full, and if there is enough time left in the program cycle, save data to the flash chip
- once the rocket lands all data should be read from the flash chip and forwarded to the sd card. the sd card data should also be verified to be correct once it is saved

The following is an approximation of the system diagram of the logging system

<pre style="line-height: 1">
<code>
┌───────┐
│startup│
└───┬───┘
    │     ┌──────────────────┐
    ├───► │measure flash baud│
    │     └──────────────────┘
    │     ┌────────────────┐
    └───► │radio setup info│
          └────────┬───────┘
            ▲      │
    no ─────┘      │
     ▲             ▼
     │     ┌─────────────────────┐
     └─────┤ground station reply │
           │or rocket launch?    │
           └────────┬────────────┘
                    │
      ┌─── yes ◄────┘
      ▼
 ┌───────────┐
 │ main loop │
 └───────┬───┘
     ▲   │    ┌───────────────┐
     │   ├──► │create log item│
     │   │    └───────────────┘
     │   │    ┌─────────────┐
     │   ├──► │fill log item│
     │   │    └─────────────┘
     │   │    ┌──────────────┐
     │   ├──► │radio log item│
     │   │    └──────────────┘
     │   │    ┌────────────────────────────────────────────────┐
     │   ├──► │in air and enough time to stash data? stash data│
     │   │    └────────────────────────────────────────────────┘
     │   │    ┌───────────────────────────────────────────────────────┐
     │   ├──► │landed and data has not been saved yet? save data to sd│
     │   │    └───────────────────────────────────────────────────────┘
     └───┘
</code>
</pre>

## drivers

In order that the log system can be tested on a computer and also be easily repurposed (for stuff like the ground station), all hardware interactions are abstracted out into a "Device" interface. (see `lib/log/device.h`)

However, in order that the logging system might remain fast while still using interfaces (because interfaces in c++ introduce a little overhead), the driver interface is _very_ closely tied to the actual hardware we are using on the board.

A "device" must support multiple operations:

- send messages via radio synchronously and asynchronously
- synchronously receive a message from the radio if one is available
- send an item to a stash (flash SPI chip) (all items of the same type have the same size)
- get an item from the stash by the id number
- delete all stashes
- write lines to the sd card
- confirm the last lines written to the sd card
