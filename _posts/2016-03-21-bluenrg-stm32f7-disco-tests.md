---
ID: 458
post_title: BlueNRG + STM32F7-DISCO tests
author: admin
post_excerpt: ""
layout: post
permalink: >
  http://www.iwasz.pl/uncategorized/bluenrg-stm32f7-disco-tests/
published: true
post_date: 2016-03-21 01:18:39
---
<a href="http://www.iwasz.pl/wp-content/uploads/2016/03/IMG_20160321_020856.jpg" rel="attachment wp-att-462"><img class="size-medium wp-image-462 alignleft" src="http://www.iwasz.pl/wp-content/uploads/2016/03/IMG_20160321_020856-300x225.jpg" alt="IMG_20160321_020856" width="300" height="225" /></a>
<ul>
	<li>Tested 32bit cross-compiler from launchpad, it worked OK, but binaries were much bigger, and debugger lacked Python support which is required by QtCreator which is an IDE of my choice (for all C/C++ development including embedded).</li>
	<li>Made my own compiler with crosstool-NG (master from GIT). Used my own tutorial, which can be found <a href="http://www.iwasz.pl/electronics/toolchain-for-cortex-m4/">somewhere over this blog</a>.</li>
	<li>Verified that a "hello world" / "blinky" program works. It did not, because I copied code for STM32F4, and I messed the clock configuration. Examples from STM32F7-cube helped.</li>
	<li>I <a href="https://github.com/iwasz/blue-nrg-test">ported my project</a> which aimed to bring <a href="http://www.st.com/web/catalog/tools/FM116/SC1075/PF260517">my BlueNRG dev board</a> to life. The project consists of a USB vendor-specific class for debugging (somewhat more advanced, and tuned for the purpose than simple CDC class), slightly adapted BlueNRG <a href="http://www.st.com/web/en/catalog/tools/PF261442">example code from ST (for Nucleo)</a>, and some glue code.</li>
	<li>I switched to STM32F7-disco because I couldn't get BlueNRG to work with STM32F4-disco. F7 has Arduino connector on the back, so I thought it'd be better than bunch of loosely connected wires which was required for F4. At that time I thought that there is something wrong with my wiring.</li>
	<li>On STM32F7-disco BLE still would not work. I hooked up a logic analyzer, and noticed that MISO and CLK is silent. It turned up that I have to use alternative CLK configuration on BlueNRG board which required to move 0R resistor from R10 to R11. Although it was small (0402), at first I had problem desoldering it, probably because of unleaded solder used. See image.</li>
</ul>
<p style="direction: ltr;"><a href="http://www.iwasz.pl/wp-content/uploads/2016/03/IMG_20160320_175826.jpg" rel="attachment wp-att-460"><img class="size-medium wp-image-460 alignright" src="http://www.iwasz.pl/wp-content/uploads/2016/03/IMG_20160320_175826-300x182.jpg" alt="IMG_20160320_175826" width="300" height="182" /></a></p>

<ul>
	<li style="direction: ltr;">Next problem I had, was with SPI_CS pin. According to the BlueNRG-dev board docs, I wanted to use D8 pin (this is PI2 pin on STM32F7-disco), but it didn't work. Glimpse on the schematics revealed that the SPI_CS is connected to A1, which is marked as "alternative" in the docs. So IMHO this is an error in the docs.</li>
	<li style="direction: ltr;">Final pin configuration that worked was:</li>
</ul>
<pre>// SPI Reset Pin
#define BNRG_SPI_RESET_PIN GPIO_PIN_3
#define BNRG_SPI_RESET_PORT GPIOI
// SCLK
#define BNRG_SPI_SCLK_PIN GPIO_PIN_1
#define BNRG_SPI_SCLK_PORT GPIOI
// MISO (Master Input Slave Output)
#define BNRG_SPI_MISO_PIN GPIO_PIN_14
#define BNRG_SPI_MISO_PORT GPIOB
// MOSI (Master Output Slave Input)
#define BNRG_SPI_MOSI_PIN GPIO_PIN_15
// NSS/CSN/CS
#define BNRG_SPI_CS_PIN GPIO_PIN_10
#define BNRG_SPI_CS_PORT GPIOF
// IRQ
#define BNRG_SPI_IRQ_PIN GPIO_PIN_0
// !!!
#define BNRG_SPI_IRQ_PULL GPIO_PULLDOWN
#define BNRG_SPI_IRQ_PORT GPIOA</pre>
<ul>
	<li>Next thing that got me puzzled for quite a time was that after initial version query command, which seemed to work OK (I got some reasonably looking responses) the communication hanged. IRQ pin went high, and wouldn't go low. So µC thought that there is data to read and was continuously reading and reading.</li>
	<li>Only after I read in <a href="http://www.st.com/st-web-ui/static/active/en/resource/technical/document/user_manual/DM00114498.pdf">UM1755 User manual</a> that IRQ goes HI when there is data from NRG to µC, and it goes HI-Z when there is no data, I double checked the schematics, and found that BlueNRG-dev board has no pull-down resistor in it. The ST example code I had also hadn't have IRQ pin configured in pull-down state, so I wonder how this could work. Maybe Nucleo boards have pull-down resistor soldered there.</li>
	<li>For now (after 3 or 4 nights, and I consider myself as quite experienced) I got this (see photo):<a href="http://www.iwasz.pl/wp-content/uploads/2016/03/Screenshot-from-2016-03-21-02-09-30.png" rel="attachment wp-att-463"><img class="alignnone size-medium wp-image-463" src="http://www.iwasz.pl/wp-content/uploads/2016/03/Screenshot-from-2016-03-21-02-09-30-300x217.png" alt="Screenshot from 2016-03-21 02-09-30" width="300" height="217" /></a></li>
	<li>"Iwona" is probably my neighbor's phone.</li>
	<li>Quick update made the next day morning. My Linux computer is able to connect to the test project, and query it (right now it simply counts up):</li>
</ul>
<a href="http://www.iwasz.pl/wp-content/uploads/2016/03/Screenshot-from-2016-03-21-13-05-25.png" rel="attachment wp-att-466"><img class="size-medium wp-image-466 aligncenter" src="http://www.iwasz.pl/wp-content/uploads/2016/03/Screenshot-from-2016-03-21-13-05-25-300x179.png" alt="Screenshot from 2016-03-21 13-05-25" width="300" height="179" /></a>