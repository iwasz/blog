---
ID: 206
post_title: 'Motorcycle black box. Part 1 : data acquisition with Arduino Mega.'
author: admin
post_excerpt: ""
layout: post
permalink: >
  http://www.iwasz.pl/electronics/motorcycle-black-box-part-1-data-acquisition-with-arduino-mega/
published: true
post_date: 2013-06-02 09:27:04
---
<strong>The objective</strong>

Make a black box, a device which would record short clips from camera in loop overwriting oldest clips. That would produce constant stream of short movies which put together would make one long recording containing last 30minutes (or more - depending on storage capacity) of my riding. Among with the recording, telemetric data would be collected such as velocity, RPM, temperature, front and rear brake pressed, depressed, and turn signals, and maybe more. Then this data could be put on the video in overlay.

<strong>The motorcycle</strong>

Yamaha XJ6SA 2010.

<strong>Identify by what means communication between the ECU and the dashboard is performed.</strong>

Which cables are used and what is the protocol used? After inspecting the service manual for my bike I was able to eliminate wires which are used for other things (see picture), and I left with only one, which is connecting the dash, the ECU and the immobilizer. Other ones was for flashing some warning LEDs, gathering information from fuel pump, oil switch and so on. So the yellow-blue wire was my first guess and it was correct. On the <a href="http://ecuhacking.activeboard.com/t38959479/yamaha-fi-diagnostics-tool/">ecuhacking forum</a>, which was source of very helpful information, guys was talking about something called K-line. I believe it is something widely common in the automotive industry, and for certain it has bunch of ISO standards describing it, but hey, if this is only one wire, and data flowing inside is some kind of serial communication, I bet I could sniff it, and figure out without any standards, which are hard to find and get (there is so much of them, I've got confused after few minutes of googling).

[caption id="attachment_218" align="alignnone" width="245"]<a href="http://www.iwasz.pl/wp-content/uploads/2013/06/wiring2.png"><img class="size-medium wp-image-218" alt="Dashboard connections" src="http://www.iwasz.pl/wp-content/uploads/2013/06/wiring2-245x300.png" width="245" height="300" /></a> Dashboard connections[/caption]

<strong>Figure out how to interface this, make some circuit if needed.</strong>

Again on the ecuhacing forum I've found information that K-line has logic levels relative to 12V, where 0V is logic 0, and 12V logic 1. There has to be some voltage level converter if I want to connect some TTL stuff to it. On the forum some guys were using a L9637 chip which is described as "ISO 9141 INTERFACE" by its datasheet. So I bought a few of these and connected it as follows :

[caption id="attachment_207" align="alignnone" width="300"]<a href="http://www.iwasz.pl/wp-content/uploads/2013/06/L9637.png"><img class=" wp-image-207" alt="L9637" src="http://www.iwasz.pl/wp-content/uploads/2013/06/L9637-300x156.png" width="300" height="156" /></a> L9637[/caption]

Later on I <strong>removed the 510 ohm pull up resistor</strong>, because after turning engine on there was error showing of on the dash. I guessed that could be this resistor, and it helped. Dunno what was wrong.

Most difficult part for me was to find a place in motorcycle's wiring to connect to. After some time I managed to insert piece of rigid wire to the back side of ECU connector as depicted below:

[caption id="attachment_209" align="alignnone" width="300"]<a href="http://www.iwasz.pl/wp-content/uploads/2013/06/IMG_20130529_234215.jpg"><img class="size-medium wp-image-209" alt="ECU connector" src="http://www.iwasz.pl/wp-content/uploads/2013/06/IMG_20130529_234215-300x225.jpg" width="300" height="225" /></a> ECU connector[/caption]

5V supply voltage I was drawing from step down voltage converter which I bought on the Internets. It is a PCB with a few discrete parts and quite huge radiator with some coils inside. There is [EDIT] label on it. To the output of L9637 I connected a Saleae logic analyzer, which I also bought. It is quite cheap and has good software for Windows, Linux and MAC OS. I can definitely recommend it, especially if you want to use it with Linux (I use Ubuntu).

<strong>Sniff the data, and collect it for further investigation.</strong>

My circuit worked the first time. After turning ignition to the ON state this is what I got:

[caption id="attachment_210" align="alignnone" width="300"]<a href="http://www.iwasz.pl/wp-content/uploads/2013/06/screenshot.png"><img class="size-medium wp-image-210" alt="Sealae logic analyzer window" src="http://www.iwasz.pl/wp-content/uploads/2013/06/screenshot-300x164.png" width="300" height="164" /></a> Sealae logic analyzer window[/caption]

Sealae app, among others, has a "Async serial" analyzer which I used with default configuration, and "use autobaud" checkbox checked. This useful option corrected initial 9600 baud to whatever it thought to be appropriate, and showed 16064 baud. Pretty odd value isn't it? None the less it is very useful information if I want to read the data with some AVR, or something like that. It turns out, that data comes in packets which are 6B long. First byte is some kind of command, and the rest is the reply i.e the dashboard issues command 0x01 ant then ECU replies with 5 bytes of data. From the ecuhacking I knew that forst byte of the reply would be the RPM, second probably velocity, third an error code, fourth would be engine temperature (that is coolant or oil temperature?) and the very last is the checksum.

[caption id="attachment_228" align="alignnone" width="300"]<a href="http://www.iwasz.pl/wp-content/uploads/2013/06/Screenshot-from-2013-05-30-023913.png"><img class="size-medium wp-image-228" alt="Sealae async serial analyzer setup" src="http://www.iwasz.pl/wp-content/uploads/2013/06/Screenshot-from-2013-05-30-023913-300x256.png" width="300" height="256" /></a> Sealae async serial analyzer setup[/caption]

<strong>Collect the data in some more useful format.</strong>

Although Sealae has an option to store the analyzer's data in CSV, sooner or later I would have to make some custom electronics anyway. Yet I want to make self contained logger hidden in neat case somewhere in the bike. I want to use a Raspberry PI i own as the main component of the gizmo, but after reading about serial communication with the Pi, I gave up for that time, and connected Arduino Mega. The main problem with PI is that I don't know how to set it up for so unusual baud rate as 16064 baud (or 15625 as someone suggested). I had limited time, so for now I chose the Arduino. Below the code I uploaded :
<pre lang="cpp">#include <SoftwareSerial.h>

SoftwareSerial mySerial(10, 11); // RX, TX

void setup() {
    // initialize both serial ports:
    Serial.begin(115200);

    // set the data rate for the SoftwareSerial port
    mySerial.begin(16064);
}

void loop() {
    static int count = 0;

    // read from port 1, send to port 0:
    if (mySerial.available()) {
        int inByte = mySerial.read();

        if (inByte == 0x01 && count >= 5) {
            Serial.println(' ');
            count = 0;
        }

        Serial.print(inByte, HEX);
        Serial.print (' ');
        ++count;
    }
}</pre>
As you can see, RX is set up for pin 10, so the only thing I did was to disconnect the logic analyzer, and connect Arduino pin 10 instead. But nothing happened. This was because the SoftSerial library have a few baud rates predefined, namely the most usual ones. The solution was to modify arduino-1.0.5/libraries/SoftwareSerial/SoftwareSerial.cpp so it looks like that:
<pre lang="cpp">// .... line 59
static const DELAY_TABLE PROGMEM table[] =
{
    // baud rxcenter rxintra rxstop tx
    { 115200, 1, 17, 17, 12, },
    { 57600, 10, 37, 37, 33, },
    { 38400, 25, 57, 57, 54, },
    { 31250, 31, 70, 70, 68, },
    { 28800, 34, 77, 77, 74, },
    { 19200, 54, 117, 117, 114, },
    { 16064, 66, 140, 140, 137, }, // added baud rate
    { 15625, 68, 144, 144, 141, }, // added baud rate 2
    { 14400, 74, 156, 156, 153, },
    { 9600, 114, 236, 236, 233, },
    { 4800, 233, 474, 474, 471, },
    { 2400, 471, 950, 950, 947, },
    { 1200, 947, 1902, 1902, 1899, },
    { 600, 1902, 3804, 3804, 3800, },
    { 300, 3804, 7617, 7617, 7614, },
};
// ....</pre>
<a href="http://www.tocpcs.com/newsoftserial-adding-a-baud-rate/">Here is the link</a> you should follow for more info. I haven't figured it out by myself. After this modification Arduino happily sent me my precious data, which I observed in serial monitor.

[caption id="attachment_211" align="alignnone" width="300"]<a href="http://www.iwasz.pl/wp-content/uploads/2013/06/serial-data.png"><img class="size-medium wp-image-211" alt="Serial data fas seen on arduino serial monitor" src="http://www.iwasz.pl/wp-content/uploads/2013/06/serial-data-300x168.png" width="300" height="168" /></a> Serial data fas seen on arduino serial monitor[/caption]

This is the data I collected so far (motorcycle standing on central stand, back wheel revolving, velocity comes from the back wheel, ABS LED blinking).

Ignition ON, engine stopped.
<pre>1 0 0 0 30 30
1 0 0 0 30 30
1 0 0 0 30 30
1 0 0 0 30 30
...</pre>
Ignition ON, engine started, no throttle (~1200 RPM), no gear (N) i.e. wheel not revolving, cold engine:
<pre>1 24 0 0 30 54
1 24 0 0 30 54
1 24 0 0 30 54
1 24 0 0 30 54
...</pre>
Ignition ON, engine started, little throttle (more RPM), no gear (N) i.e. wheel not revolving, cold engine:
<pre>1 34 0 0 53 87
1 34 0 0 53 87
1 34 0 0 53 87
1 34 0 0 53 87
...</pre>
1st gear, no throttle (engine warmed up thus less RPM). About 10 km/h
<pre>1 17 0 0 5C 73
1 17 1 0 5C 74
1 17 1 0 5C 74
1 17 0 0 5C 73
1 17 1 0 5C 74
1 18 1 0 60 79
1 18 1 0 60 79
1 18 0 0 60 78
1 18 1 0 60 79
1 17 1 0 60 78
1 17 0 0 60 77
1 17 1 0 60 78
1 17 1 0 60 78
1 17 0 0 60 77
1 17 1 0 60 78
1 17 1 0 60 78
1 17 0 0 60 77
1 18 1 0 60 79
1 18 1 0 60 79
1 18 1 0 60 79
1 18 0 0 60 78
1 18 1 0 60 79
1 18 1 0 60 79
1 18 0 0 60 78
1 18 1 0 60 79
1 18 1 0 60 79
1 17 0 0 60 77</pre>
23 km/h (6th gear)
<pre>1 17 2 0 68 81
1 17 2 0 69 82
1 17 2 0 69 82
1 17 2 0 69 82
1 17 1 0 69 81
1 17 2 0 68 81
1 17 2 0 68 81
1 17 2 0 68 81
1 17 2 0 68 81
1 17 1 0 68 80
1 17 2 0 69 82
1 17 2 0 68 81
1 17 2 0 68 81
1 17 1 0 68 80
1 17 2 0 68 81</pre>
40 km/h
<pre>1 2B 4 0 70 9F
1 2B 3 0 70 9E
1 2B 3 0 72 A0
1 2B 3 0 72 A0
1 2B 4 0 72 A1
1 2B 3 0 72 A0
1 2C 3 0 72 A1
1 2C 4 0 72 A2
1 2C 3 0 72 A1
1 2C 3 0 72 A1
1 2C 4 0 72 A2
1 2C 3 0 72 A1
1 2C 3 0 72 A1
1 2C 4 0 72 A2
1 2C 3 0 72 A1
1 2C 4 0 72 A2
1 2C 3 0 72 A1
1 2C 3 0 72 A1</pre>
60 km/h
<pre>1 40 5 0 74 B9
1 40 5 0 74 B9
1 40 5 0 74 B9
1 40 5 0 74 B9
1 40 5 0 74 B9
1 40 5 0 74 B9
1 40 5 0 74 B9
1 40 5 0 74 B9
1 41 4 0 74 B9
1 41 6 0 74 BB
1 41 4 0 74 B9
1 40 5 0 74 B9
1 40 5 0 74 B9
1 41 5 0 74 BA
1 40 5 0 74 B9
1 40 5 0 74 B9</pre>
80 km/h
<pre>1 56 7 0 77 D4
1 55 6 0 77 D2
1 56 7 0 77 D4
1 55 6 0 77 D2
1 56 7 0 77 D4
1 56 6 0 77 D3
1 56 7 0 77 D4
1 56 6 0 77 D3
1 56 7 0 77 D4
1 56 6 0 77 D3
1 56 7 0 77 D4
1 56 7 0 77 D4
1 56 6 0 77 D3
1 56 7 0 77 D4
1 56 7 0 77 D4
1 56 6 0 77 D3</pre>
<strong>Future</strong>

Due to uncertainty Raspberry PI serial communication I plan to make custom PCB with AVR in form of shield to Raspberry Pi. It will have two purposes. First it will acquire the data in some similar manner as depicted above, and will pass it to Pi by I2C or SPI (probably with velocity in some more usable form), among with some other data such as brakes and turn-lights, an maybe even outdoor temperature (I always wanted to have this information, especially in spring and autumn). And secondly it will drive some relay to cut the power to Pi which draws 2W of power even when shut down. Of course it also should send some signal first to Raspberry to shut it down correctly. If someone know something more about Pi's UART and custom baud rates in particular, please let me know. Maybe there is a way to read K-line directly without AVR.

&nbsp;