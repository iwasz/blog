---
ID: 345
post_title: Internet thermal printer
author: admin
post_excerpt: ""
layout: post
permalink: >
  http://www.iwasz.pl/electronics/internet-thermal-printer/
published: true
post_date: 2014-07-11 12:31:10
---
<del>The idea is shamelessly <strong>stolen</strong> from <a title="Pigeon post on hackaday.io" href="https://hackaday.io/project/1085-Pigeon-Post">this hackaday.io project</a>.</del> <strong>EDIT</strong> : It evolved... What this project is intended to be:
<ul>
	<li>A toy printer (for my son) with some light and sound signal connected to the Internet and accessible by some web interface. Anyone with password (basic auth configured in .htaccess) could send a graphic and/or text message to the printer which would immediately flash the light, beep the buzzer as well as print the message. Protocol to the printer (on the network level) : whatever a.k.a my own &amp; super-simple.</li>
</ul>
<del>What this project shall not become</del> <strong>EDIT</strong> : It evolved.. (note to myself because I tend to complicate things):
<ul>
	<li>An over-engineered wireless CUPS compatible Postscript full featured printer which also makes coffee.</li>
</ul>
After deciding that I would try to make such a thing which took approx. 1 second after seeing Jim's site I went to <a href="http://allegro.pl">allegro.pl</a> (local EBay. BTW we have ebay.com.pl here in Poland, but allegro seems to be winning the battle) and found something printer-ish alike and seemingly broken, with some parts missing. It is a <strong>Intermec PW40</strong> mobile printer. Useful links I found on this printer:
<ul>
	<li><a href="http://www.intermec.com/support/manuals/search.aspx?categoryid=7&amp;familyid=11&amp;productnodeid=PRTRPW50A">Manuals - Intermec site</a> (those are for PW50, but I assume they are compatible in some way).</li>
	<li><a href="http://community.intermec.com/">Intermec</a> community - they even have forum, and some community around the site.</li>
</ul>
Photos after dismantling the thing:

[gallery ids="378,375,376,377,379,380,381,382,383,384"]

Looks like it <a title="ESC/P on Wikipedia - an old printer language." href="http://en.wikipedia.org/wiki/ESC/P">uses ESC/P</a> like Jim's printer and 7.2V battery pack also. Looks promising (at least some standard language). Elements found on the main board of PW-40:
<ul>
	<li><a href="http://pl.mouser.com/ProductDetail/Toshiba/TMP95C061BFZ/?qs=f0GatGUIxV3RUG14ubT6RQ==">Toshiba TMP95C061BF</a> : some 16 bit microcontroller. <a href="http://en.wikipedia.org/wiki/Toshiba_TLCS">TLCS-900</a> family. No free C compiler available according to Wikipedia.</li>
	<li>NEC <a href="http://search.datasheetcatalog.net/key/D431000">D431000AGW</a> : SRAM 1Mb.</li>
	<li><a href="http://pdf.datasheetcatalog.com/datasheet/SGSThomsonMicroelectronics/mXyxzsz.pdf">ST M29F040B</a> : Flash 4Mb.</li>
	<li>LinearTechnology <a href="http://www.51samples.com/upfiles/Manual/1384fa%5B1%5D.pdf">LTC1384CG</a> : this is what I was looking for : a RS232 transceiver.</li>
	<li><a href="http://www.cirrus.com/en/pubs/proDatasheet/CS8130_F1.pdf">CS8130</a> : IR transceiver.</li>
</ul>
I've written the LTC chip looks promising, because it connects the printer to the outside world, and gives a hint where to start hacking. It translates RS232 high voltage levels to TTL, but since I wanted to drive the printer directly from some µC I needed to bypass the LTC. After some research I determined what follows : RS 232 port (this with RJ socket) is connected to pins 14 (232 input) and 15 (232 output). Corresponding TTL pins are : pin 13 (logic output), and 12 (logic input). So as far as I am reasoning correctly :
<ul>
	<li>Pin 13 is connected to the Toshiba's RX pin.</li>
	<li>Pin 12 is connected to the Toshiba's TX pin.</li>
	<li>Whole device can be powered from 12V supply (I read that somewhere).</li>
	<li>Let's try it! Seems to work. At least PC and the printer are communicating. Wiring looks like this:</li>
</ul>
[gallery ids="386,385"]
Costs so far:
<ul>
	<li>Printer : 25PLN ($8).</li>
	<li>10 rolls of thermal paper 20PLN ($7)</li>
</ul>
<a href="http://www.intermec.com/public-files/technology-briefs/en/CUPS-Printing-In-Linux-UNIX-For-Intermec-Printers-Tech-Brief.pdf">Intermec provides a CUPS driver</a> for Linux which enables you to use their printer as regular printer in the OS. Apparently PW40 isn't supported. I successfully compiled and installed the software, but printing a random text file gave me some gibberish. After that I tried to communicate with te printer in ESC/P language directly, but with no luck. I <a href="http://community.intermec.com/t5/General-Intermec-Device/PW40-is-it-broken-or-not/m-p/27118">described my problems on the Intermec</a> forums and still waiting for some reply. In short the problem is, that I don't really know for sure if this is me doing something wrong, or the printer is broken (it was sold on auction as broken, but seller couldn't tell for sure if it is really broken or not). So after two evenings the situation looks that I am able to print only one character in a row. If I'm sending more than 1 character to print, it hangs. To make matters worse, my printer won't print a self test page as it is described in the manual. It feeds paper a little and that's all. At the other hand I <a href="http://zival.ru/sites/default/files/download/ltp3445.pdf">found a datasheet</a> of the printer head used in my printer, but using it directly would be a triumph of form over the content I'm afraid, and I don't have enough time for that (i.e. making my own printer from scratch). But I'm overambitious you know, so who knows...

[caption id="attachment_387" align="alignnone" width="300"]<a href="http://www.iwasz.pl/wp-content/uploads/2014/07/IMG_20140620_123905.jpg"><img class="size-medium wp-image-387" src="http://www.iwasz.pl/wp-content/uploads/2014/07/IMG_20140620_123905-300x225.jpg" alt="This is the only thing It can print. If I try to print more than 1 character in a row, It hangs." width="300" height="225" /></a> This is the only thing It can print. If I try to print more than 1 character in a row, It hangs.[/caption]