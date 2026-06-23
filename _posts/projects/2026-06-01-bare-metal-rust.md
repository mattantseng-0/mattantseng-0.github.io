---
title: "Bare Metal Rust Application"
layout: gridlay
date: 2026-06-01
sitemap: false
permalink: /projects/bare_metal_rust_application
thumbnail: "/images/project_images/negative_friction_pendulum/raised_pendulum.png"
---

# Bare Metal Rust Application

<img src="../../images/project_images/bare_metal_rust/accel_data.gif" alt="Accelerometer Data" width="100%"/>

## Motivation

The goal of this exercise is to learn about using Rust to write baremetal firmware and an associated application to transmit and receive data from a sensor. The task accomplished in this project would be trivial in most languages such as Python. I would encourange the reader to at least read through this article from Adafruit to see the Circuit Python method of streaming accelerometer data: [CircuitPython Made Easy on Circuit Playground Express and Bluefruit](https://learn.adafruit.com/circuitpython-made-easy-on-circuit-playground-express/acceleration)


## Article Structure:

The article walks through the iterative development process. As such, the sections will require re-writing and re-structuring code to add new features. While the article itself will only discuss the main points of each step, a link to the relevant diff can be found at the top of each section.


- Phase 0: Project Setup:
    - Before we begin this project, we will spend some time setting up the workspaces necessary for the project and the bare metal development environment.
- Phase 1: Streaming Data
    - Once the initial project setup is complete, we can begin streaming dummy data to a serial data reader running on a laptop. 
- Phase 2: Talking to the Accelerometer
    - Once we have the data streaming functionality written, we will introduce the necessary code to talk to the LIS3DH accelerometer. 
- Phase 3: Receiving Data
    - Once the streaming functionality is written on the bare metal side, we will transition to writing a data receiver and plotter running on a laptop.
- Phase 4: Improving Performance
    - Finally, once the core functionality is fleshed out, we can focus on increasing the data transfer rates. 

## Hardware Overview

The hardware used for this project is a [Circuit Playground Express](https://www.adafruit.com/product/3333). 
- The processor on this device is a ATSAMD21
- The accelerometer on this device is a LIS3DH. 

## Crates Overview

### Bare Metal Crates

Below are the crates used for the hardware portions of this project:
- Hardware Abstraction Layer
    - [ATSAMD-HAL](https://crates.io/crates/atsamd-hal)
- Accelerometer Crate
    - [LIS3DH](https://crates.io/crates/lis3dh)

### Other Crates

Below are the crates used for data plotting and serialization:
- Data Serialization:
    - [Postcard](https://crates.io/crates/postcard)
- Plotting Crate:
    - [Egui Plot](https://crates.io/crates/egui_plot)

## Phase 0: Project Setup

### Workspace Initialization

Step Diffs: [Workspace Initialization](https://github.com/mattantseng-0/rust_cpx_accel_rx_tx/compare/224597734f99f05dc2a84728e3708da79ecf4a37...cb757fd6660f9731bcf793b1702d525b6f387df8)

Begin by creating Cargo workspaces with the following file structure: 
```
.
├── Cargo.toml
├── firmware
│   ├── Cargo.toml
│   └── src
│       └── main.rs
├── plotting_app
│   ├── Cargo.toml
│   └── src
│       └── main.rs
├── README.md
└── shared
    ├── Cargo.toml
    └── src
        └── lib.rs
```

In your top level Cargo.toml enter the following: 
```rust
[workspace]
resolver = "3"
members = [
    "firmware",
    "plotting_app",
    "shared"
]

```

Cargo Workspaces are important because the `plotting_app` is written for a laptop that has std whereas the `firmware` is written for a different target. Both the `plotting_app` and the `firmware` make use of the same `acc_msg` struct, so without seperate workspaces, the dependencies of the `firmware` and the `plotting_app` will get quite confused. 

For detailed information about Cargo Workspaces, I would recommend reading the related Rust documentation page: [14.3: Cargo Workspaces](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html)

### Message Setup

Step Diffs: [Message Definition](https://github.com/mattantseng-0/rust_cpx_accel_rx_tx/commit/ca24390728a566a0bab9c658cbbb368936823b56)

Create new files so that your `shared` directory has the following structure:

```
shared
├── Cargo.toml
└── src
    ├── lib.rs
    └── messages
        ├── acc_msg.rs
        ├── message_id.rs
        ├── message.rs
        └── mod.rs
```

Add the following to the `Cargo.toml`

```rust
[package]
name = "shared"
version = "0.1.0"
edition = "2024"

[features]
default = ["std"]
# When 'std' is active, it passes standard library support to serde
std = ["serde/std"] 

[dependencies]
serde = {version = "1.0.228", default-features = false, features = ["derive"] }
serde_repr = "0.1.20"
```

Importantly, we define the shared package to have an std feature. By marking this as a feature, we will be able to use shared in an std and no-std environments: necessary for hosted and baremetal code. 


Before we write the firmware or plotting application, let's begin by defining the message used to stream the accelerometer data from our CPX to the plotter. For our accelerometer data, we will define a serializable struct that contains the following information: 

```rust
#[derive(Serialize, Deserialize, Debug, Default)]
pub struct AccMsg {
    pub id: MessageId, 
    pub counter: u16,
    pub acc_x: i16,
    pub acc_y: i16,
    pub acc_z: i16, 
}
```

While the inclusion of the MessageID field is unecessary for this excercise, it will allow for future introductions of additional message types. 


## Phase 1: Firmware

Step Diffs: [Basic Serial Writing](https://github.com/mattantseng-0/rust_cpx_accel_rx_tx/commit/d14d39b990380cb7829f4a3f3f15f7d11b9e4c34)

Next, move to the firmware directory. It is important that we define the target architecture of the CPX device. 

Run the following command to add the necessary target architecture:
```
rustup target add thumbv6m-none-eabi
```

Next, follow the instructions from the ATSAMD repo to build/deploy a basic blinky script to the CPX [circuit_playground_express](https://github.com/atsamd-rs/atsamd/tree/master/boards/circuit_playground_express). This will help you get familiar with the hf2 tool. 

Once you have successfully built and deployed an example script to the CPX, we can continue to setup our development environment. 

Write the following to your `Cargo.toml`:

```rust
[package]
name = "firmware"
version = "0.1.0"
edition = "2024"

[dependencies]
panic-halt = "0.2"
cortex-m = { version = "0.7", features = ["critical-section-single-core"] }
circuit_playground_express = {version = "0.12.1", features = ["default","rtic", "usb"]}
rtic = { version = "2.1.1", features = ["thumbv6-backend"] }
usb-device = "0.3.2"
usbd-serial = "0.2.2"
```

The hardware setup for the USB virtual com port is derived from the usb_serial exampl in the ATSAMD crate: [usb_serial.rs](https://github.com/atsamd-rs/atsamd/blob/master/boards/circuit_playground_express/examples/usb_serial.rs). 

One of the most critical pieces of this example is the `poll_usb` function. For this project, we will use a stripped down version of this function. The core of this function is to periodically check the USB bus to see if there is a pending action. In our case, the pending action will be transmitted data from the accelerometer.

```rust
#[task(binds = USB, shared = [usb_bus, usb_serial])]
fn poll_usb(cx: poll_usb::Context) {
    let mut serial = cx.shared.usb_serial;
    let mut usb_bus = cx.shared.usb_bus;

    (&mut serial, &mut usb_bus).lock(|s, b| {
        if !b.poll(&mut [s]) {
            return;
        }
    })
}
```

Next, we define an async function to send data out of the virtual com port: 

```rust
#[task(shared = [usb_serial])]
async fn usb_tx_loop(mut cx: usb_tx_loop::Context)
{
    loop {

        let serialized_slice:&[u8] = b"123 456\n";

        cx.shared.usb_serial.lock(|serial| {
            let _ = serial.write(serialized_slice);
        });

        Mono::delay(100u64.micros()).await;
    }
}
```

In order for this function to work, it must have access to the usb_serial device. When you deploy this code, you should be able to see `123 456` being written in a serial listener: 


<img src="../../images/project_images/bare_metal_rust/basic_serial_data.png" alt="Dummy Serial Data" width="100%"/>

## Sending Dummy Accelerometer Data

Step Diffs: [CPX send dummy accel msg](https://github.com/mattantseng-0/rust_cpx_accel_rx_tx/commit/6bbbf7c5867254bcb0fd58806f1485a21a126183)

Now that our CPX is able to stream basic data, we can bring in our accelerometer message definition and send a dummy struct. Update your `Cargo.toml` as follows: 

```diff
[package]
name = "firmware"
version = "0.1.0"
edition = "2024"

[dependencies]
panic-halt = "0.2"
cortex-m = { version = "0.7", features = ["critical-section-single-core"] }
circuit_playground_express = {version = "0.12.1", features = ["default","rtic", "usb"]}
rtic = { version = "2.1.1", features = ["thumbv6-backend"] }
usb-device = "0.3.2"
usbd-serial = "0.2.2"
+ postcard = {version = "1.1.3", default-features = false, features = ["heapless", + "use-crc"]}
+ crc = "3.4.0"
+ serde = { version = "1.0", default-features = false, features = ["derive"] }
+ shared = { path = "../shared", default-features = false }
```

Note that this update to the `Cargo.toml` sets the `default-features = false` in our shared include. Recall that we made our `std` dependency configurable in `shared`. By setting `default-features = false`, we do not rely on `std` making shared compatible with baremetal.

Next, update the `usb_tx_loop` function as follows: 

```rust
#[task(shared = [usb_serial])]
async fn usb_tx_loop(mut cx: usb_tx_loop::Context)
{
    let counter: u16 = 0;
    let mut tx_msg = AccMsg::new();
    let mut output_buffer = [0u8; core::mem::size_of::<AccMsg>() + 4];
    loop {

        tx_msg.acc_x = (tx_msg.counter as i16);
        tx_msg.acc_y = (tx_msg.counter as i16) + 1i16;
        tx_msg.acc_z = (tx_msg.counter as i16) - 1i16; 

        let serialized_slice = postcard::to_slice_crc32(
            &tx_msg, 
            &mut output_buffer, 
            CRC_ALGO.digest()
        ).expect("Serialization failed");

        cx.shared.usb_serial.lock(|serial| {
            let _ = serial.write(serialized_slice);
        });

        Mono::delay(100u64.micros()).await;
    }
}
```

Once again, we can use a serial monitor tool to read our data. 

<img src="../../images/project_images/bare_metal_rust/dummy_serial_data.png" alt="Dummy Serial Data" width="100%"/>

In the updated code, the postcard crate is used to calculate a CRC on the contents of our message. The fact that we are using a virtual com port actually makes this operation redundant because the USB virtual com port already includes error checking.

## Communicating with the Accelerometer

Step Diffs: [Read and Stream Accel Data](https://github.com/mattantseng-0/rust_cpx_accel_rx_tx/commit/298a4d83478887ec7eb1d1a9927bb20f9dae2e43)

At this point in the project, we have all of the message plumbing in place to stream our formatted message out of the virtual com port. At this point in the project, we can introduce the accelerometer. 

In order to keep the tx code seperate from the accelerometer reading logic, we will introduce a data queue that contains the values read from the accelerometer. Then, we will update the `usb_tx_loop` to pull data from the queue and send it out. 

The following async function is responsible for reading from the accelerometer:

```rust
    #[task(local = [lis3dh], shared = [data_queue])]
    async fn poll_accel(mut cx: poll_accel::Context)
    {
        loop {


            if let Ok(sample) = cx.local.lis3dh.accel_raw() {
                cx.shared.data_queue.lock(|queue| {
                    let _ = queue.enqueue((sample.x, sample.y, sample.z));
                });
            } else {
                cx.shared.data_queue.lock(|queue| {
                    let _ = queue.enqueue((-1i16, -1i16, -1i16));
                });
            }
        
            Mono::delay(1u64.millis()).await;
        }
    }
```

Next, update the `usb_tx_loop` to deque the accelerometer data: 

```rust
    #[task(shared = [usb_serial, data_queue])]
    async fn usb_tx_loop(mut cx: usb_tx_loop::Context)
    {
        let mut counter: u16 = 0;
        let mut tx_msg = AccMsg::new();
        let mut output_buffer = [0u8; core::mem::size_of::<AccMsg>() + 4];
        loop {

            let data = cx.shared.data_queue.lock(|queue| queue.dequeue());

            if let Some((raw_x, raw_y, raw_z)) = data {
                tx_msg.acc_x = raw_x;
                tx_msg.acc_y = raw_y;
                tx_msg.acc_z = raw_z; 

            } else {
                tx_msg.acc_x = -1i16;
                tx_msg.acc_y = -1i16;
                tx_msg.acc_z = -1i16; 
            }

            let serialized_slice = postcard::to_slice_crc32(
                &tx_msg, 
                &mut output_buffer, 
                CRC_ALGO.digest()
            ).expect("Serialization failed");

            cx.shared.usb_serial.lock(|serial| {
                let _ = serial.write(serialized_slice);
            });
            
            counter = counter.wrapping_add(1);

            Mono::delay(100u64.micros()).await;
        }
    }
```

## 

### Old Content
--- 
## Phase 1: Streaming Data

The ATSAMD-HAL crate has some really good information to get started developing bare metal firmware in Rust. The starting point of the data-streaming portion of this project can be found in the usb_serial example for the CPX: [usb_serial.rs](https://github.com/atsamd-rs/atsamd/blob/master/boards/circuit_playground_express/examples/usb_serial.rs)

One interesting observation of this example code, is the use of `#[task]`

To begin modifying this example to meet our needs, I wrote the following async function: 
```rust
    #[task(shared = [usb_serial])]
    async fn usb_tx_loop(mut cx: usb_tx_loop::Context)
    {
        let counter: u16 = 0;
        let mut tx_msg = AccMsg::new();
        let mut output_buffer = [0u8; core::mem::size_of::<AccMsg>() + 4];

        tx_msg.counter = 0 as u16;
        tx_msg.acc_x = 1 as f32;
        tx_msg.acc_y = 2 as f32;
        tx_msg.acc_z = 3 as f32;

        loop {

            tx_msg.acc_x = (tx_msg.counter as f32);
            tx_msg.acc_y = (tx_msg.counter as f32) + 1f32;
            tx_msg.acc_z = (tx_msg.counter as f32) - 1f32; 


            let serialized_slice = postcard::to_slice_crc32(
                &tx_msg, 
                &mut output_buffer, 
                CRC_ALGO.digest()
            ).expect("Serialization failed");

            cx.shared.usb_serial.lock(|serial| {
                let _ = serial.write(serialized_slice);
            });
            tx_msg.counter = tx_msg.counter.wrapping_add(1);

            Mono::delay(100u64.micros()).await;
        }
    }
```
The above function trivially sends our data structure over the virtual com port. Using a serial data reader plugin in VSCode, we can verify that data is in fact getting sent to our laptop.

<img src="../../images/project_images/bare_metal_rust/dummy_serial_data.png" alt="Dummy Serial Data" width="100%"/>
---

## Phase 2: Talking to the Accelerometer

There is an example of using the LIS3DH written for the EdgeBadge [neopixel_tilt.rs](https://github.com/atsamd-rs/atsamd/blob/master/boards/edgebadge/examples/neopixel_tilt.rs)

The key code snippet from this example is: 

```rust
    let i2c = pins.i2c.init(
        &mut clocks,KiloHertz
        peripherals.SERCOM2,
        &mut peripherals.MCLK,
        &mut pins.port,
    );

    let mut lis3dh = Lis3dh::new(i2c, 0x19).unwrap();
    lis3dh.set_range(lis3dh::Range::G2).unwrap();
    lis3dh.set_datarate(lis3dh::DataRate::Hz_100).unwrap();

```

While this example shows us roughly how to setup the hardware, there are some differences that need to be addressed. In the pin definition of the EdgeBadge, we find that there are already these handy definitions for the `sda` and `scl` pins: 
```rust
define_pins!(
    ...
    // I2C (connected to LIS3DH accelerometer)
    /// STEMMA SDA
    pin sda = a12,
    /// STEMMA SCL
    pin scl = a13,
    ...
)
```

The pin definitions for the CPX are different and the only mention of the `sda` and `scl` pins are: 

```rust
pub mod pins {
    use super::hal;

    hal::bsp_pins!(
        ... 
        PA00 {
            name: accel_sda,
            aliases: {
                AlternateD: AccelSda
            }
        },
        PA01 {
            name: accel_scl,
            aliases: {
                AlternateD: AccelScl
            }
        },
        ...
    );
}     
```

Using these definitions, we can initialize the hardware for the LIS3DH with the following code snippet: 
```rust
let sda_pin: bsp::AccelSda = pins.accel_sda.into();
let scl_pin: bsp::AccelScl = pins.accel_scl.into();


let i2c_pads = i2c::Pads::new(sda_pin, scl_pin);

let i2c = i2c::Config::new(
    &mut peripherals.pm,
    peripherals.sercom1,
    i2c_pads, 
    freq,
).baud(Hertz::Hz(400000)) // Configure for 400kHz fast mode
.enable();

let mut lis3dh = Lis3dh::new_i2c(i2c, SlaveAddr::Alternate).unwrap();

lis3dh.set_range(lis3dh::Range::G2).unwrap();
lis3dh.set_datarate(lis3dh::DataRate::Hz_400).unwrap();
```

In addition to the hardware setup, we need to think about how we take our data from the accelerometer and send it over the serial interface. The accomplish this, we can leverage the `heapless` crate. Our `Shared` struct can be updated to include a Queue that holds the accelerometer data: 

```rust
#[shared]
struct Shared {
    // The LED could be a local resource, since it is only used in one task
    // But we want to showcase shared resources and locking
    red_led: bsp::RedLed,
    usb_bus: UsbDevice<'static, UsbBus>,
    usb_serial: SerialPort<'static, UsbBus>,
    data_queue: Queue<(i16, i16, i16), 8>, 
}
```

Importantly, this new definition will allow us to store up to 8 readings from the accelerometer that can then be read and sent over serial to the receiver.

Once the hardware has been initialized, we can write another async function whose responsibility is to poll the accelerometer and put the data into our queue. 

```rust
#[task(local = [lis3dh], shared = [data_queue])]
async fn poll_accel(mut cx: poll_accel::Context)
{
    loop {


        if let Ok(sample) = cx.local.lis3dh.accel_raw() {
            cx.shared.data_queue.lock(|queue| {
                let _ = queue.enqueue((sample.x, sample.y, sample.z));
            });
        } else {
            cx.shared.data_queue.lock(|queue| {
                let _ = queue.enqueue((-1i16, -1i16, -1i16));
            });
        }
    
        Mono::delay(1u64.millis()).await;
    }
}
```



---
## Phase 3: Receiving Data
---
## Phase 4: Improving Performance
