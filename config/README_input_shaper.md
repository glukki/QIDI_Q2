# Steps

1. Update `printer.cfg`, save and reboot
2. Connect to `ssh mks@<printer ip address>`, use `makerbase` password
3. Run `pip install numpy matplotlib`
4. From Web UI run printer commands:
```
# home
G28

# collect X resonance in different points of Y axis
TEST_RESONANCES AXIS=X FREQ_START=20 FREQ_END=100 OUTPUT=raw_data POINT=135,135,10
TEST_RESONANCES AXIS=X FREQ_START=20 FREQ_END=100 OUTPUT=raw_data POINT=135,70,10
TEST_RESONANCES AXIS=X FREQ_START=20 FREQ_END=100 OUTPUT=raw_data POINT=135,5,10

# collect Y resonance
TEST_RESONANCES AXIS=Y FREQ_START=20 FREQ_END=100 OUTPUT=raw_data POINT=135,135,10
```
5. Connect to `ssh mks@<printer ip address>`, use `makerbase` password, run commands:
```sh
mkdir ~/printer_data/config/shapers
~/klipper/scripts/calibrate_shaper.py /tmp/raw_data_x*.csv -o ~/printer_data/config/shapers/x.png
~/klipper/scripts/calibrate_shaper.py /tmp/raw_data_y*.csv -o ~/printer_data/config/shapers/y.png
```
6. From Web UI open graphs from the `shapers` directory
7. Put recommended settings in the end of `printer.cfg` like so:
```
#*# [input_shaper]
#*# shaper_type_x = zv
#*# shaper_freq_x = 50.1
#*# shaper_type_y = zv
#*# shaper_freq_y = 51.3
```
8. Save and reboot
