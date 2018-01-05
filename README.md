# MessageSerial 
Arduino and Linux compatible packet-based serial communication library. It was originally created based on [PacketSerial by bakercp](https://github.com/bakercp/PacketSerial) and extended with flexible message payload functionality and CRC error checking.
* Uses COBS or SLIP protocol to reduce framing errors and CRC for general error detection.
* Supports flexible message definition.

More info in my blog post: https://BoredomProjects.net/index.php/projects/robot-navigation-using-stereo-vision

## Usage

### Initialization
First, we must configure hardware serial port and instantiate Message serial object with it. Both Arduino and Unix are supported:

#### _Linux_
```
// configure hardware serial port
serial::Serial uart("/dev/ttyAMA0", 115200, serial::Timeout(0,0,0,250,0));

// MessageSerial object will use the above port
MessageSerial serial(uart);
```

#### _Arduino_
```
MessageSerial serial(Serial1);
```

Next, some message data types should be created, for example, using `struct`:

```
typedef struct {
    float theta;
    float dx;
    float dth;
    uint16_t dt_ms;
} tOdom;

typedef struct {
    char str[100];
} tStr;
```

Then, we create some Message objects, by referencing above data types using class template:

```
Message<tOdom,5> odom(serial);
Message<tStr,1> text(serial);
```
where second template parameter is a numeric constant for message ID.
* __NOTE:__ Text strings must have ID = 1, to use null-terminator for data size.

### Message processing loop
Next, we need to ensure the following function gets called as often as possible. It is driving our message processing loop.
```
serial.update();
```

### Communication
Finally, we can send and receive our data, like this:

#### _Sending_
```
odom.data.theta = 3.14;
odom.data.dx = 0.4;
odom.data.dth = 0.0;
odom.data.dt_ms = 1234;
odom.send();

strncpy(text.data.str, "Hello World!", sizeof(text.data.str));
text.send();
```

#### _Receiving_
```
if( odom.available() )
{
   double theta = odom.data.theta;
   double dx = odom.data.dx;
   double dth = odom.data.dth;
   double dt = odom.data.dt_ms / 1000.0;
   odom.ready();
}

if( text.available() )
{
   ROS_INFO(text.data.str);
   text.ready();
}
```
Here, after the data has been copied out, the `ready()` method clears this packet and prepares the engine to receive next one.


