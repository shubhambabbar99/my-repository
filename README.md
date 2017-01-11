# my projects repository
project name : Enhancement in road safety using raspberry pi 3.
Our project is based on raspberry pi 3 which we have used to enhance the road safety measures. Here we are using various sensors to explain our prototype model.
We have attached a smoke sensor, a water sensor, a traffic red light sensor attached with our raspberry pi device and also we have attached a camera to take snaps of every minute and whenever the sensors are active a notification will be sent to the respected authorities and the snap will be sent to a dropbox account where these snaps can be saved. Here we have used PYTHON  to implement this project.
The script to implement this is as follows:

import os
import time
import urllib
import urllib2
import cookielib
import subprocess
from subprocess import call
import RPi.GPIO as gpio

#=========================================================================

#message = input("Enter Message :")
message = 'System Started'
#number = input("Enter reciever phone number :")
number = '9999047983'
flag1=100
flag2=100
flag3=100
flag4=100
count=0
#=========================================================================
def sendSMS(uname, hashCode, numbers, sender, message):
	data =  urllib.urlencode({'username': uname, 'hash': hashCode, 'numbers': numbers, 'message' : message, 'sender': sender})
	data = data.encode('utf-8')
	request = urllib2.Request("http://api.textlocal.in/send/?")
	f = urllib2.urlopen(request, data)
	fr = f.read()
    	return(fr)
#=========================================================================
def sw1_detect(pin):
	print 'Water Logging Detected'
	gpio.remove_event_detect(7)
	global flag1
	if (flag1==100):
		flag1=0
		subprocess.call("./save1.sh", shell=False)
		print('W inactive')
		resp =  sendSMS('bishan@bm-es.com', 'satnamWAHEGURU123', number, 'TXTLCL', 'Water Logging Detected')
		print (resp)

#=========================================================================
def sw2_detect(pin):
	print 'Smoke Detected'
	gpio.remove_event_detect(11)
	global flag2
	if (flag2==100):
		flag2=0
		subprocess.call("./save2.sh", shell=False)
		print('S inactive')
		resp =  sendSMS('bishan@bm-es.com', 'satnamWAHEGURU123', number, 'TXTLCL', 'Smoke Detected')
		print (resp)
#=========================================================================
def sw3_detect(pin):
	print 'Traffic Light Crossing Detected'
	global count
	if count>=6 and count<=12 :
		gpio.remove_event_detect(12)
		global flag3
		if (flag3==100):
			flag3=0
			subprocess.call("./save3.sh", shell=False)
			print('T inactive')
			resp =  sendSMS('bishan@bm-es.com', 'satnamWAHEGURU123', number, 'TXTLCL', 'Traffic Light Crossing Detected')
			print (resp)
#=========================================================================
def sw4_detect(pin):
	print 'Image Saving Detected'
	gpio.remove_event_detect(13)
	global flag4
	if (flag4==100):
		flag4=0
		subprocess.call("./save4.sh", shell=False)
		print('I inactive')
#=========================================================================
def init_io():
	gpio.setmode(gpio.BOARD) # Set pin numbering to board numbering
	gpio.setwarnings(False)
	gpio.setup(7, gpio.IN) # Set up pin 7 as an input
	gpio.add_event_detect(7, gpio.RISING, callback=sw1_detect, bouncetime=200) # Set up an interrupt to look for button presses
	gpio.setup(11, gpio.IN) # Set up pin 11 as an input
	gpio.add_event_detect(11, gpio.FALLING, callback=sw2_detect, bouncetime=200) # Set up an interrupt to look for button presses
	gpio.setup(12, gpio.IN) # Set up pin 12 as an input
	gpio.add_event_detect(12, gpio.FALLING, callback=sw3_detect, bouncetime=200) # Set up an interrupt to look for button presses
	gpio.setup(13, gpio.IN) # Set up pin 12 as an input
	gpio.add_event_detect(13, gpio.RISING, callback=sw4_detect, bouncetime=200) # Set up an interrupt to look for button presses
	
	gpio.setup(16, gpio.OUT) # Set up pin 16 as an input
	gpio.output(16, False)
	gpio.setup(18, gpio.OUT) # Set up pin 18 as an input
	gpio.output(18, True)

	gpio.setup(22, gpio.OUT) # Set up pin 22 as an input
	gpio.output(22, False)


#=========================================================================
def check_red_green() :
	global count
	count=count+1
	#Red 6-12
	if count==6 :
		gpio.output(16, True)
		gpio.output(18, False)
	#Green 1-6
	if count==12 :
		gpio.output(16, False)
		gpio.output(18, True)
		gpio.output(22, True)
		count=0

#=========================================================================
def check_sw1() :
	global flag1
	flag1=flag1+1
	if flag1>=6 and flag1<=15 :
		flag1=100
		print('W active')
		gpio.add_event_detect(7, gpio.RISING, callback=sw1_detect, bouncetime=200) # Set up an interrupt to look for button presses	
	if flag1>=100 :
		flag1=100

#=========================================================================
def check_sw2() :
	global flag2
	flag2=flag2+1
	if flag2>=6 and flag2<=15 :
		flag2=100
		print('S active')
		gpio.add_event_detect(11, gpio.FALLING, callback=sw2_detect, bouncetime=200) # Set up an interrupt to look for button presses	
	if flag2>=100 :
		flag2=100
#=========================================================================
def check_sw3() :
	global flag3
	flag3=flag3+1
	if flag3>=6 and flag3<=15 :
		flag3=100
		print('T active')
		gpio.add_event_detect(12, gpio.FALLING, callback=sw3_detect, bouncetime=200) # Set up an interrupt to look for button presses	
	if flag3>=100 :
		flag3=100
#=========================================================================
def check_sw4() :
	global flag4
	flag4=flag4+1
	if flag4>=6 and flag4<=15 :
		flag4=100
		print('I active')
		gpio.output(22, False)
		gpio.add_event_detect(13, gpio.RISING, callback=sw4_detect, bouncetime=200) # Set up an interrupt to look for button presses	
	if flag4>=100 :
		flag4=100
#=========================================================================
# main() function
def main():
	print("\n******** System Started ********")
	print ("Sending Initial SMS...")
	resp =  sendSMS('bishan@bm-es.com', 'satnamWAHEGURU123', number, 'TXTLCL', message)
	print (resp)
	time.sleep(2)
	init_io()
	while (True):
		time.sleep(5)
		check_red_green()
		check_sw1()
		check_sw2()
		check_sw3()
		check_sw4()
		

#=========================================================================
if __name__ == '__main__':
	main()
#=========================================================================

