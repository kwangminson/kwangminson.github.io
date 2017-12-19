---
layout: post
title: "Code"
---
#Play Music

	import time
	import vlc
	import smbus
	import RPi.GPIO as GPIO
	import os

	# button
	GPIO.setmode(GPIO.BCM)

	GPIO.setup(18,GPIO.IN)#pause
	GPIO.setup(23,GPIO.IN)#change
	GPIO.setup(24,GPIO.IN)#down
	GPIO.setup(25,GPIO.IN)#break


	# Define some device parameters
	I2C_ADDR  = 0x27 # I2C device address
	LCD_WIDTH = 16   # Maximum characters per line
	
	# Define some device constants
	LCD_CHR = 1 # Mode - Sending data
	LCD_CMD = 0 # Mode - Sending command
	
	LCD_LINE_1 = 0x80 # LCD RAM address for the 1st line
	LCD_LINE_2 = 0xC0 # LCD RAM address for the 2nd line
	LCD_LINE_3 = 0x94 # LCD RAM address for the 3rd line
	LCD_LINE_4 = 0xD4 # LCD RAM address for the 4th line
	
	LCD_BACKLIGHT  = 0x08  # On
	#LCD_BACKLIGHT = 0x00  # Off
	
	ENABLE = 0b00000100 # Enable bit
	
	# Timing constants
	E_PULSE = 0.0005
	E_DELAY = 0.0005
	
	#Open I2C interface
	#bus = smbus.SMBus(0)  # Rev 1 Pi uses 0
	bus = smbus.SMBus(1) # Rev 2 Pi uses 1
	
	def lcd_init():
	  # Initialise display
	  lcd_byte(0x33,LCD_CMD) # 110011 Initialise
		  lcd_byte(0x32,LCD_CMD) # 110010 Initialise
	  lcd_byte(0x06,LCD_CMD) # 000110 Cursor move direction
	  lcd_byte(0x0C,LCD_CMD) # 001100 Display On,Cursor Off, Blink Off
	  lcd_byte(0x28,LCD_CMD) # 101000 Data length, number of 	lines, font size
	  lcd_byte(0x01,LCD_CMD) # 000001 Clear display
	  time.sleep(E_DELAY)
	
	def lcd_byte(bits, mode):
	  # Send byte to data pins
	  # bits = the data
	  # mode = 1 for data
	  #        0 for command
	
	  bits_high = mode | (bits & 0xF0) | LCD_BACKLIGHT
	  bits_low = mode | ((bits<<4) & 0xF0) | LCD_BACKLIGHT
	
	  # High bits
	  bus.write_byte(I2C_ADDR, bits_high)
	  lcd_toggle_enable(bits_high)
	
	  # Low bits
	  bus.write_byte(I2C_ADDR, bits_low)
	  lcd_toggle_enable(bits_low)	

	def lcd_toggle_enable(bits):
	  # Toggle enable
	  time.sleep(E_DELAY)
	  bus.write_byte(I2C_ADDR, (bits | ENABLE))
	  time.sleep(E_PULSE)
	  bus.write_byte(I2C_ADDR,(bits & ~ENABLE))
	  time.sleep(E_DELAY)
	
	def lcd_string(message,line):
	  # Send string to display
	  message = message.ljust(LCD_WIDTH," ")
	
	
	  lcd_byte(line, LCD_CMD)
	
	  for i in range(LCD_WIDTH):
	    lcd_byte(ord(message[i]),LCD_CHR)
	
	def read():
	    f = open("newfile.txt","r")
	    num= f.readline()
	    num = int(num)
	    f.close()
	    return num
	
	def 	write(a):
    	f =open("newfile.txt","w")
    	f.write(a)
    	f.close()
	
	
	def main():
	  # Main program block
	
	  # Initialise display
	  lcd_init()
	  path_home = "/home/pi/Music/"
	  a = os.listdir(path_home)
	  music_num = read()
	  file = ('/home/pi/Music/'+a[music_num])
  	instance = vlc.Instance()
  	media = instance.media_new(file)
  	player=instance.media_player_new()
  	player.set_media(media)
  	player.play()
  	time.sleep(1)
	
	  while True:
	    number = player.get_state()
	    if(number==6):
	      music_num+=1
	      file = ('/home/pi/Music/'+a[music_num])
	      instance = vlc.Instance()
	      player=instance.media_player_new()
	      media = instance.media_new(file)
	      player.set_media(media)
	      player.play()
	
	    if GPIO.input(18)==0:
	        if number ==3:
	            player.pause()
	            time.sleep(1)
	        if number ==4:
	            player.play()
	            time.sleep(1)
	
	    elif GPIO.input(23)==0:
	        player.pause()
	        time.sleep(1)
	        print("change the music")
	        music_num+=1
	        if (music_num > 3):
	            music_num = 0
	        file = ('/home/pi/Music/'+a[music_num])
	        instance = vlc.Instance()
	        player=instance.media_player_new()
	        media = instance.media_new(file)
	        player.set_media(media)
	        player.play()
	
	    elif GPIO.input(24)==0:
	        player.pause()
	        time.sleep(1)
    	    print("down music")
    	    music_num -= 1
    	    if (music_num < 0):
    	        music_num = 3
    	    file = ('/home/pi/Music/'+a[music_num])
    	    instance = vlc.Instance()
    	    player=instance.media_player_new()
    	    media = instance.media_new(file)
        	player.set_media(media)
        	player.play()

    elif GPIO.input(25)==0:
        print(" Turnoff the programm !")
        music_num=str(music_num)
        write(music_num)
        break
    
    
    if music_num ==0:
      lcd_string("     Know    ",LCD_LINE_1)
      lcd_string("    C-jamm   ",LCD_LINE_2)
      time.sleep(0.3)
      lcd_string("       Know  ",LCD_LINE_1)
      lcd_string("      C-jamm ",LCD_LINE_2)
      time.sleep(0.3)
      lcd_string("         Know ",LCD_LINE_1)
      lcd_string("        C-jamm",LCD_LINE_2)
      time.sleep(0.3)
      lcd_string("Know          ",LCD_LINE_1)
      lcd_string("C-jamm        ",LCD_LINE_2)
      time.sleep(0.3)
      lcd_string("   Know       ",LCD_LINE_1)
      lcd_string("  C-jamm      ",LCD_LINE_2)
      time.sleep(0.3)
      
    if music_num ==1:
      lcd_string("     Phonecert  ",LCD_LINE_1)
      lcd_string("     10cm   ",LCD_LINE_2)
      time.sleep(0.3)
      lcd_string("     Phonecert  ",LCD_LINE_1)
      lcd_string("       10cm ",LCD_LINE_2)
      time.sleep(0.3)
      lcd_string("      Phonecert ",LCD_LINE_1)
      lcd_string("        10cm",LCD_LINE_2)
      time.sleep(0.3)
      lcd_string(" Phonecert      ",LCD_LINE_1)
      lcd_string("  10cm     ",LCD_LINE_2)
      time.sleep(0.3)
      lcd_string("   Phonecert    ",LCD_LINE_1)
      lcd_string("     10cm   ",LCD_LINE_2)
      time.sleep(0.3)

    if music_num ==2:
      lcd_string("     11:11   ",LCD_LINE_1)
      lcd_string("   Tae-yeon   ",LCD_LINE_2)
      time.sleep(1)

    if music_num ==3:
      lcd_string("      JOAH   ",LCD_LINE_1)
      lcd_string("    Jay-park   ",LCD_LINE_2)
      time.sleep(1)



    if __name__ == '__main__':

	  try:
	    main()
	  except KeyboardInterrupt:
	    pass
	  finally:
	    lcd_byte(0x01, LCD_CMD)	
        
