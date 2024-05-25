# Embaded-WarriorWatchdog
The Soldier Health and Position Tracking System monitors soldiers' GPS location and vital signs like body temperature and heart rate. It includes an emergency distress signal feature. Smart sensors transmit data to a personal server, which communicates with the base station via GSM, enhancing soldier safety and operational efficiency.

            ========== DECLARATION=========

          import time
          import os
          import sys
          import RPi.GPIO as GPIO
          import string
          import pynmea2
          import urllib.request
          import time
          from RPLCD import CharLCD
          from random import *

          sensor = Adafruit_DHT.DHT11

          pin = 4
          buzzerpin=3
          hbpin=29

          hbadcpin=1
          hbrate=0
          humidity=0
          temperature=0
          tempvalstr="0"  
          humidvalstr="0"
          tempsms=0
          hbsms= 0
          hbval=0
          strhbrate="0"
          recvdata=0

          lcd = CharLCD(cols=16, rows=2, pin_rs=14, pin_e=13, pins_data=[12, 11, 10, 7],numbering_mode = GPIO.BCM)


=========== FUNCTION SETUP==============

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)

GPIO.setup(hbpin, GPIO.IN )
GPIO.output(buzzerpin,GPIO.LOW)

port2="/dev/ttyUSB0"
ser2=serial.Serial(port2, baudrate=9600, timeout=0.5)

lcd.clear()
lcd.cursor_pos = (0, 0)
lcd.write_string('SOLDIER SECURITY')
lcd.cursor_pos = (1, 0)
lcd.write_string('MONITORING SYS')

time.sleep(2)

lcd.clear()
                lcd.cursor_pos = (0, 0)
                lcd.write_string('TEMP:')
                lcd.write_string(tempvalstr)
                lcd.write_string("  deg")
                lcd.cursor_pos = (1, 0)
                lcd.write_string('HB RATE:')
                lcd.write_string(str(hbrate))
                lcd.write_string(" bps")
                time.sleep(2)
                lcd.clear()
                lcd.cursor_pos = (0, 0)
                lcd.write_string('LAT:')
                lcd.write_string(strlati)
                lcd.cursor_pos = (1, 0)
                lcd.write_string('LONG:')
                lcd.write_string(strlongi)
                time.sleep(2)


def buzzering():
    GPIO.output(buzzerpin,GPIO.HIGH)
    time.sleep(1)
    GPIO.output(buzzerpin,GPIO.LOW)

def sendsms(msg):
    global strlati
    global strlongi
    global smsnumber
    cmd2 = cmd21 + quote+ smsnumber + quote
    print(cmd2)
    ser2.write(cmd1.encode()) 
    time.sleep(3)
    ser2.write(cmd2.encode())
    ser2.write(cmd22.encode())
    time.sleep(3)
    compmsg = msg + " http://maps.google.com/?q="+strlati+","+strlongi
    print ("Sending SMS with status info:" + compmsg)
    ser2.write(compmsg.encode())
    time.sleep(2)
    ser2.write(chr(26).encode())
    time.sleep(3)
    
def locationsendsms():
    global strlati
    global strlongi
    global strhbrate
    global tempvalstr
    global smsnumber
     
    msg="STATUS : TEMPERATURE:"+ tempvalstr +" HEART BEAT:"+ strhbrate
    
    cmd2 = cmd21 + quote+ smsnumber + quote
    print(cmd2)
    ser2.write(cmd1.encode()) 
    time.sleep(3)
    ser2.write(cmd2.encode())
    ser2.write(cmd22.encode())
    time.sleep(3)
    compmsg = msg + " http://maps.google.com/?q="+strlati+","+strlongi
    print ("Sending SMS with status info:" + compmsg)
    ser2.write(compmsg.encode())
    time.sleep(2)
    ser2.write(chr(26).encode())
    time.sleep(3)    
  
============= LOGIC=====================
         
while True:
                humidity, temperature = Adafruit_DHT.read_retry(sensor, pin)
                tempval=int(temperature)
                humidval=int(humidity)
                tempvalstr=str(tempval)
                humidvalstr=str(humidval)
                #print("*********TEMP:",tempval)
                #print("***************HUMID:",humidval)
                if temperature > 35: 
                    lcd.clear()
                    lcd.cursor_pos = (0, 0)
                    lcd.write_string('HIGH TEMPERATURE')
                    buzzering()
                    if tempsms==0:
                        tempsms=1
                        smsg ="ALERT:HIGH TEMPERATURE DETECTED" + tempvalstr + " deg"
                        
                        sendsms(smsg);
                        print("TEMPERATURE SMS MESSAGE SENT")
                    else:
                        
                        if tempsms==1:
                            tempsms=0
                    
                    
                
           
                print("TEMPERATURE:",tempvalstr)
                print("HUMIDITY:",humidvalstr)
                
             
                    hbrate=randint(75,105)
                else:
                    hbrate=0
                    
                    
                if hbrate > 95:
                    print("HIGH HEART BEAT")
                    lcd.clear()
                    lcd.cursor_pos = (0, 0)
                    lcd.write_string('HIGH HEART BEAT')
                    buzzering()
                    if hbsms==0:
                        hbsms=1
                        smsg ="ALERT:HIGH HEART BEAT DETECTED " + str(hbrate) + " bps"
                        
                        sendsms(smsg);
                        print("HEART BEAT SMS MESSAGE SENT")
                    else:
                        
                        if tempsms==1:
                            tempsms=0
               
                print("HBRATE:",hbrate)
                strhbrate=str(hbrate)
                
                if (ser2.inWaiting()>0):
                     gsmdata=ser2.readline().decode('utf-8',errors='replace')
                     print("========================")
                     
                     if recvdata==0:
                         gsmdata1=gsmdata
                         #print(gsmdata1)
                         recvdata=recvdata+1
                     elif recvdata==1:
                         gsmdata2=gsmdata
                         print("2: ",gsmdata2)
                         print(type(gsmdata2))
                         recvdata=recvdata+1
                     elif recvdata==2:
                         gsmdata3=gsmdata
                         print("3: ",gsmdata3)
                         gsmdata3=gsmdata3.strip()
                         #recvdata=0
                         if gsmdata3=="STATUS":
                             print("**** STATUS &*****")
                             locationsendsms()
                             gsmdata=""
                             recvdata=0
                             gsmdata3=""
                     print("===============\n")
                     #locationsendsms()
                    
                    
                
                
                
                
                if (ser.inWaiting()>0):      
                    newdata=ser.readline().decode('utf-8',errors='replace')
                    #print(newdata)
                    if newdata[0:6] == "$GPRMC":
                        #print(newdata)
                        newmsg=pynmea2.parse(newdata)
                        lat=newmsg.latitude
                        lng=newmsg.longitude
                        lat=round(lat,6)
                        lng=round(lng,6)
                        if lat > 1 and lng > 1:
                            strlati=str(lat)
                            strlongi=str(lng)
                            gps = "Lat=" + strlati + "and Long=" + strlongi
                            print(gps)
                        else:
                            strlati="17.4316"
                            strlongi="78.4399"
                            gps = "Lati=" + strlati + "and Longi=" + strlongi
                            print(gps)
