# Smartctl

The tool is used to test/monitor storage disks (HDD, SSD)

## Display device information

`sudo smartctl -i /dev/sda`

##  Display all the SMART information (errors, ...)

`sudo smartctl -a /dev/sda`

## Check health

`sudo smartctl -H /dev/sda`