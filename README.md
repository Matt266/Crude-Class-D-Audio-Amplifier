# Crude-Class-D-Audio-Amplifier

## General
  The purpose of the Project was to build a functional circuit without microcontroller and advanced ICs in order
  to pracitce simulation and design of circuits.
  As an old Audio-Amplifier stopped working I decided to try to make a new Amplifier for the Subwoofer it powered.
  
  I had not yet started studying electrical engineering at an unviersity when I did this project - it is 100 percent
  based on web researching! With simulating the circuit in LTSpice it was possible for me to build a working concept though.
  So don't expect perfect descisions - I researched for the most optimal design I could find and realisticaly implement.
  
  I restrcited myself further by only allowing components I had laying around at home. This means the component choice 
  is suboptimal and could be improved by using more suitable components (e.g. MOSFETs with lower on resistance).
  So for simulating the circuit I downloaded the corresponding models or used components with similiar charackteristics.
  
  ![crude class-d amp](/images/class_d_amp_pcb.jpg)
  
## Choosing an amplifier topology
  For the basic ampflifier conecept there are a few topoligies to choose from.
  * Class-A which would have been a single-resistor amplifier in my case
  * Class-B which basically would split the audio signal into a positive and negative phase and amplify those with different amplifiers
  * Class-C same as Class-B but splitting the signal into more than two "sections"
  * Class-D convert the unamplified analog audio to a digital signal which can be amplfified a lot easier and filtering it back to an analog signal

  I found the Class-D amp to be the most suitable design. First of all it is the most energy efficient topology. There is no "standby current"
  as in a Class-A design where the amplifiying transistor is partly turned on (this is because DC-biasing is needed for a single transistor to allow 
  it to amplifiy the complete fullwave in the linear region and without clipping). Also the amplification of the digital Signals itself is much more 
  efficient than an analog amplification. And at last I don't have to fine tune DC-biasing and component values in order to fight crossover distortion which 
  is a present problem in Class-B and C amplifiers. Calculating a good Class A, B or C design would have been beyond my possiblities at this time and 
  as just copying values from internet resources would have violated the restrictions I gave myself. The Class-D design was a clear topoligy choice.
  
## Concept
  The basic concept of the Class-D design I chose is rather simple:
  1. The analog audio is filtered and converted to a pwm signal with changing duty cycle (this might sound like a complex conversion but really is easy to do)
  2. then the digital pwm signal is amplified through the output stage
  3. the high frequency pwm signal still contains the low frequency information which therefore can be extracted by lowpassing the output signal

## Design choices
  Now that I had an Idea of the functional blocks that I had to implement I started to research for the best ways to implement those blocks.
  
### Filters
  First things first I omitted the filters. The input filter would be needed to protect for any noise - as my input signal is rather clear and I epexted my design
  to introduce a lot of distortion due to the unsuited components available and my lack in experience and knowledge it wouldn't improve the quality and be useless
  anyway. The ouput filter will filter out the high frequency content of the pwm signal - as this frequency is way to high to be heard with 200kHz (I hadn't 
  chosen a fixed  frequency yet but knew it would be at least that high) I omitted this filter too. There are other reasons for applying the lowpass like avoid
  possible damage to the speakers, EMI, etc. which I also ignored for this project.
 
### Analog-Digital Converter
  To convert the input signal into a pwm signal a comparator (which is one of three integrated circuits I ended up using) compares it with a triangle or sawtooth wave.
  The higher a point of the audio signal is in amplitude/voltage the longer the output of the comparator stays high as a higher percentage of the triangle or sawtooth wave
  is below that value. There are two comparators fed by the audio and triangle signal with the inverting and noninverting input switched. This generates the required
  opposite switching signal for the H-Brigde I use as the main amplifier - this part of the circuit will be discussed in a moment.
  ![crude class-d amp](/images/ADC_Input_stage.jpg)
  
  This is a 2KHz sinewave I used as test signal for simulation with the corresponding triangle wave:
  ![crude class-d amp](/images/ADC_Input_stage_waveform_input.jpg)
  
  And this is the resulting output of one of the comparators:
  ![crude class-d amp](/images/ADC_Input_stage_waveform_output.jpg)
  
  Beside the clearly visible distortion that is introduced by the circuit during conversion one can also see the remaining sine wave in the output signal.
  This is the triangle oscillator:
  ![crude class-d amp](/images/tri_oscillator.jpg)
  
  I decided to go for a triangle wave as it was easier for me to achieve a linear tri wave than a sawtooth (mainly because of the one very steep edge).
  A relaxation oscillator produces a square wave which serves as input signal for two opposite current mirros. One mirror charges a capacitor during one half
  of the square singal while the other discharges it  on the other. If the relaxation oscillator would directly charge the capacitor a pseudo tri wave could be
  seen. The capacitor would charge and discharge exponentially as the rising/sinking voltage across it would slow the process. By using current mirrors the voltage
  on the square oscillator side remains constant during the charging process and with this also the input current remains the same. Now the current mirrors force
  this constant current in/out of the capacitor thus charging it linear - which even though i didn't match any of the transistors or resistors did work surprisingly
  well in the real circuit.
  
  This image shows the square wave and the resulting triangle wave:
  ![crude class-d amp](/images/tri_oscillator_waveform.jpg)
  
  So why did I use the NE555 while an relaxation oscillator could be built discrete? I honestly wasn't able to calculate and thus design the schmitt trigger 
  for replacing the 555 timer with the knowledge I had gained so far. Same goes for the voltage divider I used for DC-offseting the trianglewave after decoupling
  it as input for the comparator. Using two resistors with the same value to offset it to half of the supply voltage dind't work - I don't know why. I just tweaked
  the values until I found a sweet spot.
  
