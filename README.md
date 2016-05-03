# hx711_SPI
HX711 weight scale interface with Raspberry PI using SPI communication

To use this code, you need to install the c-periphery library (https://github.com/vsergeev/c-periphery).
To compile the code, put the code into c-periphery folder and compile as gcc HX711_SPI.c -I /src periphery.a -o spi

Also you need to activate SPI on your raspberry, this tutorial show how: https://learn.sparkfun.com/tutorials/raspberry-pi-spi-and-i2c-tutorial

To describe how to connect the pins, I'll be based on this two images:

Raspberry: http://www.instructables.com/files/orig/FN1/YEKZ/HN824U1E/FN1YEKZHN824U1E.jpg

HX711: (1) https://cdn.sparkfun.com/assets/parts/1/0/3/9/5/13230-04.jpg
       (2) https://raw.githubusercontent.com/intel-iot-devkit/upm/master/docs/images/hx711.jpeg

Hardware Hookup:
Raspberry 3.3V (pin 1)  -> VCC  (It's better to use 3.3V as the RASPBERRY GPIOs are 3.3V, using 5v you can damage these pins!)
Raspberry MISO (pin 21) -> DOUT
Raspberry MOSI (pin 19) -> PD_SCK
Raspberry GND  (pin 20) -> GND 


The four wires coming out from the wheatstone bridge on the load cell are usually:
Excitation+ (E+) or VCC is red
Excitation- (E-) or ground is black.
Output+ (O+), Signal+ (S+)+ or Amplifier+ (A+) is white
O-, S-, or A- is green or blue

**https://learn.sparkfun.com/tutorials/load-cell-amplifier-hx711-breakout-hookup-guide?_ga=1.114317697.789230018.1460136273

After execute the code, it will calculate the tare.
To actually be able to convert to any usable reading, you would have to calibrate the scale using a known weight.

(1)Start the code WITHOUT a weight on the scale
You will see something like that:
Wait: Getting Tare
Offset: 16724211
Starting...

(2)Once readings are displayed place the weight on the scale
I used 3kg and the code returned this:
Value: -426470 grams
Value: -426470 grams
Value: -426485 grams
Value: -426485 grams

(3)Get the reading value and divide by the weight value.

-426485/3000 which is approximated -142.
Your calibration factor may be very positive or very negative. It all depends on the setup of your scale system
and the direction the sensors deflect from zero state

(4)Set the SCALE value.

(5)Compile again and run!

The digital interface to the HX711 is a proprietary SPI-like interface that uses a clock (SCK) and data (DOUT) line.
Unfortunately, it needs at least 25 clock pulses, but the SPI hardware is only capable of doing in increments of 8-bits so I wrote a function to bit-bang the interface.
I opted to use the MOSI data line from SPI peripheral to generate a clock pulse by writing the value “0xAA”, which will go high 4 times each byte, so we need 6 bytes of
to achieve 24 pulses, after I send one byte to set the gain, which will be (0x80) for 124, 0xA8 for 64 and 0xA0 for 32!

