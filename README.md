# Domoticz on Pi3 with SDM120C Modbus
This file describes how to install SDM120c Modbus in Domoticz on Raspberry Pi 3

You need a Raspberry Pi with Domoticz and a Modbus to USB converter

Then follow the following steps:

## Step 1: Install libmodbus and SDM120C software <br />
Make a telnet connection to the Pi <br />
```
sudo apt-get install libmodbus5 libmodbus-dev git
git clone https://github.com/gianfrdp/SDM120C.git
cd SDM120C
make clean && make
sudo make install
cd ..
```

## Step 2: Add hardware and devices in Domoticz <br />
Go to the hardware page and select by type Dummy (Does nothing, use for virtual switches only) <br />
And name it SDM120C, if you like <br />
Create now a virtual sensor with the name of the first SDM120C meter with type Electric(Instant+counter) <br />
Do this for every SDM120C meter you connect <br />
Remember the ID of every SDM120C you add <br />
 <br />
## Step 3: Script for reading and putting it in Domoticz <br />
Copy the following text: <br />
```
#!/bin/bash
#title           :SDM120C Modbus
#description     :Read current from sdm120 and sends it to Domoticz.
#author          :RP
#date            :20170920
#version         :0.1
#usage           :
#notes           :
#bash_version    :
#==============================================================================
SERVER="http://127.0.0.1:8084"   #server location
IDXTGroep3=10	#id of your device in Domoticz for total + power
IDGroep3=1	#id of device (modbus id)

IDXTGroep4=11
IDGroep4=2

IDXTGroep5=12
IDGroep5=3

IDXTGroep11=13
IDGroep11=4

VALUES=NOK
#echo $VALUES #Debugging line
while [ "$VALUES" = "NOK" ]
do
	VALUES=`/home/pi/SDM120C/sdm120c /dev/ttyUSB1 -a $IDGroep3 -b 9600 -P N -p -t -q` #i=imported / e=exported / t=totaal / p=vermogen/ q=short output
	#echo $VALUES #Debugging line
	if [[ $VALUES == *NOK* ]]
	then
		VALUES=NOK
	fi
done
VERMOGEN=`echo $VALUES | cut -d ' ' -f1`;
TOTAAL=`echo $VALUES | cut -d ' ' -f2`;
#echo $VERMOGEN
#echo $TOTAAL
URL=$SERVER"/json.htm?type=command&param=udevice&idx="$IDXTGroep3"&nvalue=0&svalue="$VERMOGEN";"$TOTAAL"";
#echo $URL;
#printf %s "$URL" | xxd;
curl $URL #send to domoticz

VALUES=NOK
#echo $VALUES #Debugging line
while [ "$VALUES" = "NOK" ]
do
	VALUES=`/home/pi/SDM120C/sdm120c /dev/ttyUSB1 -a $IDGroep4 -b 9600 -P N -p -t -q` #i=imported / e=exported / t=totaal / p=vermogen/ q=short output
	#echo $VALUES #Debugging line
	if [[ $VALUES == *NOK* ]]
	then
		VALUES=NOK
	fi
done
VERMOGEN=`echo $VALUES | cut -d ' ' -f1`;
TOTAAL=`echo $VALUES | cut -d ' ' -f2`;
#echo $VERMOGEN
#echo $TOTAAL
URL=$SERVER"/json.htm?type=command&param=udevice&idx="$IDXTGroep4"&nvalue=0&svalue="$VERMOGEN";"$TOTAAL"";
#echo $URL;
#printf %s "$URL" | xxd;
curl $URL #send to domoticz

VALUES=NOK
#echo $VALUES #Debugging line
while [ "$VALUES" = "NOK" ]
do
	VALUES=`/home/pi/SDM120C/sdm120c /dev/ttyUSB1 -a $IDGroep5 -b 9600 -P N -p -t -q` #i=imported / e=exported / t=totaal / p=vermogen/ q=short output
	#echo $VALUES #Debugging line
	if [[ $VALUES == *NOK* ]]
	then
		VALUES=NOK
	fi
done
VERMOGEN=`echo $VALUES | cut -d ' ' -f1`;
TOTAAL=`echo $VALUES | cut -d ' ' -f2`;
#echo $VERMOGEN
#echo $TOTAAL
URL=$SERVER"/json.htm?type=command&param=udevice&idx="$IDXTGroep5"&nvalue=0&svalue="$VERMOGEN";"$TOTAAL"";
#echo $URL;
#printf %s "$URL" | xxd;
curl $URL #send to domoticz

VALUES=NOK
#echo $VALUES #Debugging line
while [ "$VALUES" = "NOK" ]
do
	VALUES=`/home/pi/SDM120C/sdm120c /dev/ttyUSB1 -a $IDGroep11 -b 9600 -P N -p -t -q` #i=imported / e=exported / t=totaal / p=vermogen/ q=short output
	#echo $VALUES #Debugging line
	if [[ $VALUES == *NOK* ]]
	then
		VALUES=NOK
	fi
done
VERMOGEN=`echo $VALUES | cut -d ' ' -f1`;
TOTAAL=`echo $VALUES | cut -d ' ' -f2`;
#echo $VERMOGEN
#echo $TOTAAL
URL=$SERVER"/json.htm?type=command&param=udevice&idx="$IDXTGroep11"&nvalue=0&svalue="$VERMOGEN";"$TOTAAL"";
#echo $URL;
#printf %s "$URL" | xxd;
curl $URL #send to domoticz
```
And save it as a .sh file <br />
This file get the information of 4 meters, so change it for the total of meters you need <br />
Also change the text /dev/ttyUSB1 to the USB which is the Modbus-to-USB converter <br />
Copy this file to the folder: /home/pi/domoticz/scripts/SDM120C <br />
 <br />
## Step 4: Automatic readings <br />
Connect to the Pi with Telnet <br />
`crontab -e` <br />
Add the following text at the end of the file <br />
```
PATH=~/bin:/usr/bin/:/bin
SHELL=/bin/bash
*/1 * * * * /home/pi/domoticz/scripts/SDM120C/nameofthefile.sh >/dev/null 2>&1
```

Change the name of the file to what you name it <br />
For more information about crontab: http://www.adminschoice.com/crontab-quick-reference <br />

Now it will work <br />
