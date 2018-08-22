# WolkConnect C
WolkAbout C99 Connector library for connecting devices to [WolkAbout IoT platform](https://demo.wolkabout.com/#/login).

WolkConnect-C is transportation layer agnostic which means it is up to the user of the library to open socket to WolkAbout IoT platform,
configure SSL if desired, and forward read/write implementation to WolkConnect-C Connector.
WolkConnect-C Connector is not thread safe, and is written with cooperative scheduling in mind.
This allows WolkConnect-C Connector to work on wide variety of systems, bare metal to OS based ones.

Given examples are intended to be run on Debian based system.

Supported protocol(s):
* JSON single

Prerequisite
------
Following tools/libraries are required in order to build WolkConnect-C Connector

* cmake - version 2.8 or later

Former can be installed on Debian based system from terminal by invoking

`apt-get install cmake`

When dependencies are installed, Unix Makefiles build system can generated by invoking `./configure`

Generated build system is located inside 'build' directory

WolkConnect-C Connector is built to `out/lib` directory, and example application is built to `out/bin` directory.

Example Usage
-------------
**Initialize WolkConnect-C Connector**

Create a device on WolkAbout IoT platform by importing `simple-example-manifest.json` located in `examples/simple/`.
This manifest fits `simple` example and demonstrates the sending of a temperature sensor reading.

```c
static const char *device_key = "device_key";
static const char *device_password = "some_password";
static const char *hostname = "api-demo.wolkabout.com";
static int portno = 1883;

/* Sample in-memory persistence storage - size 1MB */
static uint8_t persistence_storage[1024*1024];

/* WolkConnect-C Connector context */
static wolk_ctx_t wolk;

.
.
.

wolk_init(&wolk,                                             /* Context */
          send_buffer,                                       /* See send_func_t */
          receive_buffer,                                    /* See recv_func_t */
          actuation_handler,                                 /* Actuation handler        - see actuation_handler_t */
          actuator_status_provider,                          /* Actuator status provider - see actuator_status_provider_t */
          device_key, device_password,                       /* Device key and password provided by WolkAbout IoT Platform upon device creation */
          PROTOCOL_JSON_SINGLE,                              /* Protocol specified for device */
          actuator_references,                               /* Array of actuator references */
          num_actuator_references);                          /* Number of actuator references */

wolk_init_in_memory_persistence(&wolk,                       /* Context */
                                persistence_storage,         /* Address to start of the memory which will be used by persistence mechanism */
                                sizeof(persistence_storage), /* Size of memory in bytes */
                                true);                       /* If storage is full overwrite oldest item when pushing */
```
Considering that WolkConnect C Connector is transportation layer agnostic, it is up to the user of it to open connection to
WolkAbout IoT Platform, optionally setup TLS, and forward read/write callbacks WolkConnect-C Connector in initialization
step.

See `send_func_t` and `send_func_t` in `sources/wolk_connector.h`

**Establishing connection with WolkAbout IoT platform:**
```c
wolk_connect(&wolk);
```
**Adding sensor readings:**
```c
wolk_add_numeric_sensor_reading(&wolk, "T", 23.4, 0);
```
**Data publish strategy:**

Sensor reading is pushed to WolkAbout IoT platform on demand by calling
```c
wolk_publish(&wolk);
```

**Disconnecting from the platform:**
```c
wolk_disconnect(&wolk);
```

**Cooperative scheduling:**

Fuction `wolk_process(wolk_ctx_t *ctx)` is non-blocking in order to comply with cooperative scheduling,
and it must to be called periodically.

```c
wolk_process(&wolk, 5);
```

**Data persistence:**

WolkConnect-C provides mechanism for persisting data in situations where readings can not be sent to WolkAbout IoT platform.
Default implementation can work with in-memory or memory mapped storage.

Persisted readings are sent to WolkAbout IoT platform, in batches, on publish function call.

In cases when provided persistence implementation is suboptimal, one can use custom persistence by providing custom implementation.

```c
wolk_init_custom_persistence(&wolk,
                             persistence_push_impl,
                             persistence_peek_impl, persistence_pop_impl,
                             persistence_is_empty_impl);
```

For more info on persistence mechanism see `sources/persistence.h` and `sources/in_memory_persistence.h` files.

**Additional functionality**

WolkConnect-C library has integrated additional features which can perform full WolkAbout IoT platform potential. Read more about full feature set example [HERE](https://github.com/Wolkabout/WolkConnect-C/tree/master/examples/full_feature_set).