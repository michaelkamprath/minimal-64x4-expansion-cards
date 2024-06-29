# Expansion Cards for Minimal 64x4 Computer
This repository contains various expansion cards for the [Minimal 64x4 Home Computer](https://github.com/slu4coder/Minimal-64x4-Home-Computer/) design by Carsten Herting, known as [slu4 on YouTube](https://www.youtube.com/channel/UCXYQcMpUBT3aaQKfmAVJNow).


## Back Plane
The [back plane provided in this repository](./expansion-backplane/) allows for more than one expansion card to be connected to the Minimal 64x4. It also provides a more convenient barrel jack power connector with a on/off switch to power the Minimal 64x4.

### Shared `INH` Line
To have multiple cards connected to the backplane, they need to share the active high `INH` inhibit line. If multiple cards are emitting a LOW signal directly, and one emits a HIGH, contention will occur. To prevent such contention, each card should use some sort of barrier between its `INH` signal source and the `INH` line on the back plane. 

The technique generally used in these cards is to use a BS250 MOSFET to act as a high-side switch to the `INH` line. This requires the card to actually drive an active low `/INH` signal to the BS250's gate. 

## Cards

* [**Speech Synthesizer**](./speech-synthesizer/) - This board provides support for interfacing with a TMS5220C speech synthesizer. 