### Amplification stage
  
  For amplification I had the choice between a half-bridge or H-bridge topology. A half bridge consists mainly of two transistors - one which connects the load to
  the upper supply voltage - and one that connects it to the lower voltage level. By alternately switching one of the transistors off and the other on the pwm / 
  switching signal would be translated to the load. As only a non symmetric power supply with 12V was available for me this would have halfed the voltage amplitude
  of the resulting signal (GND or 0V for the maximum negativ voltage and 12V for the maximum positiv voltage which results in a DC-offset of 6V volt and a 6V amplitude).
  
  So I chose the H-Bridge which is only two half-bridges conncected together. While one side is driven to Vcc the other side of the load is driven to the lower voltage -
  in this case GND. With a change in the controlling square signal the side prior pushed to 12V now is connected to GND and vice verser thus alternating the direction
  of current flowing through the load. This means the load sees a +12V and a -12V connection - the amplitude just got doubled.
  
  Switching MOSFETs at high speeds is a tough job as gate capacitance requires a higher current flow in order to turn the Transistor on and off fast enough. Normally for
  such a purpose Driver ICs are available. But since 200KHz are relatively slow and didn't want to use any "unnecessary" IC I decided to to build the driving circuit myself.
  
  In the end this is the schematic I came up with:
  ![crude class-d amp](/images/output_stage.jpg)
  
  The two comparatores mentioned in the ADC description already give me to signals inverted to each other so that I already got my signal for driving both bridge sides.
  As the current output of those comparators is way to small to sufficiently switch the MOSFETs I amplfied this signal with two BJT amplification stages. 
  The first one (q5 and q6 on the complete schematic) are simple common emitter amplfifiers and as such don't need further mention and are not visible on the part of the
  schematic above.
  The second stage where I used BD139 transistors to be able to handle higher currents is also a common emitter amplifier but with some tricks to be mentionend:
  1. the diode present between base and collector of each transistor is a so called "Baker Clamp". If too much current from the first amplification stage would flow
     into the base of the BJT it would saturate. This would significantly slow down transition time between on and off states of the BJT. The diode simply enables
     this portion of current to flow through an alternate path (in detail there is negative feedback going on but for the scope of this decription it is sufficient
     to say the BJT just won't saturate as much as before and thus switch faster).
  2. The gate of the MOSFEt driven is pulled-up through a higher value resistor than pulled-down. This ensures that the lower MOSFET is always turned off when the
     upper MOSFET on the same side is turned on. This would result in large current spikes as for a short period of time a short circuit would be present. While 
     simulating everything looked fine with 220 ohm resistors as pull-ups but in reality the MOSFETs heated so much that I had to change those to 1 kilo ohm resistors.
     A Driver-IC would implement a dead time function for this where for a short time both transistors are explicitely turned off. As I coudln't implement a behaviour
     like this on the digital side I had to settle with the solution explained before.
  3. As the high side MOSFETs are placed before the load their emitter voltage will rise to 12V while they are turned on. A 12V signal isn't sufficient to turn these
     on. So I utilized a bootstrapping circuit. A capcitor is on one side connected to the emmiter of the MOSFET while on the other side being charged through a diode.
     If the MOSFET is turned off the capacitor is charged to 12V. If the MOSFET now turns on the potential connected to GND before quickly rises to 12V. This rise
     will be conducted by the capacitor so that the other side - before 12V - now becomes 24V. The diode hinders current to flow into the power supply and the gate 
     of the high side MOSFET can now be switched. This however works only with switching applications as the bootstrap capacitor is discharged in the process and
     leakage currents will ultimately turn off the high side after a period of time. Therefore the bootstrap capacitor needs to be recharged by switching the state of
     the H-brigde.
     
   The last thing to mention is the flyback diode connected to the MOSFETS. They serve as current path if all transistors are closed and allow inductive spikes to dissipate
   to heat. Otherwise the MOSFETs could get killed by voltage spikes happening because of load inductance trying to maintain a current flow.
   
   The gate voltage looks like this:
   ![crude class-d amp](/images/output_stage_gate.jpg)
   
   The resulting waveform like this (current over time as SPICE didn't allow me to FFT voltage diffence graphs): 
    ![crude class-d amp](/images/output_waveform.jpg)
    
   The 2KHz sine wave is clearly visible as peak after FFt even though a lot of noise is present: 
    ![crude class-d amp](/images/output_fft.jpg)
    
   This is the complete schematic:
    ![crude class-d amp](/images/class_d_amp.jpg)
   
 # Conclusion
   After finally adding a LDO-Regulator (the third IC used) for the 5V regions of the circuit, soldering the circuit, troubleshooting a lot of mistakes I made during this          procedure and changing the pull-ups to the value of 1 kilo ohm the amplifier finally **worked**!! Sadly the amplification is too low and the noise / distortion 
   too high to really use this amplifier (a Class-A pre amp would also help the design) but I'm happy I could bring this circuit to life. 
    
