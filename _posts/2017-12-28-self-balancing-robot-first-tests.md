---
ID: 492
post_title: Self balancing robot first tests
author: admin
post_excerpt: ""
layout: post
permalink: >
  http://www.iwasz.pl/uncategorized/self-balancing-robot-first-tests/
published: true
post_date: 2017-12-28 13:28:27
---
[embedyt] http://www.youtube.com/watch?v=VrV6Haa0kxQ[/embedyt]

<a href="http://www.iwasz.pl/wp-content/uploads/2017/12/22424639_1863508117010274_5025855902577973208_o.jpg"><img class="size-medium wp-image-499 alignright" src="http://www.iwasz.pl/wp-content/uploads/2017/12/22424639_1863508117010274_5025855902577973208_o-169x300.jpg" alt="" width="169" height="300" /></a>I have built a self balancing robot, and here I want to post some notes regarding problems that I encountered. It was (as usual) more difficult than I anticipated, but I guess every hacker know this feeling.
<h3>Frame</h3>
Frame is made of 8mm aluminium pipes, 2 sheets of 2mm solid transparent polycarbonate, and aluminium angle bars. The pieces are screwed together with threaded rods and self locking nuts.  Making the frame was the easiest part, so I won't write much about it.
<h3>Motors</h3>
<a href="http://www.iwasz.pl/wp-content/uploads/2017/12/86_P_1388386610626.jpg"><img class="size-medium wp-image-494 alignleft" src="http://www.iwasz.pl/wp-content/uploads/2017/12/86_P_1388386610626-300x219.jpg" alt="" width="300" height="219" /></a>Motors are labeled "Dagu DG02S 48:1", and I am disappointed, because they have very low torque at low RPM, and are of low quality I think (their low price also suggests that). The consequences are, that they are unable to move my robot gently and precisely, but instead they start all of a sudden with couple of RPMs. At very low PWM duty cycle they are unable to move wheels at all, even with no load. The lowest RPM i can get out of them is probably a couple of RPMs, which sounds OK, but I think, that many of my problems arise from them being unable to gently move the wheels few millimeters forth and back. <strong>Such gentle movement is in my opinion crucial</strong> for steady (nearly stationary) operation, that you have on certain YouTube videos. So next tests (if there will be any) will be performed with steppers or geared brushless motors. Oh, and there are no encoders neither.
<h3>Electronics</h3>
If motors are the brawl, electronics are the brain. Main board is <a href="http://www.st.com/en/evaluation-tools/stm32f4discovery.html">STM32F4-DISCO</a> and is connected to the battery via custom per-boards with connectors. On the top of the robot there is a single <a href="https://www.ebay.com/sch/i.html?_from=R40&amp;_trksid=p2322090.m570.l1313.TR0.TRC0.H0.TRS0&amp;_nkw=samsung+icr18650-26f&amp;_sacat=0">16850 Samsung</a> cell paired with cheap LiPo charger, and switch. I chose it, because it is a lot cheaper per mAh than those silverish ones. Battery sits inside bracket ordered on AliExpress. As of accelerometer and gyroscope, the super popular MPU 6050 does the job (also AliExpress). Motors are driven by <a href="https://www.pololu.com/product/2135">Pololu DRV8835</a>, and last but not least nRF24L01+ is used for connectivity with the robot, which is crucial to tune PID controller without dangling cables which would degrade stability, and make center of mass unstable. Like shooting to a moving target.

Oh, and I wanted to control the robot using cheap CX-10 remote, but It failed completely. I based my code on deviation and others work, but after many hours I gave up. Then I've got Syma X5HW for Christmas, and with its remote I finally had success (after first try, like 10 minutes of coding, not everyday something like this happens. But of course I only had to modify parameters for my CX-10 code). At first I confused the binding channel number (I set 0 instead of 8), and I was still able to receive data, but only from approx 5 cm apart. Then after setting it correctly to channel 8 range increased dramatically.
<h3>Programming</h3>
<ul>
 	<li><a href="https://github.com/iwasz/self-balancer">https://github.com/iwasz/self-balancer</a></li>
 	<li><a href="https://github.com/iwasz/libmicro">https://github.com/iwasz/libmicro</a></li>
</ul>
With electronics and motors on its place, there comes programming, the most difficult part. First I made peripherals of the robot to work (Arduino library called i2cdevlib was very helpful), so I was able to read raw data from MPU 6050, send basic commands via nRF, and spin the motors. Then, the most challenging part was to implement:
<ul>
 	<li>pitch angle calculation (<a href="http://www.starlino.com/imu_guide.html">http://www.starlino.com/imu_guide.html</a>),</li>
 	<li>fusion of gyroscope and accelerometer data (this was helpful : <a href="http://www.pieter-jan.com/node/11">http://www.pieter-jan.com/node/11</a>),</li>
 	<li>PID tuning (though implementing a PID is dead simple, tuning it is completely other story). Most helpful for me in this regard was probably the Wikipedia article on PID controller. In my case, the most (positive) impact came from integral part, where D part seems to have minimal influence.</li>
</ul>
I'll try to force myself to get into detail on the topics above, as I only scratched the surface.