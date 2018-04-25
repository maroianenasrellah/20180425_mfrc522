# 20180425_mfrc522
#myprogram
#!/usr/bin/env python
# -*- coding: utf8 -*-
import RPi.GPIO as GPIO
import MFRC522
import signal
import time
import datetime
#from datetime import date
GPIO.setwarnings(False)
cmpt=0

date = datetime.datetime.now()
#print(date)
    
continue_reading = True
encore = True
def function():
    today = datetime.datetime.today()
    s="X"+today.strftime("%Y%m%d")
    data = []
    for c in  s:
        if (len(data)<16):
            data.append(int(ord(c)))
    while(len(data)!=16):
        data.append(0)
        
    return data

def my_function(kel_data):
    data = []
     #texte = input("Entrez une chaine de caractère :\n")
    texte = input("Entrez votre %s"%(kel_data)+" :\n")
    for c in texte:
        if (len(data)<16):
            data.append(int(ord(c)))
    while(len(data)!=16):
        data.append(0)
        
    return data

# Capture SIGINT for cleanup when the script is aborted
def end_read(signal,frame):
    global continue_reading
    print ("Lecture terminée")
    continue_reading = False
    GPIO.cleanup()
# Hook the SIGINT
signal.signal(signal.SIGINT, end_read)
# Create an object of the class MFRC522
MIFAREReader = MFRC522.MFRC522()
#print ("Press Ctrl-C to stop.")
#secteurBloc=eval(input("Entrez un Secteur :\n"))
secteurBlock2=8
secteurBlock3=12

print ("Passer le tag RFID a lire\n")
sf=function()
# This loop keeps checking for chips. If one is near it will get the UID and authenticate
# Scan for cards 
while continue_reading:
    # Scan for cards 
    (status,TagType) = MIFAREReader.MFRC522_Request(MIFAREReader.PICC_REQIDL)
     # If a card is found
    if status == MIFAREReader.MI_OK:
        print ("Carte detectee")
    # Get the UID of the card
    (status,uid) = MIFAREReader.MFRC522_Anticoll()
    # If we have the UID, continue
    if status == MIFAREReader.MI_OK:
        print ("UID de la carte : "+str(uid[0])+"."+str(uid[1])+"."+str(uid[2])+"."+str(uid[3])+"."+str(uid[4]))
        # This is the default key for authentication
        keyA_Public = [0xFF,0xFF,0xFF,0xFF,0xFF,0xFF]
        # This is the Private key for authentication
        keyA_Privé = [0x59,0x61,0x50,0x6F,0x54,0x74] #"YaPoTt"
        
        # Select the scanned tag
        MIFAREReader.MFRC522_SelectTag(uid)
        
        # Authenticate
        status = MIFAREReader.MFRC522_Auth(MIFAREReader.PICC_AUTHENT1A, secteurBlock2,  keyA_Privé, uid)
        if status == MIFAREReader.MI_OK:
            print("\n")
            print("Authentication Avec la Clee Privé sur secteur",secteurBlock2)
            print("\n")
            print ("Le secteur ",secteurBlock2," contient actuellement : ")
            MIFAREReader.MFRC522_Read(secteurBlock2)
            
            print ("Le secteur ",secteurBlock2+1," contient actuellement : ")
            MIFAREReader.MFRC522_Read(9)
            
            print ("Le secteur ",secteurBlock2+2," contient actuellement : ")
            MIFAREReader.MFRC522_Read(10)
            
            print ("Le secteur ",secteurBlock2+3," contient actuellement : ")
            MIFAREReader.MFRC522_Read(11)
            
            MIFAREReader.MFRC522_StopCrypto1()
            next = False
        else:
            print("\nAuthentication error Avec la Clee Privé sur secteur",secteurBlock2,"\n")
            next =True
            
        if(next == True):
           #continue_reading = False
            #print("Authentication error Avec la Clee Privé sur secteur",secteurBlock2)
            (status,TagType) = MIFAREReader.MFRC522_Request(MIFAREReader.PICC_REQIDL)
            (status,uid) = MIFAREReader.MFRC522_Anticoll()
            MIFAREReader.MFRC522_SelectTag(uid)
            print ("Authentification Avec la Clee Public sur secteur ",secteurBlock2,"\n")
            status1 = MIFAREReader.MFRC522_Auth(MIFAREReader.PICC_AUTHENT1A, secteurBlock2, keyA_Public, uid)
            if(status1 == MIFAREReader.MI_OK):
                print ("Réussite Authentification Avec la Clee Public sur secteur ",secteurBlock2)
                print("\n")
                print ("Le secteur ",secteurBlock2," contient actuellement : ")
                MIFAREReader.MFRC522_Read(secteurBlock2)
                print ("Le secteur ",secteurBlock2+1," contient actuellement : ")
                MIFAREReader.MFRC522_Read(9)
                print ("Le secteur ",secteurBlock2+2," contient actuellement : ")
                MIFAREReader.MFRC522_Read(10)
                print ("Le secteur ",secteurBlock2+3," contient actuellement : ")
                MIFAREReader.MFRC522_Read(11)
                
            else:
                 print ("Erreur d\'Authentification Avec la Clee Public sur secteur",secteurBlock2)
                 continue_reading = False
        else:
            print("Authentication error Avec la Clee  sur secteur",secteurBlock2)
            continue_reading = False
            