#Button
	import time
	import RPi.GPIO as GPIO

	GPIO.setmode(GPIO.BCM)
	
	GPIO.setup(23, GPIO.OUT)
	GPIO.setup(24, GPIO.OUT)
	
	GPIO.setup(18, GPIO.IN)
	
	print("Button pressed!")
	
	try:
	    while True:
	        GPIO.output(23,False)
	        GPIO.output(24,False)
	
	    if GPIO.input(18)==0:
	        print("Button pressed!")
	
	        GPIO.output(23, True)
	        GPIO.output(24,True)
	
	        time.sleep(1)
	
	        print("Press the button (CTL-C to exit)")
	
	except KeyboardInterrupt:
	    GPIO.cleanup()				
        
#VLC

	 import RPi.GPIO as GPIO
	import time
	import vlc
	
	GPIO.setmode(GPIO.BCM)
	GPIO.setup(18, GPIO.IN)
	GPIO.setup(23, GPIO.IN)
	 
	file ="/home/pi/Desktop/phone_10cm.mp3"
	instance = vlc.Instance()
	 
	player=instance.media_player_new()
	media=instance.media_new(file)
	 
	player.set_media(media)
	
	player.play()
	time.sleep(1)
	 
	while True:
    a=player.get_state()
    
    if GPIO.input(18) == 0:
        if a == 3:
            player.pause()
            a=player.get_state()
            print"pause"
        elif a == 4:
            player.pause()
            a = player.get_state()
            print"resume"
            
    time.sleep(0.5)
 
    if GPIO.input(23) == 0:
        player.stop()
        break
