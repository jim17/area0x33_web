+++
title = "Attiny as programmable clock source"
subtitle = "using Attiny85"
bigimg = "/blog/img/attiny8mhzclock.png"
date = "2014-05-15"
+++

In a project we were working, we needed a clock source for a cpld board. After watching some IC clock sources, we thought that attiny9 could do the job. The price of an attiny9 is about 1$ and the price of an IC clock source is about 3-4$, so… you may be with me saying that an attiny can be worth it as clock source, right ?

The Attiny9 has an internal RC oscillator that you can speed up to 8MHz. Doing some tests, the maximum speed we got was 2MHz because we were wasting 1 cycle setting the output pin HIGH and 1 cycle setting the output pin LOW plus 2 cycles in the while(1) loop.

{{< highlight c >}}
while(1)
{
    PORTB |= _BV(PB2);  // Set pin high (1 cycle)
    PORTB &= ~_BV(PB2); // Set pin low  (1 cycle) + while(1) loop (2 cycles)
}
{{</ highlight >}}

With this code we got a 8MHz/4 = 2MHz waveform with 25% HIGH and 75% LOW. Hmmmm… we wanted to achieve 8MHz and we were only having a non symetrical 2MHz waveform. How to solve this F…ING problem? Yes, I got the answer.

Luckily for us, after reading the datasheet of attiny9, we discovered that we don’t needed to set the PINB2 HIGH and LOW to generate the clock because with a FUSE setting we could send the internal RC clock of the microcontroller directly to PINB2, having a symmetrical 50% HIGH 50% LOW waveform at the maximum speed of the microcontroller, 8MHz. The Fuse settings configuration was 0xFB  meaning CLKOUT bit 2 of Configuration Byte 0 programmed/enabled.
<center>
<img src="/blog/img/attiny8mhzclock.png" alt="8Mhz clock" align="middle">
</center>
As you can see on the picture, we were achieving 7,7 MHz. In order to get 8MHz output clock we should calibrate the internal RC oscillator by changing the value of a register, but this is a different story.

The final code look like this:
{{< highlight c >}}
include <avr/io.h>

int main(void)
{
    // Ensure PB2 is configured as output (really not necessary)
    DDRB |= _BV(PB2);
    // write signature to CCP allow us to write CLKPSR register and change
    // the internal RC clock speed to 8Mhz
    CCP = 0xD8;

    // clock wtih no preescaler
    CLKPSR = 0x00;
    
    while(1);
}
{{</ highlight >}}


As you can see, the clock source can be programmed by changing the preescaler of the internal RC clock (CLKPSR register of attiny).