####        # Authenticate
        status1 = MIFAREReader.MFRC522_Auth(MIFAREReader.PICC_AUTHENT1A, secteurBlock3,  keyA_Privé, uid)
        if status1 == MIFAREReader.MI_OK:
            print("\n")
            print("Authentication Avec la Clee Privé sur secteur",secteurBlock3,"\n")
            print("\n")
            print ("Le secteur ",secteurBlock3," contient actuellement : ")
            MIFAREReader.MFRC522_Read(secteurBlock3)
            #print ("mise a jour...La Date sur secteur ",secteurBlock3)
            MIFAREReader.MFRC522_Write(secteurBlock3,sf)
            #print("\n")
            print ("Date Dernier Passage : ")
            MIFAREReader.MFRC522_Read(secteurBlock3)
            
            MIFAREReader.MFRC522_StopCrypto1()
            nexto = False
        else:
            print("\nAuthentication error Avec la Clee Privé sur secteur",secteurBlock3,"\n")
            nexto =True
        #nexto =True
            
        if(nexto == True):
            (status,TagType) = MIFAREReader.MFRC522_Request(MIFAREReader.PICC_REQIDL)
            (status,uid) = MIFAREReader.MFRC522_Anticoll()
            MIFAREReader.MFRC522_SelectTag(uid)
            print ("Authentification Avec la Clee Public sur secteur ",secteurBlock3,"\n")
            status = MIFAREReader.MFRC522_Auth(MIFAREReader.PICC_AUTHENT1A, secteurBlock3, keyA_Public, uid)
            if(status == MIFAREReader.MI_OK):
                print ("Réussite Authentification Avec la Clee Public sur secteur ",secteurBlock3)
                print("\n")
                print ("Le secteur ",secteurBlock3," contient actuellement : ")
                MIFAREReader.MFRC522_Read(secteurBlock3)
                print ("Le secteur ",secteurBlock3+1," contient actuellement : ")
                MIFAREReader.MFRC522_Read(13)
                print ("Le secteur ",secteurBlock3+2," contient actuellement : ")
                MIFAREReader.MFRC522_Read(14)
                print ("Le secteur ",secteurBlock3+3," contient actuellement : ")
                MIFAREReader.MFRC522_Read(15)
            else:
                print ("Erreur d\'Authentification Avec la Clee Public sur secteur",secteurBlock3)
                continue_reading = False
        
